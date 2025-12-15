---
title: "User Management & Authentication"
description: "User authentication, profile management, and account security in PenguinMails"
last_modified_date: "2025-12-15"
level: "2"
persona: "All Users, Developers"
status: "ACTIVE"
category: "Enterprise"
---

# User Management & Authentication

Secure user authentication and comprehensive profile management powered by NileDB.

## Overview

PenguinMails provides enterprise-grade user authentication with email/password login, profile management, password security features, and session management - all built on NileDB's secure authentication framework.

### Authentication Features

- **Secure Login** - Email/password authentication via NileDB SDK
- **Profile Management** - Self-service profile editing
- **Password Security** - Forgot/reset/change password workflows
- **Session Management** - Secure token-based sessions
- **Team Management** - Multi-user tenant support
- **Email Verification** - Confirmed email addresses only

## Topic Guides

| Guide | Description |
|-------|-------------|
| [MVP Status & Roadmap](./mvp-status) | Current features and timeline |
| [Authentication](./authentication) | Signup, login, logout, verification |
| [Password Management](./password-management) | Forgot, reset, and change password |
| [Profile Management](./profile-management) | View/edit profile and preferences |
| [Team & Workspace](./team-workspace) | Team members and workspace management |
| [Session Management](./session-management) | JWT tokens and session security |
| [Account Deletion](./account-deletion) | Account deletion and recovery |

---

## Related Documentation

- [Authentication Roadmap](/docs/features/authentication/authentication-roadmap)
- [Multi-Tenant Architecture](/docs/features/infrastructure/multi-tenant-architecture)
- [Security Framework](/docs/compliance-security/enterprise/security-framework)
- [NileDB Authentication](/docs/implementation-technical/database-infrastructure/niledb)

---

**Authentication Provider:** NileDB SDK
**Current Method:** Email + Password
**MVP Status:** In Progress
