# SplitFlow Feature Index

**Last Updated:** January 2026  
**Total Features:** 18  
**Document Version:** 1.0

This index tracks all features extracted from the Product Specification document. Use this file to quickly locate features, track dependencies, and monitor development progress.

---

## Index Legend

- **Status:** Draft | In Progress | Review | Completed | Blocked
- **Priority:** P0 (Critical) | P1 (High) | P2 (Medium) | P3 (Low)
- **Category:** Auth | Group Management | Financial Operations | Notifications | Reporting | UI

---

## Quick Statistics

| Category | Count | P0 | P1 | P2 | P3 |
|----------|-------|----|----|----|----|
| Authentication & User Management | 2 | 2 | 0 | 0 | 0 |
| Group Management | 4 | 4 | 0 | 0 | 0 |
| Financial Operations | 6 | 5 | 1 | 0 | 0 |
| Notifications & Communication | 1 | 1 | 0 | 0 | 0 |
| Reporting & Analytics | 2 | 0 | 2 | 0 | 0 |
| User Interface | 2 | 2 | 0 | 0 | 0 |
| **Total** | **18** | **14** | **3** | **0** | **0** |

---

## Feature List by Category

### Authentication & User Management

| Feature ID | Feature Name | Priority | Status | File Path | Dependencies |
|------------|---------------|----------|--------|-----------|--------------|
| AUTH-001 | User Registration | P0 | Draft | [features/auth/AUTH-001-user-registration.md](./auth/AUTH-001-user-registration.md) | Email service, Database |
| AUTH-002 | User Login | P0 | Draft | [features/auth/AUTH-002-user-login.md](./auth/AUTH-002-user-login.md) | AUTH-001, Email service |

---

### Group Management

| Feature ID | Feature Name | Priority | Status | File Path | Dependencies |
|------------|---------------|----------|--------|-----------|--------------|
| GRP-001 | Group Creation | P0 | Draft | [features/group-management/GRP-001-group-creation.md](./group-management/GRP-001-group-creation.md) | AUTH-002 |
| GRP-002 | Member Invitation | P0 | Draft | [features/group-management/GRP-002-member-invitation.md](./group-management/GRP-002-member-invitation.md) | GRP-001, NOTIF-001 |
| GRP-003 | Admin Role Management | P0 | Draft | [features/group-management/GRP-003-admin-role-management.md](./group-management/GRP-003-admin-role-management.md) | GRP-001, GRP-002, NOTIF-001 |
| GRP-004 | Group List View | P0 | Draft | [features/group-management/GRP-004-group-list-view.md](./group-management/GRP-004-group-list-view.md) | GRP-001, FIN-001, FIN-002, NOTIF-001, UI-001 |

---

### Financial Operations

| Feature ID | Feature Name | Priority | Status | File Path | Dependencies |
|------------|---------------|----------|--------|-----------|--------------|
| FIN-001 | Deposit Submission | P0 | Draft | [features/financial-operations/FIN-001-deposit-submission.md](./financial-operations/FIN-001-deposit-submission.md) | GRP-001, GRP-003, FIN-004, FIN-005, NOTIF-001 |
| FIN-002 | Expense Submission | P0 | Draft | [features/financial-operations/FIN-002-expense-submission.md](./financial-operations/FIN-002-expense-submission.md) | GRP-001, FIN-003, FIN-004 |
| FIN-003 | Expense Approval Workflow | P0 | Draft | [features/financial-operations/FIN-003-expense-approval-workflow.md](./financial-operations/FIN-003-expense-approval-workflow.md) | FIN-002, FIN-004, FIN-005, NOTIF-001 |
| FIN-004 | Main Balance Calculation | P0 | Draft | [features/financial-operations/FIN-004-main-balance-calculation.md](./financial-operations/FIN-004-main-balance-calculation.md) | FIN-001, FIN-002, FIN-003, UI-002 |
| FIN-005 | Individual Balance Calculation | P0 | Draft | [features/financial-operations/FIN-005-individual-balance-calculation.md](./financial-operations/FIN-005-individual-balance-calculation.md) | FIN-001, FIN-003, FIN-004, UI-002 |
| FIN-006 | Settlement Calculation | P1 | Draft | [features/financial-operations/FIN-006-settlement-calculation.md](./financial-operations/FIN-006-settlement-calculation.md) | FIN-005, FIN-004, UI-001 |

---

### Notifications & Communication

| Feature ID | Feature Name | Priority | Status | File Path | Dependencies |
|------------|---------------|----------|--------|-----------|--------------|
| NOTIF-001 | Real-Time Notifications | P0 | Draft | [features/notifications/NOTIF-001-real-time-notifications.md](./notifications/NOTIF-001-real-time-notifications.md) | All features, Email service, WebSocket server |

---

### Reporting & Analytics

| Feature ID | Feature Name | Priority | Status | File Path | Dependencies |
|------------|---------------|----------|--------|-----------|--------------|
| RPT-001 | Spending Reports | P1 | Draft | [features/reporting/RPT-001-spending-reports.md](./reporting/RPT-001-spending-reports.md) | FIN-001, FIN-002, FIN-003, GRP-003 |
| RPT-002 | Individual Balance Tracking | P1 | Draft | [features/reporting/RPT-002-individual-balance-tracking.md](./reporting/RPT-002-individual-balance-tracking.md) | FIN-001, FIN-002, FIN-005 |

---

### User Interface

