---
title: "Import Service"
description: "TypeScript implementation for contact import processing with validation and rollback"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: import service, TypeScript, validation, field mapping, rollback
---

# Import Service

The ImportService class handles contact import operations including file parsing, validation, duplicate handling, and rollback functionality.

## Interfaces

```typescript
interface ImportOptions {
  fileName: string;
  fileUrl: string;
  fileType: 'csv' | 'xlsx' | 'json';
  fieldMapping: Record<string, string>;
  duplicateStrategy: 'skip' | 'update' | 'create';
  tags?: string[];
  segmentName?: string;
  validateEmail?: boolean;
  validatePhone?: boolean;
}

interface ImportResult {
  jobId: string;
  totalRows: number;
  imported: number;
  created: number;
  updated: number;
  skipped: number;
  errors: ImportError[];
}

interface ImportError {
  row: number;
  field?: string;
  value?: any;
  error: string;
}
```

## Starting an Import

```typescript
class ImportService {
  async startImport(
    tenantId: string,
    options: ImportOptions
  ): Promise<ImportResult> {
    // Create import job
    const job = await db.importJobs.create({
      tenantId,
      fileName: options.fileName,
      fileUrl: options.fileUrl,
      fileType: options.fileType,
      fieldMapping: options.fieldMapping,
      importOptions: {
        duplicateStrategy: options.duplicateStrategy,
        tags: options.tags,
        segmentName: options.segmentName,
      },
      status: 'pending',
    });

    // Queue for processing
    await importQueue.add('process-import', {
      jobId: job.id,
      options,
    });

    return {
      jobId: job.id,
      totalRows: 0,
      imported: 0,
      created: 0,
      updated: 0,
      skipped: 0,
      errors: [],
    };
  }
}
```

## Processing an Import

```typescript
async processImport(jobId: string, options: ImportOptions): Promise<void> {
  const job = await db.importJobs.findById(jobId);

  try {
    await db.importJobs.update(jobId, {
      status: 'processing',
      startedAt: new Date(),
    });

    // Download file from S3
    const fileContent = await s3.getObject(options.fileUrl);

    // Parse file
    const rows = await this.parseFile(fileContent, options.fileType);

    await db.importJobs.update(jobId, {
      totalRows: rows.length,
    });

    // Process rows
    const results = await this.processRows(jobId, rows, options);

    // Create segment if specified
    if (options.segmentName) {
      await this.createSegment(
        job.tenantId,
        options.segmentName,
        results.contactIds
      );
    }

    // Update job with results
    await db.importJobs.update(jobId, {
      status: 'completed',
      completedAt: new Date(),
      importedCount: results.imported,
      createdCount: results.created,
      updatedCount: results.updated,
      skippedCount: results.skipped,
      errorCount: results.errors.length,
      errors: results.errors,
    });

    await this.notifyImportComplete(job, results);

  } catch (error) {
    await db.importJobs.update(jobId, {
      status: 'failed',
      completedAt: new Date(),
      errors: [{ row: 0, error: error.message }],
    });

    throw error;
  }
}
```

## File Parsing

```typescript
private async parseFile(
  content: Buffer,
  fileType: string
): Promise<any[]> {
  switch (fileType) {
    case 'csv':
      return await this.parseCSV(content);
    case 'xlsx':
      return await this.parseExcel(content);
    case 'json':
      return JSON.parse(content.toString());
    default:
      throw new Error(`Unsupported file type: ${fileType}`);
  }
}

private async parseCSV(content: Buffer): Promise<any[]> {
  return new Promise((resolve, reject) => {
    const results: any[] = [];

    const stream = Readable.from(content);

    stream
      .pipe(csv())
      .on('data', (row) => results.push(row))
      .on('end', () => resolve(results))
      .on('error', reject);
  });
}

private async parseExcel(content: Buffer): Promise<any[]> {
  const workbook = XLSX.read(content);
  const sheetName = workbook.SheetNames[0];
  const sheet = workbook.Sheets[sheetName];

  return XLSX.utils.sheet_to_json(sheet);
}
```

## Row Processing

```typescript
private async processRows(
  jobId: string,
  rows: any[],
  options: ImportOptions
): Promise<{
  imported: number;
  created: number;
  updated: number;
  skipped: number;
  errors: ImportError[];
  contactIds: string[];
}> {
  let imported = 0;
  let created = 0;
  let updated = 0;
  let skipped = 0;
  const errors: ImportError[] = [];
  const contactIds: string[] = [];

  for (let i = 0; i < rows.length; i++) {
    const row = rows[i];

    try {
      const mappedData = this.mapFields(row, options.fieldMapping);
      const validation = await this.validateContact(mappedData, options);

      if (!validation.valid) {
        errors.push({ row: i + 1, error: validation.error });
        skipped++;
        continue;
      }

      const existing = await db.contacts.findByEmail(mappedData.email);

      if (existing) {
        if (options.duplicateStrategy === 'skip') {
          skipped++;
          continue;
        } else if (options.duplicateStrategy === 'update') {
          await db.importEvents.create({
            importJobId: jobId,
            contactId: existing.id,
            eventType: 'updated',
            previousData: existing,
            newData: mappedData,
          });

          await db.contacts.update(existing.id, mappedData);
          updated++;
          imported++;
          contactIds.push(existing.id);
        }
      } else {
        const contact = await db.contacts.create(mappedData);

        await db.importEvents.create({
          importJobId: jobId,
          contactId: contact.id,
          eventType: 'created',
          newData: mappedData,
        });

        created++;
        imported++;
        contactIds.push(contact.id);
      }

      if (options.tags?.length > 0) {
        const contact = existing || await db.contacts.findByEmail(mappedData.email);
        await this.addTags(contact.id, options.tags);
      }

      await db.importJobs.update(jobId, { processedRows: i + 1 });

    } catch (error) {
      errors.push({ row: i + 1, error: error.message });
      skipped++;
    }
  }

  return { imported, created, updated, skipped, errors, contactIds };
}
```

## Validation

```typescript
private async validateContact(
  data: any,
  options: ImportOptions
): Promise<{ valid: boolean; error?: string }> {
  if (!data.email) {
    return { valid: false, error: 'Email is required' };
  }

  if (options.validateEmail) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(data.email)) {
      return { valid: false, error: 'Invalid email format' };
    }
  }

  if (options.validatePhone && data.phone) {
    const phoneRegex = /^[\+]?[(]?[0-9]{3}[)]?[-\s\.]?[0-9]{3}[-\s\.]?[0-9]{4,6}$/;
    if (!phoneRegex.test(data.phone)) {
      return { valid: false, error: 'Invalid phone format' };
    }
  }

  return { valid: true };
}
```

## Rollback

```typescript
async rollbackImport(jobId: string): Promise<void> {
  const job = await db.importJobs.findById(jobId);

  if (!job.canRollback) {
    throw new Error('Import cannot be rolled back');
  }

  const events = await db.importEvents.findAll({
    where: { importJobId: jobId },
  });

  for (const event of events) {
    if (event.eventType === 'created') {
      await db.contacts.delete(event.contactId);
    } else if (event.eventType === 'updated') {
      await db.contacts.update(event.contactId, event.previousData);
    }
  }

  await db.importJobs.update(jobId, {
    status: 'rolled_back',
    rolledBackAt: new Date(),
  });
}
```

## Related Documentation

- [Database Schema](./database-schema.md) - Table structures
- [Export Service](./export-service.md) - Export functionality
- [Background Jobs & API](./background-jobs-api.md) - Queue processing
