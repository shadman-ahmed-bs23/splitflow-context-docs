# FIN-003: Expense Approval Workflow - Technical Specification

**Feature ID:** FIN-003  
**Category:** Financial Operations  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Get Pending Expenses
**Endpoint:** `GET /api/v1/groups/{groupId}/expenses/pending`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "uuid",
      "amount": 500.00,
      "date_time": "2026-01-15T10:30:00Z",
      "description": "Lunch at restaurant",
      "submitted_by": {
        "id": "uuid",
        "email": "member@example.com"
      },
      "created_at": "2026-01-15T10:30:00Z"
    }
  ],
  "meta": {
    "request_id": "uuid"
  }
}
```

---

### 1.2 Approve Expense
**Endpoint:** `POST /api/v1/expenses/{expenseId}/approve`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "status": "approved",
    "approved_by": {
      "id": "uuid",
      "email": "admin@example.com"
    },
    "approved_at": "2026-01-15T11:00:00Z",
    "main_balance": 1000.00,
    "splits": [
      {
        "user_id": "uuid",
        "amount": 125.00
      }
    ]
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

---

### 1.3 Reject Expense
**Endpoint:** `POST /api/v1/expenses/{expenseId}/reject`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Request Body:**
```json
{
  "rejection_reason": "Receipt not provided or amount mismatch"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "status": "rejected",
    "rejection_reason": "Receipt not provided or amount mismatch",
    "rejected_by": {
      "id": "uuid",
      "email": "admin@example.com"
    },
    "rejected_at": "2026-01-15T11:00:00Z"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Error Responses:**
- `422 Unprocessable Entity` - Rejection reason required
- `403 Forbidden` - Not an admin
- `404 Not Found` - Expense not found
- `409 Conflict` - Already approved/rejected

---

## 2. Database Schema

### 2.1 Uses Existing Tables
- `transactions` (update status, approved_by, rejection_reason)
- `expense_splits` (created on approval)
- `group_members` (update individual_balance)

---

## 3. Business Logic Implementation

### 3.1 Service: ExpenseApprovalService
**Location:** `app/Services/Financial/ExpenseApprovalService.php`

**Methods:**
- `approveExpense(Transaction $expense, User $admin): Transaction`
- `rejectExpense(Transaction $expense, User $admin, string $reason): Transaction`

**Key Logic (Approval):**
1. Verify admin authorization
2. Check expense is pending
3. Get all active group members
4. Calculate split amount (expense / member_count)
5. Create expense_splits records
6. Update group.main_balance (decrease)
7. Update each member's individual_balance (decrease)
8. Update transaction status
9. Dispatch ExpenseApproved event

**Key Logic (Rejection):**
1. Verify admin authorization
2. Validate rejection reason (required)
3. Check expense is pending
4. Update transaction status to 'rejected'
5. Set rejection_reason
6. Dispatch ExpenseRejected event
7. No balance changes

---

### 3.2 Action: ApproveExpenseAction
**Location:** `app/Actions/Financial/ApproveExpenseAction.php`

**Responsibilities:**
- Validate admin authorization
- Check expense status
- Calculate splits
- Update balances atomically
- Create split records
- Dispatch events
- Return updated expense

---

### 3.3 Action: RejectExpenseAction
**Location:** `app/Actions/Financial/RejectExpenseAction.php`

**Responsibilities:**
- Validate admin authorization
- Validate rejection reason
- Check expense status
- Update transaction
- Dispatch events
- Return updated expense

---

## 4. Technical Implementation Details

### 4.1 Expense Splitting Logic
```php
function splitExpense(Transaction $expense, Group $group): void
{
    $members = GroupMember::where('group_id', $group->id)
        ->where('is_active', true)
        ->get();
    
    $memberCount = $members->count();
    $splitAmount = $expense->amount / $memberCount;
    
    DB::transaction(function () use ($expense, $members, $splitAmount) {
        foreach ($members as $member) {
            ExpenseSplit::create([
                'transaction_id' => $expense->id,
                'user_id' => $member->user_id,
                'amount' => $splitAmount,
            ]);
            
            // Update individual balance
            $member->individual_balance -= $splitAmount;
            $member->save();
        }
    });
}
```

### 4.2 Balance Update (Atomic)
```php
DB::transaction(function () use ($group, $expense) {
    $group->lockForUpdate();
    $group->main_balance -= $expense->amount;
    $group->last_activity_at = now();
    $group->save();
    
    // Split expense and update member balances
    $this->splitExpense($expense, $group);
});
```

### 4.3 Immutability
- Once approved/rejected, transaction cannot be modified
- Database constraints prevent status changes
- Application logic enforces immutability

---

## 5. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `APP001` | 403 | Only admins can approve expenses |
| `APP002` | 404 | Expense not found |
| `APP003` | 409 | Expense already approved/rejected |
| `APP004` | 422 | Rejection reason is required |
| `APP005` | 422 | Expense is not pending |

### 5.2 Error Response Format
```json
{
  "error": {
    "message": "Validation failed",
    "code": "APP004",
    "fields": [
      {
        "field": "rejection_reason",
        "message": "Rejection reason is required"
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
2. **Immutability:** Cannot modify approved/rejected expenses
3. **Atomicity:** Database transactions for balance updates
4. **Validation:** Rejection reason required
5. **Audit Trail:** All actions logged

---

## 7. Testing Strategy

### 7.1 Unit Tests
- Split calculation
- Balance updates
- Immutability checks

### 7.2 Feature Tests
- Successful approval
- Successful rejection
- Authorization checks
- Rejection reason validation
- Balance accuracy

### 7.3 Integration Tests
- Atomic transactions
- Split creation
- Balance consistency

---

## 8. Events & Listeners

### 8.1 ExpenseApproved Event
**Location:** `app/Events/ExpenseApproved.php`

**Listeners:**
- Update unread counts for all members
- Broadcast real-time update
- Send notifications

### 8.2 ExpenseRejected Event
**Location:** `app/Events/ExpenseRejected.php`

**Listeners:**
- Notify submitter
- Update unread counts
- Broadcast real-time update

---

## 9. Dependencies

- FIN-002: Expense Submission
- FIN-004: Main Balance Calculation
- FIN-005: Individual Balance Calculation
- GRP-003: Admin Role Management
- NOTIF-001: Real-Time Notifications

---

## 10. Related Documentation

- [FIN-002 Technical Spec](./FIN-002-expense-submission-api.md) - Expense Submission
- [FIN-004 Technical Spec](./FIN-004-main-balance-calculation-api.md) - Main Balance Calculation
- [FIN-005 Technical Spec](./FIN-005-individual-balance-calculation-api.md) - Individual Balance Calculation
