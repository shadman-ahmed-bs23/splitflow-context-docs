# FIN-001: Deposit Submission - Technical Specification

**Feature ID:** FIN-001  
**Category:** Financial Operations  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Submit Deposit
**Endpoint:** `POST /api/v1/groups/{groupId}/deposits`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Request Body:**
```json
{
  "amount": 1500.00,
  "date_time": "2026-01-15T10:30:00Z",
  "description": "Initial group fund"
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "group_id": "uuid",
    "type": "deposit",
    "amount": 1500.00,
    "date_time": "2026-01-15T10:30:00Z",
    "description": "Initial group fund",
    "submitted_by": {
      "id": "uuid",
      "email": "admin@example.com"
    },
    "status": "approved",
    "main_balance": 1500.00,
    "created_at": "2026-01-15T10:30:00Z"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Error Responses:**
- `403 Forbidden` - Not an admin
- `404 Not Found` - Group not found
- `422 Unprocessable Entity` - Validation errors
- `400 Bad Request` - System limit exceeded

---

## 2. Database Schema

### 2.1 Transactions Table
```sql
CREATE TABLE transactions (
    id CHAR(36) PRIMARY KEY,
    group_id CHAR(36) NOT NULL,
    type ENUM('deposit', 'expense') NOT NULL,
    amount DECIMAL(15, 2) NOT NULL,
    date_time TIMESTAMP NOT NULL,
    description VARCHAR(200) NULL,
    submitted_by CHAR(36) NOT NULL,
    status ENUM('approved', 'pending', 'rejected') DEFAULT 'approved' NOT NULL,
    approved_by CHAR(36) NULL,
    approved_at TIMESTAMP NULL,
    rejection_reason TEXT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_group_id (group_id),
    INDEX idx_type (type),
    INDEX idx_status (status),
    INDEX idx_submitted_by (submitted_by),
    INDEX idx_date_time (date_time),
    FOREIGN KEY (group_id) REFERENCES groups(id) ON DELETE RESTRICT,
    FOREIGN KEY (submitted_by) REFERENCES users(id) ON DELETE RESTRICT,
    FOREIGN KEY (approved_by) REFERENCES users(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 2.2 Groups Table Update
```sql
-- Add trigger or use computed column for main_balance
-- Or update via application logic after each transaction
ALTER TABLE groups 
ADD COLUMN main_balance DECIMAL(15, 2) DEFAULT 0.00 NOT NULL;
```

---

## 3. Business Logic Implementation

### 3.1 Service: DepositService
**Location:** `app/Services/Financial/DepositService.php`

**Methods:**
- `submitDeposit(Group $group, User $user, array $data): Transaction`

**Key Logic:**
1. Verify user is admin of group
2. Validate amount (positive, max 2 decimals)
3. Validate date (not future)
4. Check system limit (999,999.99)
5. Create transaction with status='approved'
6. Update group.main_balance atomically
7. Update user's individual_balance
8. Dispatch DepositCreated event
9. Return transaction resource

---

### 3.2 Action: SubmitDepositAction
**Location:** `app/Actions/Financial/SubmitDepositAction.php`

**Responsibilities:**
- Validate admin authorization
- Validate deposit data
- Check system limits
- Create transaction in database transaction
- Update balances atomically
- Dispatch events
- Return transaction resource

---

### 3.3 Balance Update Logic
```php
DB::transaction(function () use ($group, $amount) {
    $group->lockForUpdate();
    $newBalance = $group->main_balance + $amount;
    
    if ($newBalance > 999999.99) {
        throw new SystemLimitExceededException();
    }
    
    $group->main_balance = $newBalance;
    $group->last_activity_at = now();
    $group->save();
});
```

---

## 4. Technical Implementation Details

### 4.1 Decimal Precision
- Amount stored as `DECIMAL(15, 2)`
- Validation: Max 2 decimal places
- Calculations use `bcmath` or database functions

### 4.2 Atomic Operations
- Database transactions ensure consistency
- Row-level locking on group table
- Balance updates are atomic

### 4.3 Real-time Updates
- WebSocket broadcast on deposit creation
- All group members receive update
- Main balance updated in real-time

---

## 5. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `DEP001` | 403 | Only admins can submit deposits |
| `DEP002` | 422 | Amount must be positive |
| `DEP003` | 422 | Date cannot be in the future |
| `DEP004` | 400 | System limit exceeded (999,999.99) |
| `DEP005` | 422 | Invalid amount format |

### 5.2 Error Response Format
```json
{
  "error": {
    "message": "Validation failed",
    "code": "DEP002",
    "fields": [
      {
        "field": "amount",
        "message": "Amount must be positive"
      },
      {
        "field": "date_time",
        "message": "Date cannot be in the future"
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

1. **Authorization:** Admin-only via Policy
2. **Validation:** Server-side amount validation
3. **Precision:** Use DECIMAL to avoid floating-point errors
4. **Atomicity:** Database transactions for balance updates
5. **Audit Trail:** All deposits logged in transactions table

---

## 7. Testing Strategy

### 7.1 Unit Tests
- Amount validation
- Date validation
- Balance calculation
- System limit check

### 7.2 Feature Tests
- Successful deposit submission
- Admin authorization check
- Balance update accuracy
- System limit enforcement
- Real-time notifications

### 7.3 Integration Tests
- Database transactions
- Balance consistency
- Event dispatching

---

## 8. Events & Listeners

### 8.1 DepositCreated Event
**Location:** `app/Events/DepositCreated.php`

**Listeners:**
- Update unread counts
- Broadcast real-time update
- Send notifications

---

## 9. Dependencies

- GRP-001: Group Creation
- GRP-003: Admin Role Management
- Database (transactions, groups tables)
- NOTIF-001: Real-Time Notifications

---

## 10. Related Documentation

- [FIN-002 Technical Spec](./FIN-002-technical.md) - Expense Submission
- [FIN-004 Technical Spec](./FIN-004-technical.md) - Main Balance Calculation
- [FIN-005 Technical Spec](./FIN-005-technical.md) - Individual Balance Calculation
