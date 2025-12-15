---
title: "Leads Management Implementation Guide"
description: "Implementation guide for Leads & Contact Management feature"
last_modified_date: "2025-12-16"
level: "3"
persona: "Technical Teams"
---

# Leads Management Implementation Guide

**Feature:** Leads & Contact Management
**Epic:** Epic 7
**Status:** Ready for Implementation

---

## 1. Database Schema (Drizzle ORM)

Define the following tables using Drizzle ORM schema definitions.

```typescript
// db/schema/leads.ts

import { pgTable, uuid, varchar, timestamp } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

// Reference: OLTP Schema Guide - leads table structure
// https://docs/implementation-technical/database-infrastructure/oltp-database/schema-guide.md#leads---contact-database

export const leads = pgTable('leads', {
  id: uuid('id').primaryKey().defaultRandom(),
  tenantId: uuid('tenant_id').notNull(),
  email: varchar('email', { length: 255 }).notNull(),
  name: varchar('name', { length: 200 }),
  company: varchar('company', { length: 200 }),
  status: varchar('status', { length: 20 }).notNull().default('active'),
  updated: timestamp('updated').defaultNow(),
});

// Relations
export const leadsRelations = relations(leads, ({ many }) => ({
  // Future: campaign_leads, lead_activities, etc.
}));


```

---

## 2. API Implementation (Next.js App Router)

### GET /api/v1/contacts

*List contacts with pagination and search.*

```typescript
// app/api/v1/leads/route.ts

import { NextResponse } from 'next/server';
import { getSession } from '@/lib/auth';
import { db } from '@/lib/db';
import { leads } from '@/db/schema/leads';
import { eq, or, ilike, desc, count } from 'drizzle-orm';
import { and } from 'drizzle-orm';

export async function GET(req: Request) {
  const session = await getSession();
  if (!session) return new NextResponse('Unauthorized', { status: 401 });

  const { searchParams } = new URL(req.url);
  const page = parseInt(searchParams.get('page') || '1');
  const limit = parseInt(searchParams.get('limit') || '20');
  const search = searchParams.get('search') || '';

  const skip = (page - 1) * limit;

  // Build search conditions using Drizzle ORM
  const searchConditions = search ? or(
    ilike(leads.email, `%${search}%`),
    ilike(leads.name, `%${search}%`),
    ilike(leads.company, `%${search}%`)
  ) : undefined;

  const whereConditions = and(
    eq(leads.tenantId, session.tenantId),
    searchConditions
  );

  // Execute queries using Drizzle ORM
  const leadResults = await db.select()
    .from(leads)
    .where(whereConditions)
    .orderBy(desc(leads.updated))
    .limit(limit)
    .offset(skip);

  const totalResult = await db.select({ count: count() })
    .from(leads)
    .where(eq(leads.tenantId, session.tenantId));

  const total = totalResult[0].count;

  return NextResponse.json({
    data: leadResults,
    meta: {
      total,
      page,
      limit,
      totalPages: Math.ceil(total / limit),
    }
  });
}


```

### POST /api/v1/contacts

*Create a single contact.*

```typescript
// app/api/v1/leads/route.ts

import { z } from 'zod';
import { db } from '@/lib/db';
import { leads } from '@/db/schema/leads';

const createLeadSchema = z.object({
  email: z.string().email(),
  name: z.string().optional(),
  company: z.string().optional(),
  status: z.string().optional(),
});

export async function POST(req: Request) {
  const session = await getSession();
  if (!session) return new NextResponse('Unauthorized', { status: 401 });

  try {
    const body = await req.json();
    const data = createLeadSchema.parse(body);

    const leadData = {
      ...data,
      tenantId: session.tenantId,
    };

    const [lead] = await db.insert(leads).values(leadData).returning();

    return NextResponse.json(lead, { status: 201 });
  } catch (error) {
    return new NextResponse('Invalid request', { status: 400 });
  }
}


```

---

## 3. Frontend Components

### Contact List Table

*Located at `components/contacts/contact-table.tsx`*

- Use `@tanstack/react-table` for functionality.

- Columns:

  - **Name**: Combine `firstName` + `lastName`

  - **Email**: Link to detail view

  - **Status**: Badge (Green for active, Red for bounced)

  - **Score**: Numeric value

  - **Added**: Date format

  - **Actions**: Edit / Delete dropdown

### Add Contact Modal

*Located at `components/contacts/add-contact-modal.tsx`*

- Form fields:

  - Email (Required)

  - First Name

  - Last Name

  - Custom Attributes (Key/Value pair input)

---

## 4. Next Steps

1. **Database Schema**: Use the existing OLTP database schema (leads table is NileDB-managed)

2. **Scaffold API**: Create the Drizzle ORM route handlers

3. **Build UI**: Implement the leads list view and forms

4. **Test**: Verify tenant isolation (User A cannot see User B's leads)

5. **Generate Migrations**: Apply Drizzle migrations if needed for additional fields
