# GRP-001: Group Creation

**Feature ID:** GRP-001  
**Category:** Group Management  
**Priority:** P0 (Critical)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Any authenticated user can create a new group and automatically becomes the primary Admin.

---

## Functional Requirements
- Group creation form with:
  - Group name (required, max 50 characters)
  - Optional description (max 200 characters)
  - Default currency selection (pre-filled with ৳ and non-editable in MVP)
- Creator automatically assigned Admin role
- Initial Main Balance set to $0.00
- Group assigned unique identifier

---

## Business Rules
- User can create unlimited groups
- Group name must be unique per user (can reuse names across different groups)
- Groups can never be deleted; they can only be archived
- Archived groups become read-only and hidden from default active list views
- All groups are always private and only visible to members
- Currency is fixed to Bangladeshi Taka (৳) for this phase

---

## Acceptance Criteria
- ✅ Group created successfully with creator as Admin
- ✅ Group appears in creator's group list immediately
- ✅ Group settings can be edited by Admin
- ✅ Group ID is unique and non-guessable
- ✅ Initial Main Balance is $0.00
- ✅ Validation prevents invalid group names

---

## Technical Notes
- Group ID: UUID v4
- Currency: ISO 4217 codes
- Database: Transaction-safe group creation
- Real-time: WebSocket notification to creator

---

## Dependencies
- AUTH-002: User Login (user must be authenticated)
- Database groups table
- Group member relationship table

---

## Related Features
- GRP-002: Member Invitation
- GRP-003: Admin Role Management
- GRP-004: Group List View

---

## Test Cases
1. **TC-GRP-001-01:** Create group with valid data
2. **TC-GRP-001-02:** Create group with duplicate name (same user) - should fail
3. **TC-GRP-001-03:** Create group with name exceeding 50 characters - should fail
4. **TC-GRP-001-04:** Creator automatically assigned Admin role
5. **TC-GRP-001-05:** Initial balance set to $0.00
6. **TC-GRP-001-06:** Group appears in list immediately after creation
