# GRP-003: Admin Role Management - Technical Specification

**Feature ID:** GRP-003  
**Category:** Group Management  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Promote Member to Admin
**Endpoint:** `POST /api/v1/groups/{groupId}/members/{memberId}/promote`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "group_id": "uuid",
    "user_id": "uuid",
    "role": "admin",
    "updated_at": "2026-01-15T10:30:00Z"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Error Responses:**
- `403 Forbidden` - Not an admin
- `404 Not Found` - Group or member not found
- `422 Unprocessable Entity` - Already an admin

---

### 1.2 Revoke Admin Status
**Endpoint:** `POST /api/v1/groups/{groupId}/members/{memberId}/revoke-admin`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "group_id": "uuid",
    "user_id": "uuid",
    "role": "member",
    "updated_at": "2026-01-15T10:30:00Z"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Error Responses:**
- `403 Forbidden` - Not an admin or cannot revoke last admin
- `404 Not Found` - Group or member not found
- `422 Unprocessable Entity` - Not an admin or last admin

---

### 1.3 Get Group Members
**Endpoint:** `GET /api/v1/groups/{groupId}/members`

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
      "user": {
        "id": "uuid",
        "email": "user@example.com"
      },
      "role": "admin",
      "individual_balance": "100.00",
      "joined_at": "2026-01-15T10:00:00Z"
    }
  ],
  "meta": {
    "request_id": "uuid"
  }
}
```

---

## 2. Database Schema

### 2.1 Uses Existing Tables
- `group_members` (from GRP-001)
- `audit_logs` (new table for role changes)

### 2.2 Audit Logs Table
```sql
CREATE TABLE audit_logs (
    id CHAR(36) PRIMARY KEY,
    user_id CHAR(36) NOT NULL,
    group_id CHAR(36) NOT NULL,
    action VARCHAR(50) NOT NULL,
    target_user_id CHAR(36) NULL,
    old_value JSON NULL,
    new_value JSON NULL,
    ip_address VARCHAR(45) NULL,
    user_agent TEXT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user_id (user_id),
    INDEX idx_group_id (group_id),
    INDEX idx_action (action),
    INDEX idx_created_at (created_at),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE RESTRICT,
    FOREIGN KEY (group_id) REFERENCES groups(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## 3. Business Logic Implementation

### 3.1 Service: RoleManagementService
**Location:** `app/Services/Group/RoleManagementService.php`

**Methods:**
- `promoteToAdmin(Group $group, GroupMember $member, User $actor): GroupMember`
- `revokeAdmin(Group $group, GroupMember $member, User $actor): GroupMember`
- `ensureAtLeastOneAdmin(Group $group): void`

**Key Logic:**
1. Verify actor is admin
2. Check target is member of group
3. For promotion: Check not already admin
4. For revocation: Ensure not last admin
5. Update role in transaction
6. Log audit trail
7. Dispatch RoleChanged event

---

### 3.2 Action: PromoteToAdminAction
**Location:** `app/Actions/Group/PromoteToAdminAction.php`

**Responsibilities:**
- Validate admin authorization
- Check member exists
- Check not already admin
- Update role
- Log audit trail
- Dispatch event
- Return updated member

---

### 3.3 Action: RevokeAdminAction
**Location:** `app/Actions/Group/RevokeAdminAction.php`

**Responsibilities:**
- Validate admin authorization
- Check member exists and is admin
- Ensure not last admin
- Update role
- Log audit trail
- Dispatch event
- Return updated member

---

## 4. Technical Implementation Details

### 4.1 Last Admin Check
```php
function ensureAtLeastOneAdmin(Group $group): void
{
    $adminCount = GroupMember::where('group_id', $group->id)
        ->where('role', 'admin')
        ->where('is_active', true)
        ->count();
    
    if ($adminCount <= 1) {
        throw new CannotRevokeLastAdminException();
    }
}
```

### 4.2 Audit Logging
```php
AuditLog::create([
    'user_id' => $actor->id,
    'group_id' => $group->id,
    'action' => 'role_changed',
    'target_user_id' => $member->user_id,
    'old_value' => ['role' => $oldRole],
    'new_value' => ['role' => $newRole],
    'ip_address' => request()->ip(),
    'user_agent' => request()->userAgent(),
]);
```

---

## 5. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `ROLE001` | 403 | Only admins can change roles |
| `ROLE002` | 404 | Member not found |
| `ROLE003` | 422 | User is already an admin |
| `ROLE004` | 422 | Cannot revoke last admin |
| `ROLE005` | 422 | User is not an admin |

---

## 6. Security Considerations

1. **Authorization:** Strict admin-only checks
2. **Last Admin Protection:** Cannot revoke last admin
3. **Audit Trail:** All role changes logged
4. **Transaction Safety:** Atomic role updates
5. **Event Tracking:** IP and user agent logged

---

## 7. Testing Strategy

### 7.1 Unit Tests
- Last admin check logic
- Role update logic
- Audit logging

### 7.2 Feature Tests
- Successful promotion
- Successful revocation
- Last admin protection
- Authorization checks
- Audit log creation

---

## 8. Events & Listeners

### 8.1 RoleChanged Event
**Location:** `app/Events/RoleChanged.php`

**Listeners:**
- Send notification to affected user
- Update real-time counts

---

## 9. Dependencies

- GRP-001: Group Creation
- GRP-002: Member Invitation
- NOTIF-001: Real-Time Notifications
- Audit logging system

---

## 10. Related Documentation

- [GRP-001 Technical Spec](./GRP-001-technical.md) - Group Creation
- [GRP-002 Technical Spec](./GRP-002-technical.md) - Member Invitation
- [NOTIF-001 Technical Spec](../notifications/NOTIF-001-technical.md) - Notifications
