---
title: "Background Jobs & API"
description: "Queue processing, cleanup jobs, and REST API endpoints for import/export"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: background jobs, queue, API endpoints, REST, cron
---

# Background Jobs & API

This document covers the queue processing for async operations, cleanup cron jobs, and REST API endpoints for managing import/export operations.

## Background Jobs

### Import Processing

```typescript
importQueue.process('process-import', async (job) => {
  const { jobId, options } = job.data;
  const service = new ImportService();

  await service.processImport(jobId, options);
});
```

### Export Processing

```typescript
exportQueue.process('process-export', async (job) => {
  const { jobId, options } = job.data;
  const service = new ExportService();

  await service.processExport(jobId, options);
});
```

### Cleanup Old Exports

Exports are automatically deleted after 7 days to manage storage costs.

```typescript
cron.schedule('0 4 * * *', async () => {  // 4 AM daily
  const expiredExports = await db.exportJobs.findAll({
    where: {
      expiresAt: { [Op.lt]: new Date() },
      status: 'completed',
    },
  });

  for (const exportJob of expiredExports) {
    // Delete from S3
    if (exportJob.fileUrl) {
      await s3.deleteObject(exportJob.fileUrl);
    }

    // Delete job
    await db.exportJobs.delete(exportJob.id);
  }
});
```

## API Endpoints

### Start Import

```typescript
app.post('/api/contacts/import', authenticate, async (req, res) => {
  const { fileName, fileUrl, fieldMapping, options } = req.body;

  const service = new ImportService();
  const result = await service.startImport(req.user.tenantId, {
    fileName,
    fileUrl,
    fileType: 'csv',
    fieldMapping,
    ...options,
  });

  return res.json(result);
});
```

### Get Import Status

```typescript
app.get('/api/contacts/import/:jobId', authenticate, async (req, res) => {
  const job = await db.importJobs.findById(req.params.jobId);

  if (job.tenantId !== req.user.tenantId) {
    return res.status(403).json({ error: 'Forbidden' });
  }

  return res.json(job);
});
```

### Rollback Import

```typescript
app.post('/api/contacts/import/:jobId/rollback', authenticate, async (req, res) => {
  const service = new ImportService();
  await service.rollbackImport(req.params.jobId);

  return res.json({ success: true });
});
```

### Start Export

```typescript
app.post('/api/contacts/export', authenticate, async (req, res) => {
  const service = new ExportService();
  const result = await service.startExport(req.user.tenantId, req.body);

  return res.json(result);
});
```

### Get Export Status

```typescript
app.get('/api/contacts/export/:jobId', authenticate, async (req, res) => {
  const job = await db.exportJobs.findById(req.params.jobId);

  if (job.tenantId !== req.user.tenantId) {
    return res.status(403).json({ error: 'Forbidden' });
  }

  return res.json(job);
});
```

### Download Export

```typescript
app.get('/api/contacts/export/:jobId/download', authenticate, async (req, res) => {
  const job = await db.exportJobs.findById(req.params.jobId);

  if (job.tenantId !== req.user.tenantId) {
    return res.status(403).json({ error: 'Forbidden' });
  }

  if (job.status !== 'completed') {
    return res.status(400).json({ error: 'Export not ready' });
  }

  // Generate signed URL for S3 download
  const signedUrl = await s3.getSignedUrl(job.fileUrl, 300);  // 5 minutes

  return res.json({ downloadUrl: signedUrl });
});
```

## Endpoint Summary

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /api/contacts/import | Start import job |
| GET | /api/contacts/import/:jobId | Get import status |
| POST | /api/contacts/import/:jobId/rollback | Rollback import |
| POST | /api/contacts/export | Start export job |
| GET | /api/contacts/export/:jobId | Get export status |
| GET | /api/contacts/export/:jobId/download | Get download URL |

## Related Documentation

- [Database Schema](./database-schema.md) - Table structures
- [Import Service](./import-service.md) - Import processing
- [Export Service](./export-service.md) - Export generation
