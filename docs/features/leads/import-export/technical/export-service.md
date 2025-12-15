---
title: "Export Service"
description: "TypeScript implementation for contact export with filtering and file generation"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: export service, TypeScript, CSV, Excel, file generation
---

# Export Service

The ExportService class handles contact export operations including filtering, field selection, and file generation in multiple formats.

## Interfaces

```typescript
interface ExportOptions {
  format: 'csv' | 'xlsx' | 'json';
  filters?: any;
  segmentId?: string;
  selectedFields?: string[];
  includeCustomFields?: boolean;
  includeEngagementMetrics?: boolean;
}
```

## Starting an Export

```typescript
class ExportService {
  async startExport(
    tenantId: string,
    options: ExportOptions
  ): Promise<{ jobId: string }> {
    const job = await db.exportJobs.create({
      tenantId,
      exportType: options.segmentId ? 'segment' : options.filters ? 'filtered' : 'full',
      filters: options.filters,
      selectedFields: options.selectedFields,
      fileFormat: options.format,
      status: 'pending',
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),  // 7 days
    });

    // Queue for processing
    await exportQueue.add('process-export', {
      jobId: job.id,
      options,
    });

    return { jobId: job.id };
  }
}
```

## Processing an Export

```typescript
async processExport(jobId: string, options: ExportOptions): Promise<void> {
  try {
    await db.exportJobs.update(jobId, {
      status: 'processing',
      startedAt: new Date(),
    });

    // Fetch contacts
    const contacts = await this.fetchContacts(options);

    await db.exportJobs.update(jobId, {
      totalContacts: contacts.length,
    });

    // Generate file
    const fileContent = await this.generateFile(contacts, options);

    // Upload to S3
    const fileUrl = await s3.putObject({
      bucket: 'exports',
      key: `${jobId}.${options.format}`,
      body: fileContent,
      expiresIn: 7 * 24 * 60 * 60,  // 7 days
    });

    await db.exportJobs.update(jobId, {
      status: 'completed',
      completedAt: new Date(),
      fileUrl,
      fileSize: fileContent.length,
    });

  } catch (error) {
    await db.exportJobs.update(jobId, {
      status: 'failed',
      completedAt: new Date(),
    });
    throw error;
  }
}
```

## Fetching Contacts

```typescript
private async fetchContacts(options: ExportOptions): Promise<any[]> {
  let query: any = {};

  if (options.segmentId) {
    query.include = [{
      model: db.segmentContacts,
      where: { segmentId: options.segmentId },
    }];
  }

  if (options.filters) {
    query.where = options.filters;
  }

  return await db.contacts.findAll(query);
}
```

## File Generation

```typescript
private async generateFile(
  contacts: any[],
  options: ExportOptions
): Promise<Buffer> {
  switch (options.format) {
    case 'csv':
      return this.generateCSV(contacts, options);
    case 'xlsx':
      return this.generateExcel(contacts, options);
    case 'json':
      return Buffer.from(JSON.stringify(contacts, null, 2));
    default:
      throw new Error(`Unsupported format: ${options.format}`);
  }
}
```

## CSV Generation

```typescript
private async generateCSV(
  contacts: any[],
  options: ExportOptions
): Promise<Buffer> {
  const fields = options.selectedFields || [
    'email',
    'firstName',
    'lastName',
    'company',
    'phone',
  ];

  const csvWriter = createObjectCsvStringifier({
    header: fields.map(f => ({ id: f, title: f })),
  });

  const csv = csvWriter.getHeaderString() + csvWriter.stringifyRecords(contacts);

  return Buffer.from(csv);
}
```

## Excel Generation

```typescript
private async generateExcel(
  contacts: any[],
  options: ExportOptions
): Promise<Buffer> {
  const workbook = XLSX.utils.book_new();
  const worksheet = XLSX.utils.json_to_sheet(contacts);

  XLSX.utils.book_append_sheet(workbook, worksheet, 'Contacts');

  return XLSX.write(workbook, { type: 'buffer', bookType: 'xlsx' });
}
```

## Related Documentation

- [Database Schema](./database-schema.md) - Table structures
- [Import Service](./import-service.md) - Import functionality
- [Background Jobs & API](./background-jobs-api.md) - Queue processing
