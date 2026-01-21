# NOTIF-001: Real-Time Notifications (Unread Activity Counts) - Technical Specification

**Feature ID:** NOTIF-001  
**Category:** Notifications & Communication  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Get Unread Counts
**Endpoint:** `GET /api/v1/groups/unread-counts`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": [
    {
      "group_id": "uuid",
      "unread_count": 3
    },
    {
      "group_id": "uuid",
      "unread_count": 0
    }
  ],
  "meta": {
    "request_id": "uuid"
  }
}
```

---

### 1.2 Mark Group as Seen
**Endpoint:** `POST /api/v1/groups/{groupId}/mark-seen`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": {
    "group_id": "uuid",
    "unread_count": 0,
    "last_seen_at": "2026-01-15T10:30:00Z"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

---

## 2. Database Schema

### 2.1 User Group Activity Table
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

### 2.2 Activity Sequence Tracking
- `groups` table: `last_activity_sequence BIGINT` - Increments on each activity
- Used to compare with `user_group_activity.last_activity_sequence`

---

## 3. Business Logic Implementation

### 3.1 Service: UnreadCountService
**Location:** `app/Services/Notification/UnreadCountService.php`

**Methods:**
- `getUnreadCount(User $user, Group $group): int`
- `getAllUnreadCounts(User $user): array`
- `markGroupAsSeen(User $user, Group $group): void`
- `incrementActivitySequence(Group $group): void`

**Calculation Logic:**
```php
function getUnreadCount(User $user, Group $group): int
{
    $userActivity = UserGroupActivity::firstOrCreate([
        'user_id' => $user->id,
        'group_id' => $group->id,
    ]);
    
    if (!$userActivity->last_seen_at) {
        // User hasn't seen group yet, return all activities
        return Transaction::where('group_id', $group->id)
            ->where('created_at', '>', $group->created_at)
            ->count();
    }
    
    // Count activities after last_seen_at
    return Transaction::where('group_id', $group->id)
        ->where('created_at', '>', $userActivity->last_seen_at)
        ->count();
}
```

---

### 3.2 Event Listeners
**Location:** `app/Listeners/IncrementUnreadCount.php`

**Triggers:**
- DepositCreated
- ExpenseSubmitted (pending)
- ExpenseApproved
- ExpenseRejected
- MemberJoined
- RoleChanged

**Logic:**
```php
public function handle($event)
{
    $group = $event->group;
    
    // Increment activity sequence
    $group->last_activity_sequence++;
    $group->last_activity_at = now();
    $group->save();
    
    // Broadcast to all group members via WebSocket
    broadcast(new UnreadCountUpdated($group));
}
```

---

## 4. Technical Implementation Details

### 4.1 WebSocket Implementation
**Backend:** Laravel Broadcasting (Pusher/Socket.io)

**Channels:**
- `group.{groupId}` - Group-specific updates
- `user.{userId}` - User-specific updates

**Events:**
- `UnreadCountUpdated` - Broadcasted to group members

### 4.2 Fallback to Polling
**Frontend:** If WebSocket unavailable, poll every 10 seconds
```javascript
// Polling fallback
setInterval(() => {
  fetch('/api/v1/groups/unread-counts')
    .then(res => res.json())
    .then(data => updateUnreadCounts(data));
}, 10000);
```

---

## 5. Activity Types That Increment Count

1. **New Pending Expense** - Admins only
2. **Expense Approved** - All members
3. **Expense Rejected** - Submitter only
4. **New Deposit** - All members
5. **New Member Joined** - All members
6. **Admin Role Change** - Affected user only

---

## 6. Error Handling

### 6.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `UNREAD001` | 404 | Group not found |
| `UNREAD002` | 403 | Not a member |

---

## 7. Security Considerations

1. **Authorization:** Member check for all endpoints
2. **Privacy:** Users only see their own unread counts
3. **Real-time:** WebSocket authentication required
4. **Rate Limiting:** Polling endpoints rate-limited

---

## 8. Testing Strategy

### 8.1 Unit Tests
- Unread count calculation
- Activity sequence increment
- Mark as seen logic

### 8.2 Feature Tests
- Count increments on events
- Mark as seen resets count
- Multi-device sync
- WebSocket delivery

### 8.3 Integration Tests
- Real-time updates
- Polling fallback
- Event dispatching

---

## 9. Dependencies

- All feature modules (for event triggers)
- WebSocket server (Laravel Broadcasting)
- Database (user_group_activity, groups tables)

---

## 10. Related Documentation

- [GRP-004 Technical Spec](../group-management/GRP-004-group-list-api.md) - Group List View
- [UI-002 Technical Spec](../ui/UI-002-real-time-updates.md) - Real-Time Updates
