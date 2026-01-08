# FIN-004: Main Balance Calculation

**Feature ID:** FIN-004  
**Category:** Financial Operations  
**Priority:** P0 (Critical)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Real-time calculation and display of group's Main Balance.

---

## Functional Requirements
- Main Balance = Sum of all Admin deposits - Sum of all approved expenses
- Displayed prominently in group detail view
- Updates in real-time across all user sessions
- Formatted with currency symbol and 2 decimal places
- Color coding:
  - Positive: Green
  - Zero: Gray
  - Negative: Red (with warning indicator)

---

## Business Rules
- Only Admin deposits increase Main Balance
- Only approved expenses decrease Main Balance
- Pending expenses do not affect Main Balance
- Balance can be negative (indicates group owes money)
- Calculation is always current (not cached)

---

## Acceptance Criteria
- ✅ Balance calculates correctly for all scenarios
- ✅ Updates propagate within 1 second
- ✅ Display formatting correct for all currencies
- ✅ Negative balance clearly indicated
- ✅ Color coding works correctly
- ✅ Real-time updates across all sessions

---

## Technical Notes
- Calculation: Database aggregation query (optimized with indexes)
- Real-time: WebSocket push on any balance-affecting transaction
- Currency formatting: Locale-aware number formatting
- Performance: Cached calculation with invalidation on transactions

---

## Dependencies
- FIN-001: Deposit Submission
- FIN-002: Expense Submission
- FIN-003: Expense Approval Workflow
- UI-002: Real-Time Updates

---

## Related Features
- FIN-001: Deposit Submission
- FIN-002: Expense Submission
- FIN-003: Expense Approval Workflow
- UI-002: Real-Time Updates

---

## Test Cases
1. **TC-FIN-004-01:** Main Balance calculates correctly
2. **TC-FIN-004-02:** Deposit increases balance
3. **TC-FIN-004-03:** Approved expense decreases balance
4. **TC-FIN-004-04:** Pending expense does not affect balance
5. **TC-FIN-004-05:** Negative balance displayed correctly
6. **TC-FIN-004-06:** Real-time updates work
7. **TC-FIN-004-07:** Currency formatting correct
8. **TC-FIN-004-08:** Color coding accurate
