# FIN-006: Settlement Calculation - Technical Specification

**Feature ID:** FIN-006  
**Category:** Financial Operations  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Get Settlement Recommendations
**Endpoint:** `GET /api/v1/groups/{groupId}/settlement`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": {
    "group_id": "uuid",
    "total_settlement_amount": "500.00",
    "recommendations": [
      {
        "from_user": {
          "id": "uuid",
          "email": "user1@example.com",
          "balance": "-200.00"
        },
        "to_user": {
          "id": "uuid",
          "email": "user2@example.com",
          "balance": "300.00"
        },
        "amount": "200.00"
      },
      {
        "from_user": {
          "id": "uuid",
          "email": "user1@example.com",
          "balance": "-200.00"
        },
        "to_user": {
          "id": "uuid",
          "email": "user3@example.com",
          "balance": "200.00"
        },
        "amount": "100.00"
      }
    ],
    "summary": {
      "users_owing": 2,
      "users_owed": 2,
      "total_transactions": 2
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
- `group_members` (individual_balance)
- No new tables required

---

## 3. Business Logic Implementation

### 3.1 Service: SettlementService
**Location:** `app/Services/Financial/SettlementService.php`

**Methods:**
- `calculateSettlement(Group $group): array`
- `minimizeTransactions(array $balances): array`

**Algorithm:**
1. Get all members' individual balances
2. Separate into:
   - Owing (negative balance)
   - Owed (positive balance)
3. Use greedy algorithm to minimize transactions
4. Return payment recommendations

---

### 3.2 Minimization Algorithm
**Greedy Approach:**
```php
function minimizeTransactions(array $balances): array
{
    $owing = array_filter($balances, fn($b) => $b < 0);
    $owed = array_filter($balances, fn($b) => $b > 0);
    
    $transactions = [];
    
    foreach ($owing as $userId => $amount) {
        $remaining = abs($amount);
        
        foreach ($owed as $targetId => $targetAmount) {
            if ($remaining <= 0) break;
            if ($targetAmount <= 0) continue;
            
            $transfer = min($remaining, $targetAmount);
            $transactions[] = [
                'from' => $userId,
                'to' => $targetId,
                'amount' => $transfer
            ];
            
            $remaining -= $transfer;
            $owed[$targetId] -= $transfer;
        }
    }
    
    return $transactions;
}
```

---

## 4. Technical Implementation Details

### 4.1 Settlement Calculation
- Based on individual balances
- Minimizes number of transactions
- Accounts for main balance state
- Informational only (no automatic transfers)

### 4.2 Real-time Updates
- Recalculates when balances change
- WebSocket updates on expense approval

---

## 5. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `SET001` | 404 | Group not found |
| `SET002` | 403 | Not a member |

---

## 6. Security Considerations

1. **Authorization:** Member check
2. **Privacy:** All members can view settlement
3. **Accuracy:** Mathematical verification

---

## 7. Testing Strategy

### 7.1 Unit Tests
- Minimization algorithm
- Edge cases (all positive, all negative)
- Calculation accuracy

### 7.2 Feature Tests
- Settlement calculation
- Transaction minimization
- Real-time updates

---

## 8. Dependencies

- FIN-005: Individual Balance Calculation
- Database (group_members table)

---

## 9. Related Documentation

- [FIN-005 Technical Spec](./FIN-005-technical.md) - Individual Balance
- [FIN-004 Technical Spec](./FIN-004-technical.md) - Main Balance
