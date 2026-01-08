# GRP-002: Member Invitation

**Feature ID:** GRP-002  
**Category:** Group Management  
**Priority:** P0 (Critical)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Admins can invite members to join a group via email.

---

## Functional Requirements
- Invitation form with email input
- Invitation email sent with unique invitation link
- Invitation link expires after 7 days
- Invited user can accept/decline invitation
- Member added to group upon acceptance
- Notification sent to all group members when new member joins

---

## Business Rules
- Only Admins can send invitations
- Email must be registered SplitFlow account
- User cannot be invited if already a member
- Maximum 50 members per group
- Invitation link is single-use only
- Invitation expires after 7 days

---

## Acceptance Criteria
- ✅ Invitation email delivered within 60 seconds
- ✅ Invitation link works only once
- ✅ Member appears in group member list after acceptance
- ✅ All group members notified of new member
- ✅ Invitation cannot be sent to existing members
- ✅ Expired invitations cannot be accepted
- ✅ Group member limit enforced (50 max)

---

## Technical Notes
- Invitation token: Cryptographically secure, single-use
- Email template: Branded invitation email
- Database: Invitations table with expiration tracking
- Real-time: WebSocket notification on member join

---

## Dependencies
- GRP-001: Group Creation
- Email service integration
- NOTIF-001: Real-Time Notifications

---

## Related Features
- GRP-001: Group Creation
- GRP-003: Admin Role Management
- NOTIF-001: Real-Time Notifications

---

## Test Cases
1. **TC-GRP-002-01:** Admin sends invitation to valid user
2. **TC-GRP-002-02:** Invitation email delivered successfully
3. **TC-GRP-002-03:** User accepts invitation and joins group
4. **TC-GRP-002-04:** Invitation link works only once
5. **TC-GRP-002-05:** Expired invitation cannot be accepted
6. **TC-GRP-002-06:** Cannot invite existing member
7. **TC-GRP-002-07:** Member limit enforced at 50
8. **TC-GRP-002-08:** Non-admin cannot send invitations
