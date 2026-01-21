# RPT-001: Spending Reports - Technical Specification

**Feature ID:** RPT-001  
**Category:** Reporting & Analytics  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Generate Report
**Endpoint:** `POST /api/v1/groups/{groupId}/reports`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Request Body:**
```json
{
  "period": "monthly",
  "start_date": "2026-01-01",
  "end_date": "2026-01-31",
  "transaction_type": "both",
  "member_id": null
}
```

**Response (200 OK):**
```json
{
  "data": {
    "group_id": "uuid",
    "period": {
      "start_date": "2026-01-01",
      "end_date": "2026-01-31"
    },
    "summary": {
      "total_deposits": "5000.00",
      "total_expenses": "3500.00",
      "net_change": "1500.00",
      "currency": "BDT"
    },
    "transactions": [
      {
        "id": "uuid",
        "type": "deposit",
        "amount": "1500.00",
        "date_time": "2026-01-15T10:30:00Z",
        "description": "Initial fund",
        "submitted_by": "uuid"
      }
    ],
    "charts": {
      "daily_spending": [
        {"date": "2026-01-01", "amount": "500.00"},
        {"date": "2026-01-02", "amount": "300.00"}
      ],
      "by_member": [
        {"member_id": "uuid", "total": "1000.00"}
      ]
    }
  },
  "meta": {
    "request_id": "uuid",
    "generated_at": "2026-01-15T10:30:00Z"
  }
}
```

---

### 1.2 Export PDF
**Endpoint:** `GET /api/v1/groups/{groupId}/reports/{reportId}/pdf`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response:** PDF file download

---

## 2. Database Schema

### 2.1 Uses Existing Tables
- `transactions` - For transaction data
- `expense_splits` - For member-wise breakdown
- `group_members` - For member information

---

## 3. Business Logic Implementation

### 3.1 Service: ReportService
**Location:** `app/Services/Reporting/ReportService.php`

**Methods:**
- `generateReport(Group $group, User $user, array $filters): array`
- `exportToPdf(array $reportData): string`

**Key Logic:**
1. Verify admin authorization
2. Validate date range (max 1 year)
3. Query transactions with filters
4. Calculate totals and net change
5. Generate chart data
6. Return report data

---

### 3.2 Report Generation
```php
function generateReport(Group $group, array $filters): array
{
    $query = Transaction::where('group_id', $group->id)
        ->where('status', 'approved');
    
    // Apply date filter
    if (isset($filters['start_date'])) {
        $query->where('date_time', '>=', $filters['start_date']);
    }
    if (isset($filters['end_date'])) {
        $query->where('date_time', '<=', $filters['end_date']);
    }
    
    // Apply type filter
    if ($filters['transaction_type'] !== 'both') {
        $query->where('type', $filters['transaction_type']);
    }
    
    $transactions = $query->get();
    
    // Calculate summary
    $deposits = $transactions->where('type', 'deposit')->sum('amount');
    $expenses = $transactions->where('type', 'expense')->sum('amount');
    
    // Generate chart data
    $dailySpending = $this->generateDailySpendingChart($transactions);
    $byMember = $this->generateByMemberChart($transactions, $group);
    
    return [
        'summary' => [
            'total_deposits' => $deposits,
            'total_expenses' => $expenses,
            'net_change' => $deposits - $expenses,
        ],
        'transactions' => $transactions,
        'charts' => [
            'daily_spending' => $dailySpending,
            'by_member' => $byMember,
        ],
    ];
}
```

---

## 4. Technical Implementation Details

### 4.1 PDF Generation
**Library:** DomPDF or Snappy (wkhtmltopdf)

**Template:** Blade template for report layout
**Location:** `resources/views/reports/spending-report.blade.php`

### 4.2 Chart Data Generation
**Frontend:** Chart.js or similar
**Backend:** Returns data arrays, frontend renders charts

### 4.3 Performance Optimization
- Indexed queries on date_time
- Limit transaction history queries
- Cache report data (5 minutes TTL)

---

## 5. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `RPT001` | 403 | Only admins can generate reports |
| `RPT002` | 422 | Date range exceeds 1 year |
| `RPT003` | 422 | Invalid date range |
| `RPT004` | 404 | Group not found |

---

## 6. Security Considerations

1. **Authorization:** Admin-only
2. **Data Privacy:** Only group data
3. **Rate Limiting:** Report generation endpoints

---

## 7. Testing Strategy

### 7.1 Unit Tests
- Report calculation
- Chart data generation
- Date filtering

### 7.2 Feature Tests
- Report generation
- PDF export
- Authorization checks
- Filter validation

---

## 8. Dependencies

- FIN-001: Deposit Submission
- FIN-002: Expense Submission
- FIN-003: Expense Approval
- GRP-003: Admin Role Management
- PDF generation library

---

## 9. Related Documentation

- [RPT-002 Technical Spec](./RPT-002-technical.md) - Individual Balance Tracking
- [FIN-001 Technical Spec](../financial-operations/FIN-001-technical.md) - Deposits
