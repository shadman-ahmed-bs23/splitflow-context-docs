# RPT-002: Individual Balance Tracking

**Feature ID:** RPT-002  
**Category:** Reporting & Analytics  
**Priority:** P1 (High)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Users can view their personal transaction history and balance trends.

---

## Functional Requirements
- Personal dashboard showing:
  - Current individual balance
  - Transaction history (deposits, expenses)
  - Balance over time graph
  - Contribution vs. spending breakdown
- Filterable by date range
- Export personal statement (future)

---

## Business Rules
- Users can only view their own data
- Transaction history includes all relevant transactions
- Balance trend calculated from historical data
- Breakdown shows deposits vs. expense shares

---

## Acceptance Criteria
- ✅ Personal dashboard loads within 2 seconds
- ✅ Transaction history accurate
- ✅ Graphs render correctly
- ✅ Filters work as expected
- ✅ Balance trend calculated correctly
- ✅ Breakdown shows accurate data

---

## Technical Notes
- Data fetching: Optimized queries for user's transactions
- Graph generation: Client-side charting library
- Performance: Pagination for large transaction histories
- Caching: Client-side cache for frequently accessed data

---

## Dependencies
- FIN-001: Deposit Submission
- FIN-002: Expense Submission
- FIN-005: Individual Balance Calculation
- Charting library

---

## Related Features
- FIN-005: Individual Balance Calculation
- RPT-001: Spending Reports

---

## Test Cases
1. **TC-RPT-002-01:** Personal dashboard loads correctly
2. **TC-RPT-002-02:** Transaction history accurate
3. **TC-RPT-002-03:** Balance over time graph renders
4. **TC-RPT-002-04:** Contribution vs. spending breakdown accurate
5. **TC-RPT-002-05:** Date range filter works
6. **TC-RPT-002-06:** Performance acceptable with many transactions
