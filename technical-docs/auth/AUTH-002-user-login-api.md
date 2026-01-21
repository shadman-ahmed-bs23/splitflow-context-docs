# AUTH-002: User Login (Email + OTP) - Technical Specification

**Feature ID:** AUTH-002  
**Category:** Authentication & User Management  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Request Login OTP
**Endpoint:** `POST /api/v1/auth/login/request-otp`

**Request Body:**
```json
{
  "email": "user@example.com"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "OTP sent to your email",
    "expires_in": 600,  // OTP expiration time in seconds (10 minutes)
    "email": "user@example.com"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Error Responses:**
- `422 Unprocessable Entity` - Invalid email format
- `404 Not Found` - Email not registered
- `403 Forbidden` - Email not verified
- `429 Too Many Requests` - Rate limit exceeded

---

### 1.2 Verify OTP and Login
**Endpoint:** `POST /api/v1/auth/login/verify-otp`

**Request Body:**
```json
{
  "email": "user@example.com",
  "otp": "123456"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "email_verified_at": "2026-01-15T10:30:00Z"
    },
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "token_type": "Bearer",
    "expires_in": 604800000  // Token expiration time in milliseconds (7 days)
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Error Responses:**
- `422 Unprocessable Entity` - Invalid OTP
- `429 Too Many Requests` - Too many attempts
- `404 Not Found` - OTP request not found

---

### 1.3 Logout
**Endpoint:** `POST /api/v1/auth/logout`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Successfully logged out"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

---

## 2. Database Schema

### 2.1 Uses Existing Tables
- `users` (from AUTH-001)
- `otp_requests` (from AUTH-001, purpose='login')

### 2.2 OAuth Access Tokens (Laravel Passport)
```sql
-- Managed by Laravel Passport
-- Table: oauth_access_tokens
-- Table: oauth_clients
-- Table: oauth_personal_access_clients
```

---

## 3. Business Logic Implementation

### 3.1 Service: LoginService
**Location:** `app/Services/Auth/LoginService.php`

**Methods:**
- `requestOtp(string $email): array`
- `verifyOtp(string $email, string $otp): array`
- `logout(User $user): void`

**Key Logic:**
1. Validate email exists and is verified
2. Generate login OTP (6 digits, 10 min expiry)
3. Store OTP with attempt tracking
4. Send OTP via email
5. On verification: Update last_login_at, issue Passport token
6. Rate limit: 5 requests per email per 15 minutes

---

### 3.2 Action: RequestLoginOtpAction
**Location:** `app/Actions/Auth/RequestLoginOtpAction.php`

**Responsibilities:**
- Validate email format
- Check user exists and email verified
- Generate and store OTP
- Queue email job
- Return response

---

### 3.3 Action: VerifyLoginOtpAction
**Location:** `app/Actions/Auth/VerifyLoginOtpAction.php`

**Responsibilities:**
- Validate OTP
- Check expiration and attempts
- Update user last_login_at
- Create Passport token
- Return user and token

---

## 4. Technical Implementation Details

### 4.1 Token Management (Laravel Passport)
**Configuration:** `config/auth.php`

```php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

**Token Expiration:** 7 days (configurable in Passport)

### 4.2 Rate Limiting
**Middleware:** `app/Http/Middleware/ThrottleOtpRequests.php`

**Rules:**
- 5 OTP requests per email per 15 minutes
- 5 verification attempts per OTP
- 20 total requests per IP per hour

### 4.3 Session Management
- Multiple concurrent sessions allowed
- Tokens stored in `oauth_access_tokens`
- Revocation on logout
- No server-side session storage

---

## 5. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `LOGIN001` | 422 | Invalid email format |
| `LOGIN002` | 404 | Email not registered |
| `LOGIN003` | 403 | Email not verified |
| `LOGIN004` | 422 | Invalid or expired OTP |
| `LOGIN005` | 429 | Too many OTP requests |
| `LOGIN006` | 429 | Maximum verification attempts exceeded |
| `LOGIN007` | 401 | Invalid or expired token |

### 5.2 Error Response Format
```json
{
  "error": {
    "message": "Authentication failed",
    "code": "LOGIN002",
    "fields": [
      {
        "field": "email",
        "message": "Email not registered"
      }
    ]
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

---

## 6. Security Considerations

1. **OTP Security:** Same as registration (encrypted, short-lived)
2. **Account Lockout:** 5 failed attempts → 15-minute cooldown
3. **Token Security:** Passport tokens, HTTP-only cookies preferred
4. **Rate Limiting:** Aggressive limits on OTP endpoints
5. **Email Verification:** Only verified accounts can login
6. **Token Revocation:** Immediate on logout

---

## 7. Testing Strategy

### 7.1 Unit Tests
- OTP generation
- Email verification check
- Token creation
- Rate limiting logic

### 7.2 Feature Tests
- Successful login flow
- Unverified email rejection
- Expired OTP handling
- Account lockout
- Multi-device sessions

### 7.3 Integration Tests
- Passport token issuance
- Email service integration
- Token revocation

---

## 8. Dependencies

- Laravel Passport (token management)
- Email service (SendGrid/AWS SES)
- Redis (rate limiting, queue)
- Database (users, otp_requests, oauth_* tables)

---

## 9. Related Documentation

- [AUTH-001 Technical Spec](./AUTH-001-user-registration-api.md) - Registration flow
- Laravel Passport Documentation
- API Authentication Guide
