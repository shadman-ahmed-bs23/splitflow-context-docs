# FIN-004: Main Balance Calculation - Technical Specification

**Feature ID:** FIN-004  
**Category:** Financial Operations  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Get Group Main Balance
**Endpoint:** `GET /api/v1/groups/{groupId}/balance`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": {
    "group_id": "uuid",
    "main_balance": 1500.00,
    "currency": "BDT",
    "formatted_balance": "৳1,500.00",
    "last_updated": "2026-01-15T10:30:00Z"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Note:** Main balance is also included in group detail endpoint.

---

## 2. Database Schema

### 2.1 Groups Table
- `main_balance DECIMAL(15, 2)` - Stored value
- Updated on every deposit/expense approval

### 2.2 Calculation Method
**Option 1:** Stored value (current implementation)
- Updated atomically with transactions
- Fast reads, requires careful updates

**Option 2:** Computed value (alternative)
- Calculated from transactions table
- Slower reads, always accurate

---

## 3. Business Logic Implementation

### 3.1 Service: BalanceService
**Location:** `app/Services/Financial/BalanceService.php`

**Methods:**
- `getMainBalance(Group $group): array`
- `recalculateMainBalance(Group $group): void` (for consistency checks)

**Calculation Formula:**
```
main_balance = SUM(deposits) - SUM(approved_expenses)
```

---

### 3.2 Balance Update Strategy
**On Deposit:**
```php
$group->main_balance += $deposit->amount;
```

**On Expense Approval:**
```php
$group->main_balance -= $expense->amount;
```

**Atomic Updates:**
- All balance updates in database transactions
- Row-level locking on groups table
- Prevents race conditions

---

### 3.3 Consistency Check
**Scheduled Job:** `app/Console/Commands/RecalculateBalances.php`

Runs daily to verify balance accuracy:
```php
$calculatedBalance = Transaction::where('group_id', $group->id)
    ->where('type', 'deposit')
    ->where('status', 'approved')
    ->sum('amount')
    - Transaction::where('group_id', $group->id)
        ->where('type', 'expense')
        ->where('status', 'approved')
        ->sum('amount');

if ($calculatedBalance != $group->main_balance) {
    // Log discrepancy and fix
    $group->main_balance = $calculatedBalance;
    $group->save();
}
```

---

## 4. Technical Implementation Details

### 4.1 Decimal Precision
- Stored as `DECIMAL(15, 2)`
- Calculations use database functions or `bcmath`
- Avoids floating-point errors

### 4.2 Real-time Updates
- WebSocket broadcast on balance change
- All group members receive update
- Frontend updates immediately

### 4.3 Color Coding (Frontend)
- Positive: Green (#10B981)
- Zero: Gray (#6B7280)
- Negative: Red (#EF4444)

---

## 5. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `BAL001` | 404 | Group not found |
| `BAL002` | 403 | Not a member |

### 5.2 Error Response Format
```json
{
  "error": {
    "message": "Group not found",
    "code": "BAL001",
    "fields": []
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

---

## 6. Security Considerations

1. **Authorization:** Member check
2. **Precision:** Decimal calculations
3. **Atomicity:** Database transactions
4. **Consistency:** Scheduled verification

---

## 7. Testing Strategy

### 7.1 Unit Tests
- Balance calculation
- Decimal precision
- Update logic

### 7.2 Feature Tests
- Balance accuracy after deposits
- Balance accuracy after approvals
- Negative balance handling
- Concurrent updates

### 7.3 Integration Tests
- Transaction consistency
- Real-time updates

---

## 8. Dependencies

- FIN-001: Deposit Submission
- FIN-002: Expense Submission
- FIN-003: Expense Approval
- Database (groups, transactions tables)

---

## 9. Related Documentation

- [FIN-001 Technical Spec](./FIN-001-technical.md) - Deposits
- [FIN-003 Technical Spec](./FIN-003-technical.md) - Approvals
- [UI-002 Technical Spec](../ui/UI-002-technical.md) - Real-time Updates
