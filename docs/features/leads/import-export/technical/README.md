---
title: "Technical Implementation"
description: "API integration and batch processing implementation for contact import/export"
last_modified_date: "2025-01-20"
level: 3
persona: Developers
keywords: import, export, API, batch processing, technical implementation
---

# Import/Export Technical Implementation

This section provides the technical architecture and implementation details for the contact import/export system. The documentation covers database design, processing services, background jobs, and API endpoints.

## Architecture Components

The import/export system consists of four primary technical components that work together to handle bulk contact operations.

**[Database Schema](./database-schema.md)** defines the PostgreSQL tables tracking import and export jobs. The schema includes import_jobs, import_events (for rollback support), and export_jobs tables with status tracking and error logging.

**[Import Service](./import-service.md)** contains the TypeScript class responsible for parsing files, validating contacts, handling duplicates, and processing rows. The service supports CSV, Excel, and JSON formats with field mapping and rollback capabilities.

**[Export Service](./export-service.md)** handles contact export with filtering, field selection, and file generation. Exports are stored temporarily in S3 with automatic cleanup after 7 days.

**[Background Jobs & API](./background-jobs-api.md)** documents the queue processing for async operations, cleanup cron jobs, and REST API endpoints for managing import/export operations.

## Data Flow

Import jobs are created via API, queued for background processing, and track progress as rows are processed. Export jobs similarly queue for processing and generate downloadable files with signed URLs.

## Related Documentation

- [Leads Management](/docs/features/leads/leads-management) - Contact database and management
- [Contact Segmentation](/docs/features/leads/contact-segmentation) - Create segments from imported contacts
- [Campaign Management](/docs/features/campaigns/campaign-management/hub) - Use imported contacts in campaigns
