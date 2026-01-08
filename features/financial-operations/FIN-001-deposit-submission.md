# FIN-001: Deposit Submission

**Feature ID:** FIN-001  
**Category:** Financial Operations  
**Priority:** P0 (Critical)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Admins can add funds to the group's Main Balance.

---

## Functional Requirements
- Deposit form with:
  - Amount (required, positive number, max 2 decimal places)
  - Date-Time (defaults to current, can be adjusted)
  - Optional description (max 200 characters)
  - Admin identifier (auto-filled, non-editable)
- Deposit immediately added to Main Balance
- Individual balance updated for depositing Admin
- Real-time notification to all group members
- Deposit appears in transaction history

---

## Business Rules
- Only Admins can submit deposits
- Deposit amount must be positive
- Date cannot be in the future
- Main Balance cannot exceed system limit ($999,999.99)
- Deposits are immediately effective (no approval needed)

---

## Acceptance Criteria
- ✅ Deposit reflected in Main Balance instantly
- ✅ All group members see updated balance within 1 second
- ✅ Deposit appears in transaction history
- ✅ Individual balance updated correctly
- ✅ Non-admins cannot submit deposits
- ✅ Validation prevents invalid amounts
- ✅ System limit enforced

---

## Technical Notes
- Amount: Decimal precision (2 decimal places)
- Transaction: Atomic database operation
- Real-time: WebSocket broadcast to all group members
- Currency: Respects group currency setting

---

## Dependencies
- GRP-001: Group Creation
- GRP-003: Admin Role Management
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
1. **TC-FIN-001-01:** Admin submits valid deposit
2. **TC-FIN-001-02:** Main Balance updates immediately
3. **TC-FIN-001-03:** All members receive notification
4. **TC-FIN-001-04:** Deposit appears in transaction history
5. **TC-FIN-001-05:** Individual balance updated for Admin
6. **TC-FIN-001-06:** Non-admin cannot submit deposit
7. **TC-FIN-001-07:** Negative amount validation (should fail)
8. **TC-FIN-001-08:** System limit enforcement
