# RPT-002: Individual Balance Tracking - Technical Specification

**Feature ID:** RPT-002  
**Category:** Reporting & Analytics  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Get Personal Dashboard
**Endpoint:** `GET /api/v1/groups/{groupId}/members/me/dashboard`

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
    "current_balance": 250.00,
    "currency": "BDT",
    "transaction_history": [
      {
        "id": "uuid",
        "type": "deposit",
        "amount": 500.00,
        "date_time": "2026-01-10T10:00:00Z",
        "description": "Initial deposit"
      },
      {
        "id": "uuid",
        "type": "expense_share",
        "amount": -125.00,
        "date_time": "2026-01-12T14:00:00Z",
        "description": "Lunch at restaurant"
      }
    ],
    "balance_over_time": [
      {"date": "2026-01-10", "balance": 500.00},
      {"date": "2026-01-12", "balance": 375.00},
      {"date": "2026-01-15", "balance": 250.00}
    ],
    "breakdown": {
      "total_deposits": 500.00,
      "total_expense_shares": 250.00,
      "net_balance": 250.00
    }
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

---

## 2. Database Schema

### 2.1 Uses Existing Tables
- `transactions` - User's deposits
- `expense_splits` - User's expense shares
- `group_members` - Current balance

---

## 3. Business Logic Implementation

### 3.1 Service: PersonalDashboardService
**Location:** `app/Services/Reporting/PersonalDashboardService.php`

**Methods:**
- `getDashboard(User $user, Group $group): array`
- `getTransactionHistory(User $user, Group $group, array $filters): Collection`
- `getBalanceOverTime(User $user, Group $group): array`

**Key Logic:**
1. Verify membership
2. Get user's deposits
3. Get user's expense splits
4. Calculate balance over time
5. Generate breakdown
6. Return dashboard data

---

### 3.2 Balance Over Time Calculation
```php
function getBalanceOverTime(User $user, Group $group): array
{
    $deposits = Transaction::where('group_id', $group->id)
        ->where('type', 'deposit')
        ->where('status', 'approved')
        ->where('submitted_by', $user->id)
        ->orderBy('date_time')
        ->get();
    
    $expenseSplits = ExpenseSplit::whereHas('transaction', function ($query) use ($group) {
        $query->where('group_id', $group->id)
              ->where('status', 'approved');
    })
    ->where('user_id', $user->id)
    ->with('transaction')
    ->orderBy('created_at')
    ->get();
    
    $balance = 0;
    $timeline = [];
    
    // Combine and sort by date
    $events = collect($deposits)->map(fn($t) => [
        'date' => $t->date_time,
        'type' => 'deposit',
        'amount' => $t->amount,
    ])->merge(
        collect($expenseSplits)->map(fn($s) => [
            'date' => $s->transaction->date_time,
            'type' => 'expense',
            'amount' => -$s->amount,
        ])
    )->sortBy('date');
    
    foreach ($events as $event) {
        $balance += $event['amount'];
        $timeline[] = [
            'date' => $event['date'],
            'balance' => $balance,
        ];
    }
    
    return $timeline;
}
```

---

## 4. Technical Implementation Details

### 4.1 Transaction History
- Combines deposits and expense shares
- Sorted by date_time
- Paginated (20 per page)

### 4.2 Chart Data
- Balance over time: Line chart
- Contribution vs spending: Pie chart
- Frontend renders using Chart.js

---

## 5. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `DASH001` | 404 | Group not found |
| `DASH002` | 403 | Not a member |

### 5.2 Error Response Format
```json
{
  "error": {
    "message": "Access denied",
    "code": "DASH002",
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
2. **Privacy:** Users only see their own data
3. **Data Isolation:** Filtered by user_id

---

## 7. Testing Strategy

### 7.1 Unit Tests
- Balance calculation
- Timeline generation
- Breakdown calculation

### 7.2 Feature Tests
- Dashboard retrieval
- Transaction history
- Balance over time accuracy

---

## 8. Dependencies

- FIN-001: Deposit Submission
- FIN-002: Expense Submission
- FIN-005: Individual Balance Calculation
- Database (transactions, expense_splits tables)

---

## 9. Related Documentation

- [FIN-005 Technical Spec](../financial-operations/FIN-005-technical.md) - Individual Balance
- [RPT-001 Technical Spec](./RPT-001-technical.md) - Spending Reports
