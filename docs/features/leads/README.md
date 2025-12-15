---
title: "Leads & Lists"
description: "Lead management, contact lists, and segmentation"
last_modified_date: "2025-12-16"
level: "1"
persona: "All Users"
---

# Leads & Lists

Lead management, contact lists, and segmentation (scoped by tenant).

## Purpose

Organize and manage leads/contacts with tenant-scoped lists and segmentation.

## MVP Status

**Current Status**: Comprehensive documentation exists, MVP simplified to core features
**MVP Completion**: Q1 2026 (3-5 weeks)
**Priority**: P0 (Critical for campaign targeting)

### MVP Gaps (2 Features)

1. **Contact Import/Export Implementation** (P0, 2-3 weeks) - CSV/Excel import with field mapping, validation, duplicate detection

2. **Basic Contact Segmentation** (P0, 1-2 weeks) - Static lists for campaign targeting (dynamic segmentation moved to Post-MVP)

**Rationale**: MVP simplified to essential features only. Complex features (dynamic segmentation, lead scoring, activity timeline, bulk operations) moved to Post-MVP to enable faster time-to-market with core functionality.

## Features in This Category

### Available Now (Documented)

- **[Leads Management](/docs/features/leads/leads-management)** - Contact storage and segmentation

- **[Contact Segmentation](/docs/features/leads/contact-segmentation)** - Dynamic and static contact segmentation

- **[Lead Scoring](/docs/features/leads/lead-scoring)** - Engagement-based lead scoring system

- **[Import/Export](/docs/features/leads/import-export/overview)** - Bulk contact import/export with field mapping

### 🚧 MVP Implementation Required (Q1 2026)

- **Contact Import/Export System** - Complete CSV/Excel import with validation and duplicate detection

- **Basic Static Segmentation** - Manual lists for campaign targeting

### �� Post-MVP Enhancements (Q1-Q3 2026)

- **Dynamic Contact Segmentation** (Q1 2026) - Rule-based filtering with filter builder UI

- **Basic Lead Scoring System** (Q1 2026) - Behavioral scoring based on email engagement

- **Contact Activity Timeline** (Q1 2026) - Chronological interaction history

- **Bulk Contact Operations** (Q1 2026) - Multi-select actions for efficiency

- **Lead Enrichment** (Q1 2026) - Automated lead data enhancement

- **Advanced Lead Scoring Models** (Q2 2026) - Multiple models with custom formulas

- **Contact Deduplication & Merge** (Q2 2026) - Fuzzy matching and manual merge UI

- **Contact Lifecycle Stages** (Q3 2026) - Track contacts through sales funnel

- **Contact Notes & Tasks** (Q3 2026) - Sales follow-up and collaboration

### 🔮 Future Research (Q3 2026+)

- **Third-Party Integrations Research Spike** (Q3-Q4 2026) - Evaluate CRM sync, lead enrichment, advanced email validation

- **Lead Analytics Improvements** (Q4 2027) - Explore advanced analytics after 12+ months of production data

## Related Documentation

### Planning & Strategy

- [Lead Management Roadmap](/docs/features/leads/roadmap) - Detailed timeline and quarterly breakdown

- [Product Roadmap](/docs/operations/roadmap/product-roadmap/overview) - Overall product timeline

- [Executive Roadmap](/docs/business/roadmap/executive-roadmap) - Strategic feature delivery

### Route Specifications

- [Tenant Leads Routes](/docs/design/routes/tenant-leads) - 4 lead management routes with UI specs

### API Documentation

- [Leads API](/docs/implementation-technical/api/tenant-api/leads) - Contact management endpoints

### Implementation Tasks

- Epic 7: Leads Management - Internal task reference for implementation sequencing

### Related Features

- [Campaign Management](/docs/features/campaigns/campaign-management/hub) - Use segments in campaigns

- [Inbox Management](/docs/features/inbox/README) - Reply management linked to contacts

- [Integrations](/docs/features/integrations/README) - CRM integrations (Post-MVP)

---

[← Back to All Features](https://github.com/penguinmails/penguinmails.github.io/blob/main)
