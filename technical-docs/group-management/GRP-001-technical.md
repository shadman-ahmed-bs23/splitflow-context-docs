# GRP-001: Group Creation - Technical Specification

**Feature ID:** GRP-001  
**Category:** Group Management  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Create Group
**Endpoint:** `POST /api/v1/groups`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Request Body:**
```json
{
  "name": "Weekend Trip",
  "description": "Trip to Cox's Bazar"
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "name": "Weekend Trip",
    "description": "Trip to Cox's Bazar",
    "currency": "BDT",
    "main_balance": "0.00",
    "is_archived": false,
    "created_by": "uuid",
    "created_at": "2026-01-15T10:30:00Z",
    "updated_at": "2026-01-15T10:30:00Z"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Error Responses:**
- `422 Unprocessable Entity` - Validation errors
- `401 Unauthorized` - Invalid token

---

### 1.2 Update Group
**Endpoint:** `PATCH /api/v1/groups/{groupId}`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Request Body:**
```json
{
  "name": "Updated Group Name",
  "description": "Updated description"
}
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "name": "Updated Group Name",
    "description": "Updated description",
    "currency": "BDT",
    "main_balance": "0.00",
    "is_archived": false,
    "updated_at": "2026-01-15T11:00:00Z"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Error Responses:**
- `403 Forbidden` - Not an admin
- `404 Not Found` - Group not found
- `422 Unprocessable Entity` - Validation errors

---

### 1.3 Archive Group
**Endpoint:** `POST /api/v1/groups/{groupId}/archive`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": {
    "id": "uuid",
    "is_archived": true,
    "archived_at": "2026-01-15T12:00:00Z"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

---

## 2. Database Schema

### 2.1 Groups Table
```sql
CREATE TABLE groups (
    id CHAR(36) PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    description VARCHAR(200) NULL,
    currency CHAR(3) DEFAULT 'BDT' NOT NULL,
    main_balance DECIMAL(15, 2) DEFAULT 0.00 NOT NULL,
    is_archived BOOLEAN DEFAULT FALSE NOT NULL,
    archived_at TIMESTAMP NULL,
    created_by CHAR(36) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_created_by (created_by),
    INDEX idx_is_archived (is_archived),
    INDEX idx_updated_at (updated_at),
    FOREIGN KEY (created_by) REFERENCES users(id) ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 2.2 Group Members Table
```sql
CREATE TABLE group_members (
    id CHAR(36) PRIMARY KEY,
    group_id CHAR(36) NOT NULL,
    user_id CHAR(36) NOT NULL,
    role ENUM('admin', 'member') DEFAULT 'member' NOT NULL,
    individual_balance DECIMAL(15, 2) DEFAULT 0.00 NOT NULL,
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE NOT NULL,
    UNIQUE KEY unique_group_user (group_id, user_id),
    INDEX idx_group_id (group_id),
    INDEX idx_user_id (user_id),
    INDEX idx_role (role),
    FOREIGN KEY (group_id) REFERENCES groups(id) ON DELETE RESTRICT,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## 3. Business Logic Implementation

### 3.1 Service: GroupService
**Location:** `app/Services/Group/GroupService.php`

**Methods:**
- `createGroup(User $user, array $data): Group`
- `updateGroup(Group $group, User $user, array $data): Group`
- `archiveGroup(Group $group, User $user): Group`

**Key Logic:**
1. Validate group name (max 50 chars, unique per user)
2. Create group with currency='BDT' (fixed)
3. Create group_member record with role='admin' for creator
4. Set initial main_balance = 0.00
5. Return group resource

---

### 3.2 Action: CreateGroupAction
**Location:** `app/Actions/Group/CreateGroupAction.php`

**Responsibilities:**
- Validate request data
- Check name uniqueness per user
- Create group in transaction
- Create admin membership
- Dispatch GroupCreated event
- Return group resource

---

### 3.3 Policy: GroupPolicy
**Location:** `app/Policies/GroupPolicy.php`

**Methods:**
- `update(User $user, Group $group): bool` - Must be admin
- `archive(User $user, Group $group): bool` - Must be admin
- `view(User $user, Group $group): bool` - Must be member

---

## 4. Technical Implementation Details

### 4.1 Group Name Uniqueness
**Validation Rule:** Custom rule checking uniqueness per user

```php
Rule::unique('groups')->where(function ($query) use ($user) {
    return $query->where('created_by', $user->id)
                 ->where('is_archived', false);
})
```

### 4.2 Currency Handling
- Fixed to 'BDT' (Bangladeshi Taka) for MVP
- Stored as ISO 4217 code
- Formatting handled in frontend

### 4.3 Archive Logic
- Soft delete: `is_archived = true`
- Archived groups excluded from default queries
- Read-only: No transactions allowed on archived groups

---

## 5. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `GRP001` | 422 | Group name is required |
| `GRP002` | 422 | Group name exceeds 50 characters |
| `GRP003` | 422 | Group name already exists for this user |
| `GRP004` | 422 | Description exceeds 200 characters |
| `GRP005` | 403 | Only admins can update groups |
| `GRP006` | 404 | Group not found |

### 5.2 Error Response Format
```json
{
  "errors": [
    {
      "code": "GRP003",
      "message": "Group name already exists for this user",
      "field": "name"
    }
  ],
  "meta": {
    "request_id": "uuid"
  }
}
```

---

## 6. Security Considerations

1. **Authorization:** Policy checks for all group operations
2. **Validation:** Server-side validation of all inputs
3. **Uniqueness:** Database constraints + application checks
4. **Archive Protection:** Archived groups cannot be modified
5. **Currency Lock:** Currency cannot be changed in MVP

---

## 7. Testing Strategy

### 7.1 Unit Tests
- Group creation logic
- Name uniqueness validation
- Admin assignment
- Archive functionality

### 7.2 Feature Tests
- Successful group creation
- Duplicate name rejection
- Update group (admin only)
- Archive group (admin only)
- Authorization checks

### 7.3 Integration Tests
- Database transactions
- Event dispatching
- Real-time updates

---

## 8. Events & Listeners

### 8.1 GroupCreated Event
**Location:** `app/Events/GroupCreated.php`

**Listeners:**
- Update user's group list cache
- Send real-time notification to creator

---

## 9. Dependencies

- AUTH-002: User authentication
- Database (groups, group_members tables)
- Event system for notifications

---

## 10. Related Documentation

- [GRP-002 Technical Spec](./GRP-002-technical.md) - Member Invitation
- [GRP-003 Technical Spec](./GRP-003-technical.md) - Admin Role Management
- [GRP-004 Technical Spec](./GRP-004-technical.md) - Group List View
