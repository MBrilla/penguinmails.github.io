---
title: "Settings & Organization"
description: "Manage user profile, organization details, billing, and team access"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Settings & Organization

## 1. Purpose & Context (The "Why")

- **Goal**: Manage user profile, organization details, billing, and team access.

- **Feature References**:

  - [Security](/docs/compliance-security/enterprise/overview)

- **User Journey**: Infrequent access for configuration and maintenance.

## 2. UI Patterns & Components (The "How")

- **Core Components**:

  - `VerticalNav`: Sidebar for settings categories.

  - `Form`: Standard forms with validation.

  - `AvatarUpload`: Image cropper/uploader.

  - `BillingTable`: List of invoices/payment methods.

- **Analytics Patterns**: N/A.

- **Layout**: Global Context (Settings Layout).

## 3. Route Specifications

| Route | Access | Purpose | State/Data Requirements |
|---|---|---|---|
| `/dashboard/settings/profile` | All | User Profile | Form: Name, Email, Password Change, 2FA Setup. |
| `/dashboard/settings/organization` | Admin | Org Details | Form: Company Name, Logo, Timezone. |
| `/dashboard/settings/team` | Admin | Team Mgmt | List of users. Invite User modal (Email + Role). Role modification. |
| `/dashboard/settings/billing` | Admin | Billing | Stripe integration. Current Plan, Credit Card update, Invoice history. |
| `/dashboard/settings/infrastructure` | Admin | Dedicated Infra | Advanced settings for dedicated IP pools (if applicable). |

## 4. Detailed View Descriptions

### `/dashboard/settings/profile` - User Profile Settings

**User Story**: *"As a user, I want to update my personal information and security settings, so I can keep my account secure and up-to-date."*

**What You'll Find**:

**Settings Sections**:

#### Personal Information

- **Avatar**:

  - **Upload Button**: Opens file picker or drag-and-drop.

  - **Cropper**: Circular crop for profile picture.

  - **Remove Button**: Reverts to initials.

- **Name**: Input field (First + Last).

- **Email**: Display only (with "Change Email" button triggering verification flow).

- **Timezone**: Dropdown (auto-detected, editable).

#### Security

- **Password**:

  - **"Change Password" Button**: Expands form.

  - Fields: Current Password, New Password, Confirm New Password.

  - **Password Strength Indicator**.

- **Two-Factor Authentication (2FA)**:

  - **Status**: "Enabled" or "Disabled".

  - **"Enable 2FA" ButtonButton**: Opens QR code modal for authenticator app.

  - **Backup Codes**: Download once 2FA enabled.

#### Notification Preferences

- **Email Notifications**:

  - Toggles for "New Replies", "Campaign Completed", "Weekly Summary".

- **In-app Notifications**:

  - Toggle for browser notifications (requires permission).

#### Appearance & Accessibility

- **Theme**:

  - **Toggle**: Light, Dark, System preference

  - **System**: Follows OS/browser preference automatically

  - **Accessibility Note**: Ensures proper contrast ratios for all theme variants

  - **Keyboard Navigation**: Theme switching available via keyboard shortcuts

**Related Accessibility Documentation**:

- [Accessibility Guidelines](/docs/design/accessibility-guidelines) - Comprehensive WCAG 2.1 AA compliance standards

