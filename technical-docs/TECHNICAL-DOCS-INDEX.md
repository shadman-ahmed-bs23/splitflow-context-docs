# SplitFlow Technical Documentation Index

**Last Updated:** January 2026  
**Total Technical Docs:** 18  
**Document Version:** 1.0

This index tracks all technical documentation for each feature, providing quick access to API specifications, database schemas, implementation details, and coding guidelines.

---

## Index Legend

- **Category:** Auth | Group Management | Financial Operations | Notifications | Reporting | UI
- **Tech Stack:** Backend (Laravel) | Frontend (Nuxt 3) | Database (MySQL)

---

## Quick Statistics

| Category | Count | Backend | Frontend | Database |
|----------|-------|---------|----------|----------|
| Authentication & User Management | 2 | 2 | 0 | 2 |
| Group Management | 4 | 4 | 0 | 4 |
| Financial Operations | 6 | 6 | 0 | 6 |
| Notifications & Communication | 1 | 1 | 1 | 1 |
| Reporting & Analytics | 2 | 2 | 0 | 2 |
| User Interface | 2 | 0 | 2 | 0 |
| **Total** | **18** | **15** | **3** | **15** |

---

## Technical Documentation by Category

### Authentication & User Management

| Feature ID | Technical Doc | API Endpoints | Database Tables | Key Services |
|------------|---------------|---------------|-----------------|--------------|
| AUTH-001 | [AUTH-001-technical.md](./auth/AUTH-001-technical.md) | 2 | users, otp_requests | RegistrationService |
| AUTH-002 | [AUTH-002-technical.md](./auth/AUTH-002-technical.md) | 3 | users, otp_requests, oauth_* | LoginService |

---

### Group Management

| Feature ID | Technical Doc | API Endpoints | Database Tables | Key Services |
|------------|---------------|---------------|-----------------|--------------|
| GRP-001 | [GRP-001-technical.md](./group-management/GRP-001-technical.md) | 3 | groups, group_members | GroupService |
| GRP-002 | [GRP-002-technical.md](./group-management/GRP-002-technical.md) | 3 | group_invitations, group_members | InvitationService |
| GRP-003 | [GRP-003-technical.md](./group-management/GRP-003-technical.md) | 3 | group_members, audit_logs | RoleManagementService |
| GRP-004 | [GRP-004-technical.md](./group-management/GRP-004-technical.md) | 2 | groups, group_members, user_group_activity | GroupListService |

---

### Financial Operations

| Feature ID | Technical Doc | API Endpoints | Database Tables | Key Services |
|------------|---------------|---------------|-----------------|--------------|
| FIN-001 | [FIN-001-technical.md](./financial-operations/FIN-001-technical.md) | 1 | transactions, groups | DepositService |
| FIN-002 | [FIN-002-technical.md](./financial-operations/FIN-002-technical.md) | 1 | transactions, expense_splits | ExpenseService |
| FIN-003 | [FIN-003-technical.md](./financial-operations/FIN-003-technical.md) | 3 | transactions, expense_splits, group_members | ExpenseApprovalService |
| FIN-004 | [FIN-004-technical.md](./financial-operations/FIN-004-technical.md) | 1 | groups, transactions | BalanceService |
| FIN-005 | [FIN-005-technical.md](./financial-operations/FIN-005-technical.md) | 1 | group_members, transactions, expense_splits | IndividualBalanceService |
| FIN-006 | [FIN-006-technical.md](./financial-operations/FIN-006-technical.md) | 1 | group_members | SettlementService |

---

### Notifications & Communication

| Feature ID | Technical Doc | API Endpoints | Database Tables | Key Services/Composables |
|------------|---------------|---------------|-----------------|--------------------------|
| NOTIF-001 | [NOTIF-001-technical.md](./notifications/NOTIF-001-technical.md) | 2 | user_group_activity, groups | UnreadCountService, useRealtime |

---

### Reporting & Analytics

| Feature ID | Technical Doc | API Endpoints | Database Tables | Key Services |
|------------|---------------|---------------|-----------------|--------------|
| RPT-001 | [RPT-001-technical.md](./reporting/RPT-001-technical.md) | 2 | transactions, expense_splits | ReportService |
| RPT-002 | [RPT-002-technical.md](./reporting/RPT-002-technical.md) | 1 | transactions, expense_splits | PersonalDashboardService |

---

### User Interface

| Feature ID | Technical Doc | Frontend Files | Key Composables/Components |
|------------|---------------|----------------|----------------------------|
| UI-001 | [UI-001-technical.md](./ui/UI-001-technical.md) | layouts, components, styles | Layout components, responsive utilities |
| UI-002 | [UI-002-technical.md](./ui/UI-002-technical.md) | plugins, composables | useRealtime, useGroupBalance, useUnreadCounts |

---

## Database Schema Overview

### Core Tables
1. **users** - User accounts (AUTH-001)
2. **otp_requests** - OTP verification (AUTH-001, AUTH-002)
3. **groups** - Group entities (GRP-001)
4. **group_members** - Group membership (GRP-001)
5. **group_invitations** - Member invitations (GRP-002)
6. **audit_logs** - System audit trail (GRP-003)
7. **transactions** - Deposits and expenses (FIN-001, FIN-002)
8. **expense_splits** - Expense allocations (FIN-002, FIN-003)
9. **user_group_activity** - Unread tracking (NOTIF-001, GRP-004)

