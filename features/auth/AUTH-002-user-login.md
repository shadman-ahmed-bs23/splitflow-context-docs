# AUTH-002: User Login

**Feature ID:** AUTH-002  
**Category:** Authentication & User Management  
**Priority:** P0 (Critical)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Registered users can securely log into their accounts.

---

## Functional Requirements
- Email/password authentication
- "Remember me" option (30-day session)
- Password reset via email
- Account lockout after 5 failed attempts (15-minute cooldown)
- Multi-device session support

---

## Business Rules
- Only verified accounts can log in
- Failed login attempts tracked per IP and email
- Session tokens expire after 7 days (or 30 days with "Remember me")
- Password reset link expires after 1 hour
- Concurrent sessions allowed (no limit)

---

## Acceptance Criteria
- ✅ Successful login redirects to group list
- ✅ Failed login shows appropriate error message
- ✅ Password reset email delivered within 60 seconds
- ✅ Session persists across browser tabs
- ✅ Account lockout activates after 5 failed attempts
- ✅ "Remember me" extends session to 30 days
- ✅ Password reset flow works end-to-end

---

## Technical Notes
- Authentication: JWT tokens
- Session storage: HTTP-only cookies or localStorage
- Rate limiting: 5 attempts per 15 minutes per IP
- Password reset: Secure token generation
- Multi-factor authentication: Future enhancement

---

## Dependencies
- AUTH-001: User Registration
- Email service integration
- Session management service

---

## Related Features
- AUTH-001: User Registration
- AUTH-003: Password Reset
- AUTH-004: Session Management (Future)

---

## Test Cases
1. **TC-AUTH-002-01:** Successful login with valid credentials
2. **TC-AUTH-002-02:** Failed login with invalid password
3. **TC-AUTH-002-03:** Account lockout after 5 failed attempts
4. **TC-AUTH-002-04:** "Remember me" functionality
5. **TC-AUTH-002-05:** Password reset flow
6. **TC-AUTH-002-06:** Login with unverified account (should fail)
7. **TC-AUTH-002-07:** Multi-device session support