- [Accessibility Guidelines: Color and Contrast](/docs/design/accessibility-guidelines#color-and-contrast) - Theme contrast requirements

- [Accessibility Guidelines: Reduced Motion](/docs/design/accessibility-guidelines#timing-and-movement) - Animation preferences

**Save Button**: Sticky at bottom, saves all changes.

**User Journey Context**: Occasional updates. Critical for account security.

> [!NOTE]
> **Missing MVP Features**:
>
> - Session Management UI (view/revoke active sessions)
> - Email Change Verification Flow UI
> - Login Activity Log
> - Password strength enforcement (server-side)
>
> See [Authentication Roadmap](/docs/features/authentication/authentication-roadmap) for timeline.

**Related Documentation**:

- [Account Security](/docs/compliance-security/enterprise/overview)

- [Authentication Roadmap](/docs/features/authentication/authentication-roadmap)

- [2FA Setup Guide](/docs/operations/freelancer-management/support/tutorials/2fa-setup)

**Technical Integration**:

- **2FA**: TOTP (Time-based One-Time Password) using libraries like `otplib`.

- **Password Hashing**: Bcrypt with salt.

---

### `/dashboard/settings/organization` - Organization Settings

**User Story**: *"As an admin, I want to configure organization-wide settings like company name and logo, so our workspace reflects our brand."*

**What You'll Find**:

#### Company Information

- **Company Name**: Input field.

- **Logo**:

  - **Upload** (Recommended: 200x200px).

  - Shown in Sidebar, Email footers.

- **Industry**: Dropdown (SaaS, E-commerce, etc.) for analytics segmentation.

- **Company Size**: Dropdown (1-10, 11-50, 51-200, etc.).

#### Regional Settings

- **Timezone**: Affects campaign sending windows.

- **Date Format**: MM/DD/YYYY or DD/MM/YYYY.

- **Currency**: For billing display (USD, EUR, GBP, etc.).

#### Domain Verification (if applicable)

- **Custom Domain**: For white-label agencies.

- **Verification Status**: Shows DNS records to add.

**Save Button**: Bottom-right.

**User Journey Context**: Set once during onboarding, rarely changed.

**Related Documentation**:

- [Organization Setup](/docs/business/onboarding/organization-config)

---

### `/dashboard/settings/team` - Team Management

**User Story**: *"As an admin, I want to invite team members and control their access levels, so we can collaborate securely."*

**What You'll Find**:

#### Team Members Table

- **Columns**: Name, Email, Role, Status, Last Active, Actions.

- **Roles**:

  - **Owner**: Full access, cannot be removed.

  - **Admin**: Manage settings, billing, team.

  - **Member**: Access campaigns, leads, inbox.

- **Status**:

  - **Active**: Currently logged in recently.

  - **Invited**: Pending invitation acceptance.

- **Actions Per Row**:

  - **Change Role**: Dropdown.

  - **Remove**: Confirmation modal.

#### Invite Team Member

- **"+ Invite Member" Button** (top-right).

- **Modal**:

  - **Email**: Input field.

  - **Role**: Dropdown (Admin, Member).

  - **Message** (optional): Personal note in invitation email.

  - **Send Invitation** button.

**User Journey Context**: Collaboration setup. Frequency depends on team size.

> [!NOTE]
> **Missing MVP Features**:
>
> - Workspace management feature documentation
> - Workspace health scoring system
> - RBAC permission matrix documentation
> - Team member removal workflow documentation
> - Workspace member management documentation
>
> See [Team Management Roadmap](/docs/features/teams/team-roadmap) for timeline.

**Related Documentation**:

- [Team Management & Authentication](/docs/features/authentication/user-management)

- [Team Management Roadmap](/docs/features/teams/team-roadmap)

- [Multi-Tenant Architecture](/docs/features/infrastructure/multi-tenant-architecture)

- [Team Roles & Permissions](/docs/business/collaboration/team-roles)

- [RBAC Implementation](/docs/technical/security/rbac)

**Technical Integration**:

- **Invitation Emails**: Sent via **Loop.so** (transactional email service).

- **Magic Links**: Click link → Auto-login → Set password.

---

### `/dashboard/settings/billing` - Billing & Subscriptions

**User Story**: *"As an owner, I want to manage my subscription and view invoices, so I can control costs and maintain service."*

**What You'll Find**:

#### Current Plan

- **Plan Name**: "Pro Plan".

- **Limits**: "10 Workspaces, 50,000 Emails/month".

- **Usage This Month**:

  - Progress bar: "35,000 / 50,000 emails sent (70%)".

- **"Upgrade Plan" Button**: Opens pricing modal.

- **"Cancel Subscription" Button**: Opens feedback form + confirmation.

#### Payment Method

- **Card on File**: "**** **** **** 1234 (Exp: 05/26)".

- **"Update Payment Method" Button**: Opens Stripe Customer Portal.

#### Billing History

- **Invoices Table**:

  - Columns: Date, Amount, Status, Download.

  - **Status**: Paid (Green), Pending (Yellow), Failed (Red).

  - **Download**: PDF invoice link.

**User Journey Context**: Monthly check-in or when issues arise.

> [!NOTE]
> **Missing MVP Features**:
>
> - Plan comparison modal for upgrades/downgrades
> - Real-time usage progress bars with color-coded alerts
> - Direct PDF invoice download (currently redirects to Stripe portal)
> - In-app payment method updates (currently uses Stripe portal)
> - Failed payment recovery banner with one-click retry
> - Cancellation flow with retention offers
> - Budget controls and spending limits
>
> See [Billing & Subscriptions Roadmap](/docs/features/payments/roadmap) for timeline.

**Related Documentation**:

- [Billing & Subscriptions Overview](/docs/features/payments/README)

- [Subscription Management](/docs/features/payments/subscription-management)

- [Stripe Integration](/docs/features/payments/stripe-integration)

- [Billing Dashboard](/docs/features/payments/billing-dashboard)

- [Billing Roadmap](/docs/features/payments/roadmap)

**Technical Integration**:

- **Stripe Checkout**: For plan upgrades and subscription creation.

- **Stripe Customer Portal**: For payment method updates, invoice downloads (planned to be replaced with in-app UI in Q1 2026).

- **Stripe Elements**: For in-app payment method updates (Q1 2026).

- **Webhooks**: `checkout.session.completed`, `invoice.paid`, `invoice.payment_failed`, `customer.subscription.updated` events update local database.

---

### `/dashboard/settings/infrastructure` - Infrastructure Management (Advanced)

**User Story**: *"As an enterprise customer with dedicated infrastructure, I want to monitor my IP pool and server health, so I can ensure deliverability."*

**What You'll Find**:

> **Note**: This section only appears for enterprise plans with dedicated infrastructure.

#### Dedicated IP Pool

- **IP List Table**:

  - Columns: IP Address, Reputation Score, Blacklist Status, Daily Volume, Actions.

  - **Reputation Score**: 0-100 with color coding.

  - **Blacklist Status**: Shows if IP is on any blacklists (links to removal instructions).

- **"Request Additional IP" Button**: Opens support form.

#### SMTP Server Health

- **Server Status**: Uptime, Load Average, Queue Size.

- **Alerts**: "IP 203.0.113.5 blacklisted on Spamhaus. Action required."

**User Journey Context**: Rarely accessed. Critical for deliverability troubleshooting.

**Related Documentation**:

- [Dedicated Infrastructure](/docs/technical/infrastructure/dedicated-ip-pools)

- [Deliverability Monitoring](/docs/operations/monitoring/email-health)

**Technical Integration**:

- **Real-time Metrics**: Fetched from internal monitoring API (Prometheus/Grafana) (Future/2026 Spike).

- **Blacklist Checks**: Daily cron job queries MXToolbox, Spamhaus APIs.

## Related API Endpoints

| Route | Related API | Description |
|---|---|---|
| `/settings/profile` | **NileDB SDK** | `updateUser()` for profile changes. |
| `/settings/organization` | [Workspaces API](/docs/implementation-technical/api/tenant-api/workspaces) | `PATCH /api/v1/tenant/workspaces/{id}` (Update Org details). |
| `/settings/team` | [Workspaces API](/docs/implementation-technical/api/tenant-api/workspaces) | `GET /api/v1/tenant/workspaces/{id}/members` (List), `POST` (Invite). |
| `/settings/billing` | [Subscriptions API](/docs/implementation-technical/api/platform-api/subscriptions) | `GET /api/v1/platform/subscriptions/tenant/{id}` (Plan details). |

## 6. Data Strategy

- **Fetching Method**:

  - **Profile/Org/Team**: Server Components.

  - **Billing**: Server Component (proxies Stripe data).

- **Caching**:

  - **Profile**: Cached for session.

  - **Billing**: No caching (must be accurate).

- **Security**:

  - **Sensitive Data**: API keys and billing info never stored in client state/localStorage.

  - **Re-authentication**: Critical actions (change password, delete workspace) require sudo mode (re-enter password).

## 7. Edge Cases & Error Handling

- **Stripe Session Expired**: If billing portal link expires, show "Session expired. Refresh page to continue."

- **Email Already Taken**: When updating profile email, if email exists, show "Email already in use."

- **Payment Method Failed**: If card declined during plan upgrade, show Stripe error message + "Update payment method" link.

- **Downgrade Restrictions**: If downgrading would exceed new plan limits (e.g., 10 workspaces → 3), block and show "Remove 7 workspaces first."

## 8. Component Architecture

### Page Components

- **`ProfileForm`** (Client)

  - Features: Avatar upload, password change (requires re-auth).

- **`TeamList`** (Server)

  - Props: `members: Member[]`

  - Sub-components: `MemberRow`, `InviteModal`.

- **`BillingPortalEmbed`** (Client)

  - Features: Iframe or redirect handler for Stripe Customer Portal.

### Shared Components

- **`AvatarUpload`**: Reused in Workspace Settings.

- **`PlanBadge`**: Reused in Topbar and Admin.
