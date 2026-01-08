# FIN-005: Individual Balance Calculation

**Feature ID:** FIN-005  
**Category:** Financial Operations  
**Priority:** P0 (Critical)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Each user sees their personal financial standing within the group.

---

## Functional Requirements
- Individual Balance = (User's Deposits) - (User's Share of Approved Expenses)
- Displayed in user profile section of group view
- Breakdown available showing:
  - Total deposits made
  - Total expenses approved (user's share)
  - Net balance
- Updated in real-time when expenses approved

---

## Business Rules
- Expenses split equally among all members at time of approval
- If member count changes, historical splits remain unchanged
- Only approved expenses count toward individual balance
- Deposits only count if user is Admin
- Balance can be positive (user is owed) or negative (user owes)

---

## Acceptance Criteria
- ✅ Balance calculates correctly for all members
- ✅ Breakdown shows accurate components
- ✅ Updates in real-time upon expense approval
- ✅ Historical accuracy maintained
- ✅ Split calculation correct for member count
- ✅ Deposits only counted for Admins

---

## Technical Notes
- Calculation: Aggregation of deposits and expense splits
- Split storage: ExpenseSplit table tracks each member's share
- Real-time: WebSocket update on expense approval
- Performance: Indexed queries for fast calculation

---

## Dependencies
- FIN-001: Deposit Submission
- FIN-003: Expense Approval Workflow
- FIN-004: Main Balance Calculation
- UI-002: Real-Time Updates

---

## Related Features
- FIN-001: Deposit Submission
- FIN-003: Expense Approval Workflow
- FIN-006: Settlement Calculation

---

## Test Cases
1. **TC-FIN-005-01:** Individual balance calculates correctly
2. **TC-FIN-005-02:** Deposit increases individual balance (Admin only)
3. **TC-FIN-005-03:** Expense approval decreases individual balance
4. **TC-FIN-005-04:** Split calculation correct for member count
5. **TC-FIN-005-05:** Breakdown shows accurate components
6. **TC-FIN-005-06:** Real-time updates work
7. **TC-FIN-005-07:** Historical splits remain unchanged
8. **TC-FIN-005-08:** Non-admin deposits not counted
