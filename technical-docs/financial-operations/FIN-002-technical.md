# FIN-002: Expense Submission - Technical Specification

**Feature ID:** FIN-002  
**Category:** Financial Operations  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Submit Expense
**Endpoint:** `POST /api/v1/groups/{groupId}/expenses`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Request Body:**
```json
{
  "amount": "500.00",
  "date_time": "2026-01-15T10:30:00Z",
  "description": "Lunch at restaurant"
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "group_id": "uuid",
    "type": "expense",
    "amount": "500.00",
    "date_time": "2026-01-15T10:30:00Z",
    "description": "Lunch at restaurant",
    "submitted_by": {
      "id": "uuid",
      "email": "member@example.com"
    },
    "status": "pending",
    "created_at": "2026-01-15T10:30:00Z"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Note:** If submitter is Admin, status will be "approved" and main_balance updated immediately.

**Error Responses:**
- `404 Not Found` - Group not found
- `422 Unprocessable Entity` - Validation errors
- `403 Forbidden` - Not a member

---

## 2. Database Schema

### 2.1 Uses Transactions Table
- Same table as deposits (from FIN-001)
- `type='expense'` distinguishes expenses

### 2.2 Expense Splits Table
```sql
CREATE TABLE expense_splits (
    id CHAR(36) PRIMARY KEY,
    transaction_id CHAR(36) NOT NULL,
    user_id CHAR(36) NOT NULL,
    amount DECIMAL(15, 2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_transaction_id (transaction_id),
    INDEX idx_user_id (user_id),
    FOREIGN KEY (transaction_id) REFERENCES transactions(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## 3. Business Logic Implementation

### 3.1 Service: ExpenseService
**Location:** `app/Services/Financial/ExpenseService.php`

**Methods:**
- `submitExpense(Group $group, User $user, array $data): Transaction`
- `autoApproveExpense(Transaction $expense): Transaction`

**Key Logic:**
1. Verify user is member of group
2. Validate amount (positive, max 2 decimals)
3. Validate date (not future)
4. Check if user is admin:
   - If admin: Auto-approve, update balances immediately
   - If member: Create with status='pending'
5. Dispatch ExpenseSubmitted event
6. Return transaction resource

---

### 3.2 Action: SubmitExpenseAction
**Location:** `app/Actions/Financial/SubmitExpenseAction.php`

**Responsibilities:**
- Validate membership
- Validate expense data
- Create transaction
- Auto-approve if admin
- Dispatch events
- Return transaction resource

---

### 3.3 Auto-Approval Logic (Admin)
```php
if ($user->isAdminOf($group)) {
    $expense->status = 'approved';
    $expense->approved_by = $user->id;
    $expense->approved_at = now();
    $expense->save();
    
    // Split and update balances
    $this->splitExpense($expense, $group);
    $this->updateBalances($expense, $group);
}
```

---

## 4. Technical Implementation Details

### 4.1 Status State Machine
- `pending` → `approved` or `rejected`
- Admin expenses: `pending` → `approved` (immediate)
- Member expenses: `pending` → (await approval)

### 4.2 Warning for Exceeding Balance
- Frontend shows warning if expense > main_balance
- Backend allows but logs warning
- No hard restriction

---

## 5. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `EXP001` | 403 | User is not a member of this group |
| `EXP002` | 422 | Amount must be positive |
| `EXP003` | 422 | Date cannot be in the future |
| `EXP004` | 422 | Invalid amount format |
| `EXP005` | 404 | Group not found |

---

## 6. Security Considerations

1. **Authorization:** Member check via Policy
2. **Validation:** Server-side validation
3. **Auto-approval:** Only for admins
4. **Audit Trail:** All expenses logged

---

## 7. Testing Strategy

### 7.1 Unit Tests
- Amount validation
- Date validation
- Auto-approval logic
- Status assignment

### 7.2 Feature Tests
- Member expense submission (pending)
- Admin expense submission (approved)
- Authorization checks
- Validation errors

---

## 8. Events & Listeners

### 8.1 ExpenseSubmitted Event
**Location:** `app/Events/ExpenseSubmitted.php`

**Listeners:**
- Notify admins if pending
- Update unread counts
- Broadcast real-time update

---

## 9. Dependencies

- GRP-001: Group Creation
- FIN-001: Deposit Submission
- FIN-003: Expense Approval Workflow
- Database (transactions, expense_splits tables)

---

## 10. Related Documentation

- [FIN-001 Technical Spec](./FIN-001-technical.md) - Deposit Submission
- [FIN-003 Technical Spec](./FIN-003-technical.md) - Expense Approval
- [FIN-004 Technical Spec](./FIN-004-technical.md) - Main Balance Calculation
