---
title: "Authentication & User Management"
description: "Tenant user authentication and profile management"
last_modified_date: "2025-12-16"
level: "1"
persona: "All Users"
---

# Authentication & User Management

**Target Users**: Tenant users (owner, manager, employee)  
**Customer-Facing**: Yes - These features power tenant user authentication and profile management

## Overview

Authentication and user management features for PenguinMails tenant users. These features enable secure access, profile management, and onboarding experiences for customers using the platform.

## Features in This Category

### ✅ Available Now (MVP)

- **[User Management](/docs/features/authentication/user-management)** - Authentication, profiles, and account management
  - Email/password authentication via NileDB SDK
  - Email verification and password reset
  - JWT-based session management
  - Profile management
  - Role-based access within tenant (owner/manager/employee)

### ⏳ In Development (Q1 2026)

- **[Onboarding Experience](/docs/features/authentication/onboarding-experience)** - Guided setup wizard
  - Interactive checklist
  - Contextual tooltips
  - Video tutorials
  - Achievement milestones

- **Account Security Enhancements**
  - Account lockout protection
  - Session management UI
  - Login activity log
  - Password strength enforcement
  - Email change verification flow

### 📜 Coming Soon (Post-MVP)

#### Q2 2026

- **Social Login (OAuth)** - Sign in with Google, GitHub, Microsoft

#### Q4 2026  

- **Two-Factor Authentication (2FA)** - TOTP-based 2FA with authenticator apps

#### Q1 2027

- **Biometric Authentication** - WebAuthn/FIDO2 for passwordless login

## Roadmap

See **[Authentication Roadmap](/docs/features/authentication/authentication-roadmap)** for detailed timeline and quarterly breakdown.

## Related Documentation

### Route Specifications

- [Public & Authentication Routes](/docs/design/routes/public-auth) - Login, signup, password reset UI
- [Settings Routes](/docs/design/routes/settings) - Profile and security settings

### API Documentation

- [Authentication API](/docs/implementation-technical/api/tenant-api/auth) - Login, signup, password reset endpoints

### Related Features

- [Teams](/docs/features/teams/) - Team and workspace management for tenant organizations
- [Multi-Tenant Architecture](/docs/features/infrastructure/multi-tenant-architecture) - Tenant isolation

---

[← Back to All Features]()
