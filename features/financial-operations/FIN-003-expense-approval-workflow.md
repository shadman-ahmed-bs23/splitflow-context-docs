# FIN-003: Expense Approval Workflow

**Feature ID:** FIN-003  
**Category:** Financial Operations  
**Priority:** P0 (Critical)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Admins review and approve/reject pending member expenses.

---

## Functional Requirements
- Pending expenses list view for Admins
- Each pending expense displays:
  - Amount, date, description
  - Submitter name and timestamp
- Actions available:
  - Approve: Expense added to ledger, Main Balance updated
  - Reject: Expense removed, submitter notified with mandatory rejection reason
- Approval triggers:
  - Expense split equally among all members
  - Individual balances updated
  - Main Balance decreased
  - Real-time notifications to all members
- Rejection triggers:
  - Notification to submitter
  - Expense removed from pending list
  - No balance changes

---

## Business Rules
- Any Admin can approve/reject (first-come basis)
- Once approved/rejected, action cannot be undone
- Approved expenses are immutable
- Rejection reason is mandatory and must be provided by the Admin
- Expense split based on member count at time of approval

---

## Acceptance Criteria
- ✅ Approval updates balances within 1 second
- ✅ All members notified of approval
- ✅ Rejection notifies submitter with the provided reason
- ✅ Pending list updates in real-time
- ✅ Approved expenses cannot be modified
- ✅ Expense split calculated correctly
- ✅ Main Balance updated on approval

---

## Technical Notes
- Approval: Atomic transaction (expense + splits + balances)
- Split calculation: Equal division among all active members
- Real-time: WebSocket broadcast on approval/rejection
- Immutability: Approved expenses are read-only

---

## Dependencies
- FIN-002: Expense Submission
- FIN-004: Main Balance Calculation
- FIN-005: Individual Balance Calculation
- NOTIF-001: Real-Time Notifications

---

## Related Features
- FIN-002: Expense Submission
- FIN-004: Main Balance Calculation
- FIN-005: Individual Balance Calculation
- NOTIF-001: Real-Time Notifications

---

## Test Cases
1. **TC-FIN-003-01:** Admin approves pending expense
2. **TC-FIN-003-02:** Admin rejects pending expense
3. **TC-FIN-003-03:** Balances update correctly on approval
4. **TC-FIN-003-04:** Expense split calculated correctly
5. **TC-FIN-003-05:** All members notified of approval
6. **TC-FIN-003-06:** Submitter notified of rejection
7. **TC-FIN-003-07:** Approved expense cannot be modified
8. **TC-FIN-003-08:** Pending list updates in real-time
