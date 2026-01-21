# AUTH-002: User Login (Email + OTP)

**Feature ID:** AUTH-002  
**Category:** Authentication & User Management  
**Priority:** P0 (Critical)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Registered users can securely log into their accounts using email and a one-time password (OTP), with no persistent password.

---

## Functional Requirements
- Email + OTP authentication
- Login form with email field
- On submit, system:
  - Generates a one-time login OTP
  - Sends OTP to the provided email
- OTP entry screen with:
  - Email (pre-filled, read-only)
  - OTP input
- OTP verification required to start a session
- Account lockout after 5 failed OTP attempts (15-minute cooldown)
- Multi-device session support

---

## Business Rules
- Only verified accounts can log in
- Failed login attempts tracked per IP and email
- Session tokens expire after 7 days
- OTP expires after 5 minutes
- Concurrent sessions allowed (no limit)

---

## Acceptance Criteria
- ✅ Successful login redirects to group list
- ✅ Failed login shows appropriate error message
- ✅ Session persists across browser tabs
- ✅ Account lockout activates after 5 failed attempts
- ✅ OTP expiry handled correctly
- ✅ Multi-device sessions work as expected

---

## Technical Notes
- Authentication: JWT tokens
- Session storage: HTTP-only cookies or localStorage
- Rate limiting: 5 attempts per 15 minutes per IP
- OTP generation: Cryptographically secure random numeric code
- No password storage for users (passwordless model)

---

## Dependencies
- AUTH-001: User Registration
- Email service integration
- Session management service

---

## Related Features
- AUTH-001: User Registration
- AUTH-004: Session Management (Future)

---

## Test Cases
1. **TC-AUTH-002-01:** Successful login with valid email + OTP
2. **TC-AUTH-002-02:** Failed login with invalid OTP
3. **TC-AUTH-002-03:** Account lockout after 5 failed OTP attempts
4. **TC-AUTH-002-04:** OTP expiry handling
5. **TC-AUTH-002-05:** Login with unverified account (should fail)
6. **TC-AUTH-002-06:** Multi-device session support
