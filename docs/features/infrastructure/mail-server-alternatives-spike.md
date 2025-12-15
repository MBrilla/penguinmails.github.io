---
title: "Spike: Mail Server Alternatives Investigation"
description: "P0 investigation for mail server alternatives that blocks Unified Inbox MVP"
last_modified_date: "2025-12-16"
level: "3"
persona: "Technical Teams"
---

# Spike: Mail Server Alternatives Investigation

**Priority:** P0 (Blocks Unified Inbox MVP)  
**Timeline:** Q4 2025  
**Estimated Effort:** 3-5 days  
**Status:** Not Started

---

## Context

PenguinMails currently uses **Mailu** for SMTP/IMAP operations, but we're evaluating modern alternatives including **Stalwart Mail Server** and others. The unified inbox feature requires understanding how our chosen mail server stores emails and whether we can extend its schema.

**Key Questions:**

1. How does the mail server store emails in PostgreSQL?
2. Can we extend the schema with custom fields (threads, stars, categories)?
3. Where should inbox metadata live (OLTP, Content DB, or mail server schema)?

---

## Candidates to Evaluate

### 1. Mailu (Current)

- Docker-based, full-featured
- Uses standard IMAP/SMTP protocols
- Limited PostgreSQL schema extensibility

### 2. Stalwart Mail Server

- Modern alternative with native PostgreSQL support
- Reference: <https://stalw.art/docs/storage/backends/postgresql>
- Worth investigating for schema extensibility

### 3. Other Alternatives

- **Maddy** - Go-based, modular design
- **Postal** - Ruby-based, designed for transactional email
- **Mail-in-a-Box** - All-in-one solution
- **Haraka** - Node.js SMTP server (plugin architecture)

---

## Spike Goals

1. **Evaluate mail server options** for PenguinMails requirements
2. **Understand database schema** for top candidates
3. **Determine schema extensibility** options
4. **Recommend best architecture** for storing inbox metadata
5. **Create proof-of-concept** to validate approach

---

## Tasks

### 1. Requirements Analysis (1 day)

**Questions to Answer:**

- [ ] What are PenguinMails' specific mail server requirements?
- [ ] Multi-tenant support requirements?
- [ ] Volume expectations (emails/day)?
- [ ] Required features (threading, search, folders, labels)?
- [ ] Integration requirements (webhooks, APIs)?

---

### 2. Candidate Evaluation (2-3 days)

For each candidate:

- [ ] **Architecture:**
  - How does it store emails?
  - What database backends are supported?
  - Multi-tenant support?

- [ ] **Schema Extensibility:**
  - Can we add custom fields?
  - Can we add foreign keys to OLTP?
  - Will custom fields survive upgrades?

- [ ] **Integration:**
  - API availability?
  - Webhook support?
  - IMAP/SMTP compliance?

- [ ] **Performance:**
  - Indexing capabilities?
  - Query performance?
  - Scalability?

**Deliverables:**

- [ ] Comparison matrix of candidates
- [ ] Recommended solution with justification

---

### 3. Architecture Decision (1 day)

Based on evaluation, decide where inbox metadata lives:

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| A | Extend mail server schema | Single source of truth | Tight coupling |
| B | Store in OLTP | Full control | Sync required |
| C | Store in Content DB | Email data together | Violates design |
| D | Hybrid | Balanced | Complex |

---

### 4. Proof-of-Concept (1 day)

- [ ] Set up recommended mail server
- [ ] Implement test integrations
- [ ] Validate key operations
- [ ] Measure performance

---

### 5. Update Documentation (0.5 day)

- [ ] Update inbox feature docs with architecture decision
- [ ] Create implementation tasks for unified inbox

---

## Success Criteria

- [ ] Mail server decision made and justified
- [ ] Architecture for inbox metadata decided
- [ ] Proof-of-concept validates approach
- [ ] No blockers remain for unified inbox development

---

## Related Documentation

- **Routes**: [tenant-inbox.md]()
- **API**: [tenant-api/](/docs/implementation-technical/api/tenant-api/)
- **Features**: [inbox/](/docs/features/inbox/)

---

**Last Updated:** December 2025