| Feature ID | Feature Name | Priority | Status | File Path | Dependencies |
|------------|---------------|----------|--------|-----------|--------------|
| UI-001 | Mobile-Responsive Design | P0 | Draft | [features/ui/UI-001-mobile-responsive-design.md](./ui/UI-001-mobile-responsive-design.md) | All features |
| UI-002 | Real-Time Updates | P0 | Draft | [features/ui/UI-002-real-time-updates.md](./ui/UI-002-real-time-updates.md) | All features, WebSocket server |

---

## Feature Dependency Graph

### Level 0 (Foundation - No dependencies on other features)
- AUTH-001: User Registration
- AUTH-002: User Login (depends on AUTH-001)

### Level 1 (Core Group Features)
- GRP-001: Group Creation (depends on AUTH-002)
- GRP-002: Member Invitation (depends on GRP-001)
- GRP-003: Admin Role Management (depends on GRP-001, GRP-002)

### Level 2 (Financial Operations)
- FIN-001: Deposit Submission (depends on GRP-001, GRP-003)
- FIN-002: Expense Submission (depends on GRP-001)
- FIN-004: Main Balance Calculation (depends on FIN-001, FIN-002, FIN-003)
- FIN-005: Individual Balance Calculation (depends on FIN-001, FIN-003, FIN-004)

### Level 3 (Advanced Features)
- FIN-003: Expense Approval Workflow (depends on FIN-002, FIN-004, FIN-005)
- FIN-006: Settlement Calculation (depends on FIN-004, FIN-005)
- GRP-004: Group List View (depends on GRP-001, FIN-001, FIN-002)

### Level 4 (Supporting Features)
- NOTIF-001: Real-Time Notifications (depends on all features)
- RPT-001: Spending Reports (depends on FIN-001, FIN-002, FIN-003, GRP-003)
- RPT-002: Individual Balance Tracking (depends on FIN-001, FIN-002, FIN-005)
- UI-001: Mobile-Responsive Design (depends on all features)
- UI-002: Real-Time Updates (depends on all features)

---

## Implementation Priority Order

### Phase 1: Foundation (Must have for MVP)
1. AUTH-001: User Registration
2. AUTH-002: User Login
3. GRP-001: Group Creation
4. GRP-002: Member Invitation
5. GRP-003: Admin Role Management
6. FIN-001: Deposit Submission
7. FIN-002: Expense Submission
8. FIN-003: Expense Approval Workflow
9. FIN-004: Main Balance Calculation
10. FIN-005: Individual Balance Calculation
11. NOTIF-001: Real-Time Notifications
12. UI-001: Mobile-Responsive Design
13. UI-002: Real-Time Updates
14. GRP-004: Group List View

### Phase 2: Enhanced Features (High value)
15. FIN-006: Settlement Calculation
16. RPT-001: Spending Reports
17. RPT-002: Individual Balance Tracking

---

## Feature Status Tracking

### By Status
- **Draft:** 18 features
- **In Progress:** 0 features
- **Review:** 0 features
- **Completed:** 0 features
- **Blocked:** 0 features

### By Priority
- **P0 (Critical):** 14 features
- **P1 (High):** 3 features
- **P2 (Medium):** 0 features
- **P3 (Low):** 0 features

---

## Cross-Reference: Features by Dependency

### Features that depend on AUTH-002
- GRP-001, GRP-002, GRP-003, GRP-004
- FIN-001, FIN-002, FIN-003, FIN-004, FIN-005, FIN-006
- RPT-001, RPT-002
- UI-001, UI-002

### Features that depend on GRP-001
- GRP-002, GRP-003, GRP-004
- FIN-001, FIN-002, FIN-003, FIN-004, FIN-005, FIN-006
- RPT-001, RPT-002

### Features that depend on FIN-004
- FIN-005, FIN-006
- GRP-004
- UI-002

### Features that depend on NOTIF-001
- GRP-002, GRP-003, GRP-004
- FIN-001, FIN-003

---

## Notes for Development Teams

### Critical Path Features
The following features are on the critical path and must be completed in order:
1. AUTH-001 → AUTH-002 → GRP-001 → GRP-002 → GRP-003
2. GRP-001 → FIN-001 → FIN-004
3. GRP-001 → FIN-002 → FIN-003 → FIN-004 → FIN-005

### Parallel Development Opportunities
The following features can be developed in parallel:
- AUTH-001 and AUTH-002 (sequential but can start design work)
- FIN-001 and FIN-002 (after GRP-001 is complete)
- RPT-001 and RPT-002 (after financial features are complete)
- UI-001 and UI-002 (can start early with mockups)

### Testing Dependencies
- All financial features (FIN-*) require GRP-001 to be testable
- All notification features require the triggering feature to be complete
- UI features can be tested with mock data early in development

---

## Change Log

| Date | Feature ID | Change | Author |
|------|------------|--------|--------|
| 2026-01 | All | Initial feature breakdown from Product Specification | System |

---

## How to Use This Index

1. **Find a Feature:** Use the feature list tables above or search by Feature ID
2. **Check Dependencies:** Review the dependency graph to understand prerequisites
3. **Track Progress:** Update the Status column as features move through development
4. **Plan Sprints:** Use the Implementation Priority Order section
5. **Understand Relationships:** Use the Cross-Reference section to see feature connections

---

**Last Updated:** January 2026  
**Next Review:** As features progress through development lifecycle
