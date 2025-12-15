---
title: "Public & Authentication Routes"
description: "Secure entry points for users to access the platform, register, and recover accounts"
last_modified_date: "2025-12-16"
level: "2"
persona: "Technical Teams"
---

# Public & Authentication Routes

## 1. Purpose & Context (The "Why")

* **Goal**: Provide secure entry points for users to access the platform, register, and recover accounts.

* **Feature References**:

  * [User Onboarding](/docs/features/getting-started)

  * [Security & Compliance](/docs/compliance-security/enterprise/overview)

* **User Journey**: These are the first touchpoints. The experience must be fast, trustworthy, and clear.

## 2. UI Patterns & Components (The "How")

* **Core Components**:

  * `AuthLayout`: Specialized layout with split screen (Marketing visual left, Form right).

  * `Form`: Use `react-hook-form` + `zod` for validation (see [UI Library](/docs/design/ui-library/overview)).

  * `Button`: Primary variant for "Sign In"/"Register", Ghost variant for "Forgot Password".

* **Analytics Patterns**: N/A (Operational views).

* **Layout**: `PublicLayout` (Minimal header/footer) or `AuthLayout`.

## 3. Route Specifications

| Route | Access | Purpose | State/Data Requirements |
|---|---|---|---|
| `/` | Public | Marketing Landing | **SSG**. Static content. Hero section, Features grid, Pricing table. |
| `/login` | Public | User Sign-in | **Client Component**. Form (Email/Password). Redirects to `/dashboard` on success. |
| `/register` | Public | Account Creation | **Client Component**. Multi-step form: 1. Account Info, 2. Plan Selection. |
| `/forgot-password` | Public | Recovery Request | Form (Email). Triggers backend email service. Shows success message on submit. |
| `/reset-password` | Public | Password Reset | **Client Component**. Requires `?token=xyz`. Form (New Password, Confirm). |
| `/verify-email` | Public | Email Verification | **Server Component**. Validates token from URL. Displays Success/Error state + "Continue to Dashboard" button. |
| `/status` | Public | System Health | **ISR (60s)**. Fetches status from StatusPage or internal monitoring API. |

## 4. Detailed View Descriptions

### `/` - Landing Page

**User Story**: *"As a potential customer, I want to quickly understand what PenguinMails offers and how it can solve my cold email challenges, so I can decide if it's right for my business."*

**What You'll Find**:

* **Hero Section**: Clear value proposition with primary CTA ("Start Free Trial").

* **Features Grid**: 6-8 key features with icons and brief descriptions.

* **Social Proof**: Customer testimonials, logos, success metrics.

* **Pricing Table**: Transparent pricing tiers (linked to billing strategy).

* **Footer**: Links to `/status`, legal pages (on landing site), social media.

**User Journey Context**: First touchpoint. Must build trust and drive sign-ups within 30 seconds.

**Related Documentation**:

* [Marketing Strategy](/docs/business/marketing/overview)

* [Value Propositions](/docs/business/value-propositions/overview)

---

### `/login` - User Sign-in

**User Story**: *"As a returning user, I want to quickly access my account with email/password or SSO, so I can get back to managing my campaigns."*

**What You'll Find**:

* **Email/Password Form**: Standard input fields with client-side validation.

* **Forgot Password Link**: Redirects to `/forgot-password`.

* **Register Link**: "Don't have an account? Sign up".

* **Error Handling**: Inline validation, "Invalid credentials" message.

> [!NOTE]
> **Future OAuth Support**: Google OAuth and Microsoft OAuth are planned for **Q4 2026 post-MVP**. Current implementation uses email/password only.

**User Journey Context**: Daily entry point for active users. Must be fast (<2s load) and secure.

> [!NOTE]
> **Missing MVP Features**: Account lockout after 5 failed attempts, CAPTCHA after 3 failed attempts, "Remember me" functionality. See [Authentication Roadmap](/docs/features/authentication/authentication-roadmap) for timeline.

**Related Documentation**:

* [Authentication Architecture](/docs/technical/architecture/detailed-technical/onboarding-authentication)

* [Authentication Roadmap](/docs/features/authentication/authentication-roadmap)

* [Security Standards](/docs/compliance-security/enterprise/overview)

**Technical Integration**:

* **NileDB SDK** for authentication (provides front-end and back-end functions).

* Email/password authentication only.

* Session stored in HTTP-only cookies.

* CSRF protection enabled.

* **Email Verification**: Loop service handles verification emails.

> [!NOTE]
> See [Onboarding & Authentication](/docs/technical/architecture/detailed-technical/onboarding-authentication) for complete implementation details.

---

### `/register` - Account Creation

**User Story**: *"As a new user, I want to create an account quickly with minimal friction, so I can start setting up my first campaign."*

**What You'll Find**:

* **Step 1: Account Info**

  * Email, Password, Company Name.

* **Step 2: Plan Selection**

  * Radio buttons for Free, Pro, Enterprise.

  * Feature comparison table.

  * "Start Free Trial" or "Continue to Payment".

> [!NOTE]
> **Future OAuth Support**: One-click SSO with Google/Microsoft planned for **Q4 2026 post-MVP**. Current registration uses email/password only.

**User Journey Context**: Critical conversion funnel. Optimize for speed (1-click SSO preferred).

