---
title: "Technical Implementation - Manual Reporting"
description: "Database schema, report generation service, and API endpoints for manual reporting"
last_modified_date: "2025-12-15"
level: "3"
status: "ACTIVE"
parent: "manual-reporting"
---

# Technical Implementation

Technical details for implementing manual reporting and data export functionality.

## Database Schema

```sql
-- Scheduled reports
CREATE TABLE scheduled_reports (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  workspace_id UUID REFERENCES workspaces(id),

  name VARCHAR(255) NOT NULL,
  description TEXT,

  -- Schedule
  frequency VARCHAR(50), -- daily, weekly, monthly
  day_of_week INTEGER,   -- 0-6 for weekly
  day_of_month INTEGER,  -- 1-31 for monthly
  time TIME,
  timezone VARCHAR(50) DEFAULT 'UTC',

  -- Configuration
  report_type VARCHAR(50), -- campaign, deliverability, workspace_summary
  date_range VARCHAR(50),  -- last_7d, last_30d, etc.
  format VARCHAR(20),      -- csv, excel, pdf
  config JSONB,            -- Additional configuration

  -- Delivery
  recipients TEXT[],       -- Array of email addresses
  is_active BOOLEAN DEFAULT TRUE,

  -- Metadata
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  last_run_at TIMESTAMP,
  next_run_at TIMESTAMP
);

-- Report generation history
CREATE TABLE report_generations (
  id UUID PRIMARY KEY,
  scheduled_report_id UUID REFERENCES scheduled_reports(id),
  tenant_id UUID NOT NULL REFERENCES tenants(id),

  -- Generation details
  generated_at TIMESTAMP DEFAULT NOW(),
  generation_time_ms INTEGER,
  status VARCHAR(50),      -- success, failed, partial
  error_message TEXT,

  -- Output
  file_path TEXT,         -- S3 path or local path
  file_size_bytes INTEGER,
  row_count INTEGER,

  -- Delivery
  sent_to TEXT[],
  delivery_status JSONB,  -- Per-recipient delivery status

  metadata JSONB
);

-- Data export jobs
CREATE TABLE export_jobs (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  user_id UUID REFERENCES users(id),

  -- Export config
  export_type VARCHAR(50),  -- campaigns, leads, metrics
  format VARCHAR(20),       -- csv, excel, json
  date_range JSONB,
  filters JSONB,
  fields TEXT[],

  -- Processing
  status VARCHAR(50),       -- pending, processing, completed, failed
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  processing_time_ms INTEGER,

  -- Output
  file_path TEXT,
  file_size_bytes INTEGER,
  row_count INTEGER,
  download_url TEXT,
  expires_at TIMESTAMP,     -- Temporary download link expiration

  -- Error handling
  error_message TEXT,

  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## Report Generation Service

```typescript
import { Parser } from 'json2csv';
import ExcelJS from 'exceljs';
import PDFDocument from 'pdfkit';

class ReportGenerator {
  async generateReport(config: ScheduledReport): Promise<string> {
    // 1. Fetch data
    const data = await this.fetchReportData(config);

    // 2. Generate file based on format
    let filePath: string;
    switch (config.format) {
      case 'csv':
        filePath = await this.generateCSV(data, config);
        break;
      case 'excel':
        filePath = await this.generateExcel(data, config);
        break;
      case 'pdf':
        filePath = await this.generatePDF(data, config);
        break;
      default:
        throw new Error(`Unsupported format: ${config.format}`);
    }

    // 3. Upload to storage
    const uploadedPath = await this.uploadToS3(filePath);

    // 4. Send to recipients
    await this.emailReport(config.recipients, uploadedPath, config);

    // 5. Log generation
    await db.reportGenerations.create({
      scheduledReportId: config.id,
      tenantId: config.tenantId,
      generatedAt: new Date(),
      filePath: uploadedPath,
      sentTo: config.recipients,
      status: 'success',
    });

    return uploadedPath;
  }

  private async generateCSV(data: any[], config: ScheduledReport): Promise<string> {
    const fields = this.getFieldsForReportType(config.reportType);
    const parser = new Parser({ fields });
    const csv = parser.parse(data);

    const filePath = `/tmp/report-${config.id}-${Date.now()}.csv`;
    await fs.promises.writeFile(filePath, csv);

    return filePath;
  }