### OAuth Tables (Laravel Passport)
- oauth_access_tokens
- oauth_clients
- oauth_personal_access_clients

---

## API Endpoint Summary

### Authentication
- `POST /api/v1/auth/register/request-otp`
- `POST /api/v1/auth/register/verify-otp`
- `POST /api/v1/auth/login/request-otp`
- `POST /api/v1/auth/login/verify-otp`
- `POST /api/v1/auth/logout`

### Groups
- `POST /api/v1/groups`
- `PATCH /api/v1/groups/{groupId}`
- `POST /api/v1/groups/{groupId}/archive`
- `GET /api/v1/groups`
- `GET /api/v1/groups/{groupId}`
- `POST /api/v1/groups/{groupId}/invitations`
- `POST /api/v1/invitations/{token}/accept`
- `POST /api/v1/invitations/{token}/decline`
- `POST /api/v1/groups/{groupId}/members/{memberId}/promote`
- `POST /api/v1/groups/{groupId}/members/{memberId}/revoke-admin`
- `GET /api/v1/groups/{groupId}/members`
- `POST /api/v1/groups/{groupId}/mark-seen`

### Financial Operations
- `POST /api/v1/groups/{groupId}/deposits`
- `POST /api/v1/groups/{groupId}/expenses`
- `GET /api/v1/groups/{groupId}/expenses/pending`
- `POST /api/v1/expenses/{expenseId}/approve`
- `POST /api/v1/expenses/{expenseId}/reject`
- `GET /api/v1/groups/{groupId}/balance`
- `GET /api/v1/groups/{groupId}/members/me/balance`
- `GET /api/v1/groups/{groupId}/settlement`

### Notifications
- `GET /api/v1/groups/unread-counts`
- `POST /api/v1/groups/{groupId}/mark-seen`

### Reporting
- `POST /api/v1/groups/{groupId}/reports`
- `GET /api/v1/groups/{groupId}/reports/{reportId}/pdf`
- `GET /api/v1/groups/{groupId}/members/me/dashboard`

---

## Key Services & Actions

### Backend Services
- `RegistrationService` - User registration
- `LoginService` - User authentication
- `GroupService` - Group management
- `InvitationService` - Member invitations
- `RoleManagementService` - Admin role management
- `GroupListService` - Group listing
- `DepositService` - Deposit submission
- `ExpenseService` - Expense submission
- `ExpenseApprovalService` - Expense approval workflow
- `BalanceService` - Main balance calculation
- `IndividualBalanceService` - Individual balance calculation
- `SettlementService` - Settlement calculation
- `UnreadCountService` - Unread activity tracking
- `ReportService` - Spending reports
- `PersonalDashboardService` - Personal dashboard

### Frontend Composables
- `useRealtime()` - WebSocket connection management
- `useGroupBalance()` - Real-time balance updates
- `useUnreadCounts()` - Unread count tracking
- `usePolling()` - Polling fallback

---

## Technology Stack Reference

### Backend (Laravel 10+)
- **Framework:** Laravel 10+
- **Auth:** Laravel Passport
- **Database:** MySQL 8.0+
- **Queue:** Redis
- **Broadcasting:** Laravel Broadcasting (Pusher/Socket.io)
- **PDF:** DomPDF or Snappy

### Frontend (Nuxt 3)
- **Framework:** Nuxt 3 (SSR)
- **Language:** TypeScript
- **Styling:** Tailwind CSS or SCSS
- **WebSocket:** Socket.io client
- **Charts:** Chart.js
- **Image:** Nuxt Image

---

## Coding Standards

### Backend (Laravel)
- PSR-12 coding standard
- Laravel Pint for formatting
- Strict typing where possible
- Service layer pattern
- Action classes for use cases
- Policy-based authorization

### Frontend (Nuxt 3)
- TypeScript strict mode
- Composition API with `<script setup>`
- ESLint + Prettier
- Component-based architecture
- Composables for reusable logic

---

## Testing Strategy

### Backend
- PHPUnit for unit/feature tests
- HTTP tests for API endpoints
- Database transactions for isolation
- Factories for test data

### Frontend
- Vitest for unit tests
- Testing Library for components
- Playwright/Cypress for E2E
- Visual regression testing

---

## Related Documentation

- [Feature Index](../features/FEATURE-INDEX.md) - Product feature specifications
- [Architecture Guide](../technical-specs.md) - System architecture
- Laravel Documentation
- Nuxt 3 Documentation

---

## How to Use This Index

1. **Find Technical Spec:** Use the tables above to locate feature technical docs
2. **API Reference:** Check API Endpoint Summary section
3. **Database Schema:** Review Database Schema Overview
4. **Implementation:** Refer to Key Services & Actions
5. **Frontend:** Check UI technical docs for composables and components

---

**Last Updated:** January 2026  
**Next Review:** As features are implemented
