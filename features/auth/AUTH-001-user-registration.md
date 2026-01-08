# AUTH-001: User Registration

**Feature ID:** AUTH-001  
**Category:** Authentication & User Management  
**Priority:** P0 (Critical)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Users must be able to create an account to use SplitFlow using email and a one-time password (OTP), with no persistent password.

---

## Functional Requirements
- Registration form with email only
- On submit, system generates a one-time verification code (OTP) and sends it to the provided email
- OTP length configurable (default: 6 digits)
- OTP entry screen with:
  - Email (pre-filled, read-only)
  - OTP input
- OTP verification required before account activation
- Duplicate email prevention
- Terms of service and privacy policy acceptance

---

## Business Rules
- Email must be unique across all users
- Account remains inactive until OTP verification
- OTP expires after 10 minutes
- Maximum OTP verification attempts per session (e.g., 5)
- User must accept terms of service to proceed

---

## Acceptance Criteria
- ✅ User can register with valid email
- ✅ OTP email sent within 30 seconds
- ✅ Account activated only after correct OTP verification
- ✅ Invalid inputs show clear error messages
- ✅ Duplicate email shows appropriate error
- ✅ Expired OTP is rejected with appropriate error
- ✅ Exceeding OTP attempts locks verification for a cooldown period

---

## Technical Notes
- Email validation: RFC 5322 compliant
- OTP generation: Cryptographically secure random numeric code
- OTP storage: Short-lived server-side store keyed by email
- Email service: SendGrid or AWS SES

---

## Dependencies
- Email service integration
- Database user table
- Authentication service

---

## Related Features
- AUTH-002: User Login
- AUTH-003: Password Reset (Future)

---

## Test Cases
1. **TC-AUTH-001-01:** Valid registration with email verification
2. **TC-AUTH-001-02:** Registration with duplicate email (should fail)
3. **TC-AUTH-001-03:** Registration with weak password (should fail)
4. **TC-AUTH-001-04:** Registration without accepting terms (should fail)
5. **TC-AUTH-001-05:** Verification email delivery within SLA
6. **TC-AUTH-001-06:** Expired verification link handling
