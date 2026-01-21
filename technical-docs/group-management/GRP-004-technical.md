# GRP-004: Group List View - Technical Specification

**Feature ID:** GRP-004  
**Category:** Group Management  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Get User's Groups
**Endpoint:** `GET /api/v1/groups`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Query Parameters:**
- `search` (optional) - Search by group name
- `page` (optional) - Page number (default: 1)
- `per_page` (optional) - Items per page (default: 20, max: 100)

**Response (200 OK):**
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "Weekend Trip",
      "last_activity_at": "2026-01-15T10:30:00Z",
      "unread_count": 3,
      "created_at": "2026-01-10T08:00:00Z"
    }
  ],
  "meta": {
    "pagination": {
      "current_page": 1,
      "per_page": 20,
      "total": 5,
      "last_page": 1
    },
    "request_id": "uuid"
  }
}
```

**Error Responses:**
- `401 Unauthorized` - Invalid token

---

### 1.2 Get Single Group
**Endpoint:** `GET /api/v1/groups/{groupId}`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "name": "Weekend Trip",
    "description": "Trip to Cox's Bazar",
    "currency": "BDT",
    "main_balance": "1500.00",
    "unread_count": 0,
    "last_activity_at": "2026-01-15T10:30:00Z",
    "created_at": "2026-01-10T08:00:00Z"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

---

## 2. Database Schema

### 2.1 Uses Existing Tables
- `groups`
- `group_members`
- `transactions` (for last_activity_at)
- `user_group_activity` (for unread counts)

### 2.2 User Group Activity Table (Unread Tracking)
```sql
CREATE TABLE user_group_activity (
    id CHAR(36) PRIMARY KEY,
    user_id CHAR(36) NOT NULL,
    group_id CHAR(36) NOT NULL,
    last_seen_at TIMESTAMP NULL,
    last_activity_sequence BIGINT DEFAULT 0 NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY unique_user_group (user_id, group_id),
    INDEX idx_user_id (user_id),
    INDEX idx_group_id (group_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (group_id) REFERENCES groups(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## 3. Business Logic Implementation

### 3.1 Service: GroupListService
**Location:** `app/Services/Group/GroupListService.php`

**Methods:**
- `getUserGroups(User $user, array $filters = []): Collection`
- `getGroup(User $user, string $groupId): Group`
- `markGroupAsSeen(User $user, Group $group): void`

**Key Logic:**
1. Filter groups where user is active member
2. Exclude archived groups by default
3. Sort by last_activity_at DESC
4. Calculate unread_count per group
5. Apply search filter (case-insensitive)
6. Paginate results

---

### 3.2 Query Optimization
**Location:** `app/Repositories/GroupRepository.php`

**Efficient Query:**
```php
Group::whereHas('members', function ($query) use ($user) {
    $query->where('user_id', $user->id)
          ->where('is_active', true);
})
->where('is_archived', false)
->with(['members' => function ($query) use ($user) {
    $query->where('user_id', $user->id);
}])
->withCount(['transactions' => function ($query) {
    $query->where('created_at', '>', function ($subQuery) {
        // Compare with last_seen_at
    });
}])
->orderBy('last_activity_at', 'desc')
->paginate($perPage);
```

---

### 3.3 Unread Count Calculation
**Location:** `app/Services/Group/UnreadCountService.php`

**Logic:**
1. Get last_seen_at or last_activity_sequence for user+group
2. Count activities after that timestamp/sequence
3. Return count

---

## 4. Technical Implementation Details

### 4.1 Last Activity Tracking
- Updated on every transaction (deposit/expense)
- Stored in `groups.last_activity_at`
- Indexed for fast sorting

### 4.2 Unread Count Calculation
```php
function getUnreadCount(User $user, Group $group): int
{
    $userActivity = UserGroupActivity::where('user_id', $user->id)
        ->where('group_id', $group->id)
        ->first();
    
    if (!$userActivity || !$userActivity->last_seen_at) {
        // Return all activities count
        return Transaction::where('group_id', $group->id)
            ->where('created_at', '>', $group->created_at)
            ->count();
    }
    
    return Transaction::where('group_id', $group->id)
        ->where('created_at', '>', $userActivity->last_seen_at)
        ->count();
}
```

### 4.3 Mark as Seen
**Endpoint:** `POST /api/v1/groups/{groupId}/mark-seen`

Updates `user_group_activity.last_seen_at` to current timestamp.

---

## 5. Caching Strategy

### 5.1 Cache Keys
- `user:{id}:groups:list` - Cached group list
- `user:{id}:group:{groupId}:unread` - Cached unread count

### 5.2 Cache Invalidation
- On new transaction
- On group creation/update
- On member join/leave
- TTL: 5 minutes

---

## 6. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `LIST001` | 401 | Unauthorized |
| `LIST002` | 404 | Group not found |

---

## 7. Performance Considerations

1. **Indexes:** Ensure indexes on foreign keys and sort columns
2. **Eager Loading:** Load relationships efficiently
3. **Pagination:** Limit results per page
4. **Caching:** Cache group lists with short TTL
5. **Query Optimization:** Use joins instead of N+1 queries

---

## 8. Testing Strategy

### 8.1 Unit Tests
- Unread count calculation
- Sorting logic
- Search filtering

### 8.2 Feature Tests
- Group list retrieval
- Search functionality
- Pagination
- Unread count accuracy
- Mark as seen

### 8.3 Performance Tests
- Query performance with many groups
- Cache effectiveness

---

## 9. Dependencies

- GRP-001: Group Creation
- FIN-001: Deposit Submission
- FIN-002: Expense Submission
- NOTIF-001: Real-Time Notifications
- UI-001: Mobile-Responsive Design

---

## 10. Related Documentation

- [GRP-001 Technical Spec](./GRP-001-technical.md) - Group Creation
- [FIN-001 Technical Spec](../financial-operations/FIN-001-technical.md) - Deposits
- [NOTIF-001 Technical Spec](../notifications/NOTIF-001-technical.md) - Notifications