> [!NOTE]
> **Missing MVP Features**: CAPTCHA on registration form, password strength enforcement (server-side), unverified email restrictions. See [Authentication Roadmap](/docs/features/authentication/authentication-roadmap) for timeline.

**Related Documentation**:

* [Onboarding Flow](/docs/technical/architecture/detailed-technical/onboarding-authentication)

* [Authentication Roadmap](/docs/features/authentication/authentication-roadmap)

* [User Personas](/docs/business/user-personas/startup-founders)

**Post-Registration**: Redirect to `/dashboard/onboarding` for workspace setup.

---

### `/forgot-password` - Recovery Request

**User Story**: *"As a user who forgot my password, I want to receive a reset email quickly, so I can regain access to my account."*

**What You'll Find**:

* **Email Input Field**: "Enter your account email".

* **Submit Button**: Triggers backend email service.

* **Success Message**: "Check your email for password reset instructions."

* **Rate Limiting**: Max 3 requests per hour per IP.

**User Journey Context**: Edge case, but critical for user retention.

> [!NOTE]
> **Missing MVP Features**: CAPTCHA after multiple requests, rate limiting (max 3 requests per hour). See [Authentication Roadmap](/docs/features/authentication/authentication-roadmap) for timeline.

**Technical Integration**:

* Email sent via **Loop** service.

* Token expires in 1 hour.

* Logs stored for security audit.

---

### `/reset-password` - Password Reset

**User Story**: *"As a user with a reset link, I want to set a new secure password, so I can regain full access to my account."*

**What You'll Find**:

* **Token Validation**: Checks `?token=xyz` from URL.

* **New Password Form**: Password + Confirm Password fields.

* **Password Strength Meter**: Visual indicator (weak/medium/strong).

* **Success/Error State**:

  * Success → "Password updated! Redirecting to login..."

  * Error → "Invalid or expired token. Request a new reset link."

**User Journey Context**: Final step in password recovery flow.

**Related Documentation**:

* [Security Policies](/docs/compliance-security/enterprise/overview)

---

### `/verify-email` - Email Verification

**User Story**: *"As a new user, I want to verify my email address to complete account setup and access all features."*

**What You'll Find**:

* **Automatic Validation**: Token from URL is validated server-side.

* **Success State**:

  * Checkmark icon.

  * "Your email is verified!"

  * "Continue to Dashboard" button.

* **Error State**:

  * "Verification failed. Token may be expired."

  * "Resend Verification Email" button.

**User Journey Context**: Part of onboarding flow. Sent immediately after `/register`.

**Technical Integration**:

* Token stored in database with 24-hour expiration.

* Verification status gates access to certain features (e.g., sending campaigns).

---

### `/status` - System Health

**User Story**: *"As a user experiencing issues, I want to check if PenguinMails is operational or if there's a known outage, so I know if the problem is on my end."*

**What You'll Find**:

* **Service Status Grid**:

  * API: Operational

  * SMTP Service: Operational

  * Dashboard: Operational

  * Unified Inbox: Degraded Performance

* **Incident History**: Last 7 days of incidents (if any).

* **Metrics**: API Latency (avg. 150ms), Uptime (99.9%).

**User Journey Context**: Accessed during perceived downtime or from email notifications.

**Related Documentation**:

* [Infrastructure Monitoring](/docs/operations/monitoring/system-health)

**Technical Integration**:

* **Incremental Static Regeneration (ISR)**: Revalidate every 60 seconds.

* Data source: Internal monitoring API or StatusPage integration.

## 5. Related API Endpoints

| Route | Related API / Service | Description |
|---|---|---|
| `/login` | **NileDB SDK** | Handles authentication, session management, and cookie setting. |
| `/register` | **NileDB SDK** | Handles user creation and initial tenant setup. |
| `/forgot-password` | **Loop Service** | Triggers password reset emails via internal service. |
| `/status` | `GET /api/v1/status` | (Internal) System health check endpoint. |

## 6. Data Strategy

* **Fetching Method**:

  * Landing Page (`/`): **Static Site Generation (SSG)**. Built at deploy time.

  * Auth Pages: **Client-side** only. No initial data fetch required (except session check).

  * Status Page: **Incremental Static Regeneration (ISR)** with 60s revalidation.

* **Caching**:

  * Public assets cached via CDN (Cloudflare).

  * API responses for status check cached for 60s.

* **State Management**:

  * Auth state managed via `NileDB` provider context.

  * Form state managed via `react-hook-form`.

## 7. Edge Cases & Error Handling

* **Already Logged In**: Accessing `/login` or `/register` while authenticated redirects to `/dashboard`.

* **Invalid Token**: `/verify-email` or `/reset-password` with expired/invalid token shows clear error message + "Request New Link" button.

* **Rate Limiting**: Too many login attempts triggers 429 Too Many Requests. Show "Try again in X minutes".

* **Network Offline**: Forms disable submission and show "No internet connection" toast.

## 8. Component Architecture

### Page Components

* **`AuthLayout`** (Server)

  * Features: Split screen (Marketing Image | Form), Logo placement.

* **`LoginForm` / `RegisterForm`** (Client)

  * Features: Zod validation, password visibility toggle, error handling.

* **`StatusPage`** (Server - ISR)

  * Sub-components: `SystemStatusItem` (API, SMTP, DB).

### Shared Components

* **`SocialButton`**: For future OAuth providers.

* **`PasswordField`**: Reused in Settings.