  private async generateExcel(data: any[], config: ScheduledReport): Promise<string> {
    const workbook = new ExcelJS.Workbook();
    const worksheet = workbook.addWorksheet('Report');

    // Add header
    const fields = this.getFieldsForReportType(config.reportType);
    worksheet.addRow(fields.map(f => f.label));

    // Style header
    worksheet.getRow(1).font = { bold: true };
    worksheet.getRow(1).fill = {
      type: 'pattern',
      pattern: 'solid',
      fgColor: { argb: 'FF4472C4' },
    };

    // Add data rows
    data.forEach(row => {
      worksheet.addRow(fields.map(f => row[f.value]));
    });

    // Auto-fit columns
    worksheet.columns.forEach(column => {
      column.width = 15;
    });

    const filePath = `/tmp/report-${config.id}-${Date.now()}.xlsx`;
    await workbook.xlsx.writeFile(filePath);

    return filePath;
  }

  private async generatePDF(data: any[], config: ScheduledReport): Promise<string> {
    const doc = new PDFDocument();
    const filePath = `/tmp/report-${config.id}-${Date.now()}.pdf`;
    const stream = fs.createWriteStream(filePath);

    doc.pipe(stream);

    // Title
    doc.fontSize(20).text(config.name, { align: 'center' });
    doc.moveDown();

    // Date range
    doc.fontSize(12).text(`Period: ${this.formatDateRange(config.dateRange)}`);
    doc.moveDown();

    // Summary metrics
    const summary = this.calculateSummary(data);
    doc.fontSize(14).text('Summary', { underline: true });
    doc.fontSize(10);
    Object.entries(summary).forEach(([key, value]) => {
      doc.text(`${key}: ${value}`);
    });

    doc.moveDown();

    // Detailed table
    doc.fontSize(14).text('Detailed Metrics', { underline: true });
    // ... table rendering logic ...

    doc.end();

    return new Promise((resolve) => {
      stream.on('finish', () => resolve(filePath));
    });
  }
}
```

---

## Scheduled Report Runner

```typescript
// Background job to run scheduled reports
cron.schedule('*/15 * * * *', async () => {  // Every 15 minutes
  const dueReports = await db.scheduledReports.findDue();

  for (const report of dueReports) {
    await reportQueue.add('generate-report', {
      reportId: report.id,
    });

    // Update next run time
    await db.scheduledReports.update(report.id, {
      nextRunAt: calculateNextRun(report),
    });
  }
});

// Queue worker
reportQueue.process('generate-report', async (job) => {
  const { reportId } = job.data;
  const report = await db.scheduledReports.findById(reportId);

  try {
    const generator = new ReportGenerator();
    await generator.generateReport(report);

    await db.scheduledReports.update(reportId, {
      lastRunAt: new Date(),
    });
  } catch (error) {
    logger.error('Report generation failed:', error);

    await db.reportGenerations.create({
      scheduledReportId: reportId,
      tenantId: report.tenantId,
      status: 'failed',
      errorMessage: error.message,
    });
  }
});
```

---

## API Endpoints

### Create Export Job

```typescript
// POST /api/exports
app.post('/api/exports', authenticate, async (req, res) => {
  const { exportType, format, dateRange, filters, fields } = req.body;

  // Create export job
  const exportJob = await db.exportJobs.create({
    tenantId: req.user.tenantId,
    userId: req.user.id,
    exportType,
    format,
    dateRange,
    filters,
    fields,
    status: 'pending',
  });

  // Queue for processing
  await exportQueue.add('process-export', {
    exportJobId: exportJob.id,
  });

  return res.json({
    exportId: exportJob.id,
    status: 'pending',
    estimatedTime: estimateExportTime(exportType, dateRange),
  });
});
```

### Get Export Status

```typescript
// GET /api/exports/:id
app.get('/api/exports/:id', authenticate, async (req, res) => {
  const exportJob = await db.exportJobs.findById(req.params.id);

  if (exportJob.tenantId !== req.user.tenantId) {
    return res.status(403).json({ error: 'Forbidden' });
  }

  if (exportJob.status === 'completed') {
    // Generate temporary download URL
    const downloadUrl = await s3.getSignedUrl('getObject', {
      Bucket: process.env.S3_BUCKET,
      Key: exportJob.filePath,
      Expires: 3600, // 1 hour
    });

    return res.json({
      exportId: exportJob.id,
      status: 'completed',
      downloadUrl,
      expiresAt: addHours(new Date(), 1),
      fileSize: exportJob.fileSizeBytes,
      rowCount: exportJob.rowCount,
    });
  }

  return res.json({
    exportId: exportJob.id,
    status: exportJob.status,
  });
});
```

---

## Next Steps

- **[Quick Start Guide](./quick-start-guide)** - Basic report generation and export
- **[Advanced Reporting](./advanced-reporting)** - Scheduled reports and custom exports

---

**Last Updated:** December 15, 2025
**Parent:** [Manual Reporting & Data Export](./README)
