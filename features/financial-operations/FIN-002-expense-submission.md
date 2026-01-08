# FIN-002: Expense Submission

**Feature ID:** FIN-002  
**Category:** Financial Operations  
**Priority:** P0 (Critical)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Users can submit expenses for group approval.

---

## Functional Requirements
- Expense form with:
  - Amount (required, positive number, max 2 decimal places)
  - Date-Time (required, defaults to current)
  - Description (optional, max 200 characters)
  - Receipt attachment (optional, image file, max 5MB)
- Submission behavior:
  - Admin expenses: Auto-approved, immediately affect Main Balance
  - Member expenses: Enter "Pending" state, require Admin approval
- Pending expenses visible to all Admins
- Submitted expense visible to submitter with status indicator

---

## Business Rules
- Expense amount must be positive
- Date cannot be in the future
- Pending expenses do not affect Main Balance
- Only approved expenses are split among members
- Expense cannot exceed Main Balance (warning shown, but allowed)
- Receipt file types: JPG, PNG, PDF (max 5MB)

---

## Acceptance Criteria
- ✅ Admin expense immediately reflected in Main Balance
- ✅ Member expense enters pending state
- ✅ All Admins notified of pending expense
- ✅ Expense form validates all inputs
- ✅ Receipt upload works correctly
- ✅ Status indicator shows correct state
- ✅ Future date validation prevents submission

---

## Technical Notes
- File upload: Secure storage (AWS S3 or similar)
- Image processing: Thumbnail generation for receipts
- Status tracking: Pending → Approved/Rejected state machine
- Real-time: WebSocket notification to Admins for pending expenses

---

## Dependencies
- GRP-001: Group Creation
- FIN-003: Expense Approval Workflow
- FIN-004: Main Balance Calculation
- File storage service

---

## Related Features
- FIN-001: Deposit Submission
- FIN-003: Expense Approval Workflow
- FIN-004: Main Balance Calculation

---

## Test Cases
1. **TC-FIN-002-01:** Admin submits expense (auto-approved)
2. **TC-FIN-002-02:** Member submits expense (enters pending)
3. **TC-FIN-002-03:** Expense form validation works
4. **TC-FIN-002-04:** Receipt upload successful
5. **TC-FIN-002-05:** Future date prevented
6. **TC-FIN-002-06:** Negative amount prevented
7. **TC-FIN-002-07:** Admins notified of pending expense
8. **TC-FIN-002-08:** Status indicator accurate
