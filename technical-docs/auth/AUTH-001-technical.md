# AUTH-001: User Registration - Technical Specification

**Feature ID:** AUTH-001  
**Category:** Authentication & User Management  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Request OTP for Registration
**Endpoint:** `POST /api/v1/auth/register/request-otp`

**Request Body:**
```json
{
  "email": "user@example.com",
  "accept_terms": true
}
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "OTP sent to your email",
    "expires_in": 600,
    "email": "user@example.com"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Error Responses:**
- `422 Unprocessable Entity` - Validation errors
- `409 Conflict` - Email already registered
- `429 Too Many Requests` - Rate limit exceeded

---

### 1.2 Verify OTP and Complete Registration
**Endpoint:** `POST /api/v1/auth/register/verify-otp`

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
    "expires_in": 604800
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Error Responses:**
- `422 Unprocessable Entity` - Invalid OTP or expired
- `429 Too Many Requests` - Too many verification attempts
- `404 Not Found` - OTP request not found

---

## 2. Database Schema

### 2.1 Users Table
```sql
CREATE TABLE users (
    id CHAR(36) PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    email_verified_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    last_login_at TIMESTAMP NULL,
    INDEX idx_email (email),
    INDEX idx_email_verified (email_verified_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 2.2 OTP Requests Table
```sql
CREATE TABLE otp_requests (
    id CHAR(36) PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    otp_code VARCHAR(6) NOT NULL,
    purpose ENUM('registration', 'login') NOT NULL,
    attempts INT DEFAULT 0,
    max_attempts INT DEFAULT 5,
    expires_at TIMESTAMP NOT NULL,
    verified_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_email_purpose (email, purpose),
    INDEX idx_expires_at (expires_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## 3. Business Logic Implementation

### 3.1 Service: RegistrationService
**Location:** `app/Services/Auth/RegistrationService.php`

**Methods:**
- `requestOtp(string $email, bool $acceptTerms): array`
- `verifyOtp(string $email, string $otp): array`

**Key Logic:**
1. Validate email format (RFC 5322)
2. Check email uniqueness
3. Generate cryptographically secure 6-digit OTP
4. Store OTP with expiration (10 minutes)
5. Send OTP via email service
6. Rate limit: 3 requests per email per 15 minutes
7. On verification: Create user, mark email as verified, issue access token

---

### 3.2 Action: RequestRegistrationOtpAction
**Location:** `app/Actions/Auth/RequestRegistrationOtpAction.php`

**Responsibilities:**
- Validate request data
- Check duplicate email
- Generate and store OTP
- Queue email job
- Return response

---

### 3.3 Action: VerifyRegistrationOtpAction
**Location:** `app/Actions/Auth/VerifyRegistrationOtpAction.php`

**Responsibilities:**
- Validate OTP format
- Check OTP existence and expiration
- Verify attempt limits
- Create user account
- Issue access token via Passport
- Return user data and token

---

## 4. Technical Implementation Details

### 4.1 OTP Generation
```php
use Illuminate\Support\Str;

function generateOtp(): string
{
    return str_pad((string) random_int(100000, 999999), 6, '0', STR_PAD_LEFT);
}
```

### 4.2 Rate Limiting
**Middleware:** `app/Http/Middleware/ThrottleOtpRequests.php`

**Rules:**
- 3 OTP requests per email per 15 minutes
- 5 verification attempts per OTP request
- 10 total requests per IP per hour

### 4.3 Email Job
**Location:** `app/Jobs/SendRegistrationOtpJob.php`

**Queue:** `emails` (Redis)
**Retries:** 3
**Timeout:** 30 seconds

**Template Variables:**
- `{{ otp }}` - 6-digit code
- `{{ expires_in }}` - Minutes until expiration

---

## 5. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `REG001` | 422 | Invalid email format |
| `REG002` | 409 | Email already registered |
| `REG003` | 422 | Terms not accepted |
| `REG004` | 422 | Invalid or expired OTP |
| `REG005` | 429 | Too many OTP requests |
| `REG006` | 429 | Maximum verification attempts exceeded |

### 5.2 Error Response Format
```json
{
  "errors": [
    {
      "code": "REG001",
      "message": "Invalid email format",
      "field": "email"
    }
  ],
  "meta": {
    "request_id": "uuid"
  }
}
```

---

## 6. Security Considerations

1. **OTP Storage:** Encrypted in database, never logged
2. **Rate Limiting:** Per-email and per-IP limits
3. **OTP Expiration:** 10 minutes hard limit
4. **Attempt Limits:** Lock after 5 failed attempts
5. **Email Validation:** Server-side RFC 5322 validation
6. **Token Security:** Passport tokens, HTTP-only cookies preferred

---

## 7. Testing Strategy

### 7.1 Unit Tests
- OTP generation uniqueness
- Email validation logic
- Rate limiting logic
- Expiration checking

### 7.2 Feature Tests
- Successful registration flow
- Duplicate email rejection
- Expired OTP handling
- Rate limit enforcement
- Invalid OTP rejection

### 7.3 Integration Tests
- Email service integration
- Database transactions
- Token issuance

---

## 8. Dependencies

- Laravel Passport (token issuance)
- Email service (SendGrid/AWS SES)
- Redis (rate limiting, queue)
- Database (users, otp_requests tables)

---

## 9. Related Documentation

- [AUTH-002 Technical Spec](./AUTH-002-technical.md) - Login flow
- Laravel Passport Documentation
- Email Service Integration Guide
