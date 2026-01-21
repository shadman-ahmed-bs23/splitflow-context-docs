# FIN-005: Individual Balance Calculation - Technical Specification

**Feature ID:** FIN-005  
**Category:** Financial Operations  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Get Individual Balance
**Endpoint:** `GET /api/v1/groups/{groupId}/members/me/balance`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": {
    "user_id": "uuid",
    "group_id": "uuid",
    "individual_balance": 250.00,
    "currency": "BDT",
    "formatted_balance": "৳250.00",
    "breakdown": {
      "total_deposits": 500.00,
      "total_expense_shares": 250.00,
      "net_balance": 250.00
    },
    "last_updated": "2026-01-15T10:30:00Z"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

---

## 2. Database Schema

### 2.1 Group Members Table
- `individual_balance DECIMAL(15, 2)` - Stored value
- Updated on deposit (if admin) and expense approval

### 2.2 Calculation Method
**Formula:**
```
individual_balance = (user_deposits) - (user_expense_shares)
```

**From Tables:**
- Deposits: `transactions` where `type='deposit'` and `submitted_by=user_id`
- Expense Shares: `expense_splits` where `user_id=user_id`

---

## 3. Business Logic Implementation

### 3.1 Service: IndividualBalanceService
**Location:** `app/Services/Financial/IndividualBalanceService.php`

**Methods:**
- `getIndividualBalance(User $user, Group $group): array`
- `recalculateIndividualBalance(User $user, Group $group): void`
- `getBalanceBreakdown(User $user, Group $group): array`

**Calculation Logic:**
```php
$deposits = Transaction::where('group_id', $group->id)
    ->where('type', 'deposit')
    ->where('status', 'approved')
    ->where('submitted_by', $user->id)
    ->sum('amount');

$expenseShares = ExpenseSplit::whereHas('transaction', function ($query) use ($group) {
    $query->where('group_id', $group->id)
          ->where('status', 'approved');
})
->where('user_id', $user->id)
->sum('amount');

$balance = $deposits - $expenseShares;
```

---

### 3.2 Balance Update Strategy
**On Deposit (Admin only):**
```php
$member->individual_balance += $deposit->amount;
$member->save();
```

**On Expense Approval:**
```php
// Update all members' balances via expense_splits
foreach ($splits as $split) {
    $member = GroupMember::where('group_id', $group->id)
        ->where('user_id', $split->user_id)
        ->first();
    
    $member->individual_balance -= $split->amount;
    $member->save();
}
```

---

### 3.3 Balance Breakdown
**Components:**
1. Total Deposits: Sum of user's deposits
2. Total Expense Shares: Sum of user's expense splits
3. Net Balance: Deposits - Expense Shares

---

## 4. Technical Implementation Details

### 4.1 Historical Accuracy
- Expense splits calculated at time of approval
- Member count at approval time determines split
- Historical splits remain unchanged even if members leave

### 4.2 Decimal Precision
- Same as main balance (DECIMAL 15,2)
- Use database functions for calculations

### 4.3 Real-time Updates
- Updated when expenses approved
- WebSocket broadcast to affected users
- Frontend updates immediately

---

## 5. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `IBAL001` | 404 | Group not found |
| `IBAL002` | 403 | Not a member |

### 5.2 Error Response Format
```json
{
  "error": {
    "message": "Access denied",
    "code": "IBAL002",
    "fields": [
      {
        "field": "authorization",
        "message": "Not a member"
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

1. **Authorization:** Member check
2. **Privacy:** Users can only see their own balance
3. **Precision:** Decimal calculations
4. **Consistency:** Scheduled verification

---

## 7. Testing Strategy

### 7.1 Unit Tests
- Balance calculation
- Breakdown calculation
- Historical accuracy

### 7.2 Feature Tests
- Balance accuracy
- Breakdown accuracy
- Update on approval
- Member count changes

---

## 8. Dependencies

- FIN-001: Deposit Submission
- FIN-003: Expense Approval
- Database (group_members, transactions, expense_splits)

---

## 9. Related Documentation

- [FIN-001 Technical Spec](./FIN-001-deposit-submission-api.md) - Deposits
- [FIN-003 Technical Spec](./FIN-003-expense-approval-api.md) - Approvals
- [FIN-006 Technical Spec](./FIN-006-settlement-calculation-api.md) - Settlement
