# GRP-003: Admin Role Management

**Feature ID:** GRP-003  
**Category:** Group Management  
**Priority:** P0 (Critical)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Admins can promote members to Admin or revoke Admin status.

---

## Functional Requirements
- Admin dashboard showing all members with role indicators
- "Promote to Admin" action for members
- "Revoke Admin" action for other Admins
- Confirmation dialog before role changes
- Notification sent to affected user

---

## Business Rules
- At least one Admin must always exist in a group
- Admin cannot revoke their own Admin status if they are the only Admin
- Admin can resign if at least one other Admin exists
- Role changes logged in audit trail
- Immediate effect upon role change

---

## Acceptance Criteria
- ✅ Role change takes effect immediately
- ✅ User receives notification of role change
- ✅ Group always has at least one Admin
- ✅ Audit log records all role changes
- ✅ Cannot revoke last Admin's status
- ✅ Confirmation dialog prevents accidental changes

---

## Technical Notes
- Role change: Atomic database transaction
- Audit logging: All role changes recorded with timestamp and actor
- Real-time: WebSocket notification to affected user
- Authorization: Verify Admin status before allowing action

---

## Dependencies
- GRP-001: Group Creation
- GRP-002: Member Invitation
- NOTIF-001: Real-Time Notifications
- Audit logging system

---

## Related Features
- GRP-001: Group Creation
- GRP-002: Member Invitation
- NOTIF-001: Real-Time Notifications

---

## Test Cases
1. **TC-GRP-003-01:** Admin promotes member to Admin
2. **TC-GRP-003-02:** Admin revokes another Admin's status
3. **TC-GRP-003-03:** Cannot revoke last Admin's status
4. **TC-GRP-003-04:** Role change notification sent
5. **TC-GRP-003-05:** Audit log records role change
6. **TC-GRP-003-06:** Non-admin cannot change roles
7. **TC-GRP-003-07:** Admin can resign when other Admins exist
