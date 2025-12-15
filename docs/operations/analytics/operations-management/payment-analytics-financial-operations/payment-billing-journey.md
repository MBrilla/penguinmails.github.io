---
title: "Payment & Billing Journey"
description: "Complete payment flow from Stripe Connect setup through subscription activation and ongoing billing management"
last_modified_date: "2025-12-15"
level: "3"
parent: "Payment Analytics & Financial Operations"
---

# Payment & Billing Journey

This guide covers the complete payment and billing user journey from initial setup through ongoing management.

---

## Journey Flow Overview

`Stripe Connect Setup → Payment Method → Subscription Activation → Billing Dashboard`

**Overview:** Payment and billing operations with Stripe Connect integration, subscription management, and financial analytics.

---

## Normal Payment Flow

### 1. Onboarding Payment Setup

**Journey Flow:** `Onboarding Modal → Stripe Connect → Business Verification → Payment Method → Subscription Activation`

#### Onboarding Trigger

- **Context**: User reaches payment setup in onboarding flow
- **Modal Elements**:
  - "Connect Payment Method" section header
  - Stripe Connect integration button
  - Business verification requirements notice
  - Platform fee explanation ($100 → $77 model)
- **User Action**: Clicking "Setup Payments" button

#### Stripe Connect Express Setup

- **Page**: Stripe-hosted onboarding flow (external redirect)
- **Business Verification**:
  - Company legal name and tax ID
  - Business address and phone
  - Bank account or debit card details
  - Identity verification (SSN)
- **Integration**: OAuth connection back to PenguinMails

#### Payment Method Addition

- **Page**: Billing settings (`/billing`)
- **Form Elements**:
  - "Add Payment Method" button
  - Stripe Elements credit card form
  - Billing address collection
  - "Set as Default" checkbox
- **Security**: PCI-compliant card tokenization

#### Subscription Activation

- **Page**: Plan selection (`/billing`)
- **Plan Options**: Freemium → Professional → Enterprise tiers
- **Features**:
  - IP allocation limits
  - Monthly email volume
  - Team member seats
  - Premium support
- **Activation**: Immediate access after payment confirmation

#### Billing Dashboard Access

- **Page**: Main billing overview
- **Dashboard Elements**:
  - Current plan and usage metrics
  - Next billing date and amount
  - Payment history table
  - Invoice download links
  - Plan upgrade/downgrade options

---

### 2. Ongoing Billing Management

**Journey Flow:** `Billing Dashboard → Usage Monitoring → Payment Updates → Plan Changes → Invoice Handling`

#### Usage Tracking & Alerts

- **Dashboard Widgets**:
  - Email volume progress bars
  - IP reputation scores
  - Team member utilization
  - Monthly spending vs. budget
- **Alert System**: 80% usage threshold notifications

#### Payment Method Management

- **Page**: Payment methods settings (`/billing`)
- **Actions**:
  - Add/remove payment methods
  - Update billing addresses
  - Set default payment method
  - Failed payment retry attempts

#### Invoice & Receipt Access

- **Page**: Billing history (`/billing`)
- **Features**:
  - PDF invoice downloads
  - Detailed line items
  - Tax calculations
  - Payment status indicators
  - Email delivery confirmations

---

## Key User Touchpoints

| Stage | Page/Component | Primary Action |
|-------|----------------|----------------|
| Onboarding | `/dashboard/onboarding` | Connect Stripe account |
| Payment Setup | `/billing` | Add payment method |
| Plan Selection | `/billing` | Choose subscription tier |
| Dashboard | `/billing` | Monitor usage and billing |
| History | `/billing` | Download invoices |

---

## Next Steps

- **[Stripe Connect Integration](./stripe-connect-integration)** - Technical implementation details
- **[Edge Cases & Recovery](./edge-cases-recovery)** - Handling payment failures
- **[Setup & Troubleshooting](./setup-troubleshooting)** - Common issues and solutions

---

**Keywords**: payment flow, billing journey, subscription activation, usage tracking, invoice management
