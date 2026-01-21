# GRP-002: Member Invitation - Technical Specification

**Feature ID:** GRP-002  
**Category:** Group Management  
**Last Updated:** January 2026  
**Status:** Draft

---

## 1. API Endpoints

### 1.1 Send Invitation
**Endpoint:** `POST /api/v1/groups/{groupId}/invitations`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Request Body:**
```json
{
  "email": "member@example.com"
}
```

**Response (201 Created):**
```json
{
  "data": {
    "id": "uuid",
    "group_id": "uuid",
    "email": "member@example.com",
    "invitation_token": "secure-token",
    "expires_at": "2026-01-22T10:30:00Z",
    "status": "pending",
    "created_at": "2026-01-15T10:30:00Z"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Error Responses:**
- `403 Forbidden` - Not an admin
- `404 Not Found` - Group not found or user not registered
- `409 Conflict` - User already a member
- `422 Unprocessable Entity` - Validation errors
- `429 Too Many Requests` - Member limit reached (50)

---

### 1.2 Accept Invitation
**Endpoint:** `POST /api/v1/invitations/{invitationToken}/accept`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": {
    "group": {
      "id": "uuid",
      "name": "Weekend Trip"
    },
    "member": {
      "id": "uuid",
      "group_id": "uuid",
      "user_id": "uuid",
      "role": "member",
      "joined_at": "2026-01-15T10:35:00Z"
    }
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

**Error Responses:**
- `404 Not Found` - Invitation not found or expired
- `409 Conflict` - Already a member
- `410 Gone` - Invitation already used

---

### 1.3 Decline Invitation
**Endpoint:** `POST /api/v1/invitations/{invitationToken}/decline`

**Headers:**
```
Authorization: Bearer {access_token}
```

**Response (200 OK):**
```json
{
  "data": {
    "message": "Invitation declined"
  },
  "meta": {
    "request_id": "uuid"
  }
}
```

---

## 2. Database Schema

### 2.1 Group Invitations Table
```sql
CREATE TABLE group_invitations (
    id CHAR(36) PRIMARY KEY,
    group_id CHAR(36) NOT NULL,
    email VARCHAR(255) NOT NULL,
    invitation_token VARCHAR(64) UNIQUE NOT NULL,
    invited_by CHAR(36) NOT NULL,
    status ENUM('pending', 'accepted', 'declined', 'expired') DEFAULT 'pending' NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    accepted_at TIMESTAMP NULL,
    declined_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_group_id (group_id),
    INDEX idx_email (email),
    INDEX idx_token (invitation_token),
    INDEX idx_expires_at (expires_at),
    FOREIGN KEY (group_id) REFERENCES groups(id) ON DELETE CASCADE,
    FOREIGN KEY (invited_by) REFERENCES users(id) ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## 3. Business Logic Implementation

### 3.1 Service: InvitationService
**Location:** `app/Services/Group/InvitationService.php`

**Methods:**
- `sendInvitation(Group $group, User $admin, string $email): GroupInvitation`
- `acceptInvitation(string $token, User $user): GroupMember`
- `declineInvitation(string $token, User $user): void`

**Key Logic:**
1. Verify admin status
2. Check user exists and is registered
3. Check not already a member
4. Check member limit (50 max)
5. Generate secure invitation token
6. Set expiration (7 days)
7. Send invitation email
8. On accept: Create group_member, mark invitation accepted
9. Dispatch MemberJoined event

---

### 3.2 Action: SendInvitationAction
**Location:** `app/Actions/Group/SendInvitationAction.php`

**Responsibilities:**
- Validate admin authorization
- Check user registration
- Check membership status
- Check member limit
- Generate token
- Create invitation record
- Queue email job
- Return invitation resource

---

### 3.3 Action: AcceptInvitationAction
**Location:** `app/Actions/Group/AcceptInvitationAction.php`

**Responsibilities:**
- Validate token
- Check expiration
- Check not already used
- Verify user matches invitation email
- Create group_member record
- Mark invitation as accepted
- Dispatch MemberJoined event
- Return member resource

---

## 4. Technical Implementation Details

### 4.1 Invitation Token Generation
```php
use Illuminate\Support\Str;

function generateInvitationToken(): string
{
    return Str::random(64);
}
```

### 4.2 Invitation Link Format
```
https://app.splitflow.com/invitations/{invitationToken}
```

### 4.3 Member Limit Check
```php
$currentMemberCount = GroupMember::where('group_id', $group->id)
    ->where('is_active', true)
    ->count();

if ($currentMemberCount >= 50) {
    throw new MemberLimitExceededException();
}
```

### 4.4 Expiration Cleanup
**Scheduled Job:** `app/Console/Commands/ExpireInvitations.php`

Runs daily to mark expired invitations.

---

## 5. Error Handling

### 5.1 Error Codes
| Code | HTTP Status | Message |
|------|-------------|---------|
| `INV001` | 403 | Only admins can send invitations |
| `INV002` | 404 | User not found or not registered |
| `INV003` | 409 | User already a member |
| `INV004` | 429 | Group member limit reached (50) |
| `INV005` | 404 | Invitation not found |
| `INV006` | 410 | Invitation expired |
| `INV007` | 410 | Invitation already used |

---

## 6. Security Considerations

1. **Token Security:** Cryptographically secure random tokens
2. **Single Use:** Tokens invalidated after acceptance
3. **Expiration:** Hard 7-day limit
4. **Authorization:** Admin-only invitation sending
5. **Email Verification:** User must match invitation email

---

## 7. Email Template

**Subject:** "You've been invited to join {group_name}"

**Body Variables:**
- `{{ group_name }}`
- `{{ inviter_name }}`
- `{{ invitation_link }}`
- `{{ expires_in }}` - Days until expiration

---

## 8. Testing Strategy

### 8.1 Unit Tests
- Token generation uniqueness
- Expiration checking
- Member limit validation

### 8.2 Feature Tests
- Successful invitation flow
- Expired invitation rejection
- Already member rejection
- Member limit enforcement
- Single-use token enforcement

### 8.3 Integration Tests
- Email delivery
- Event dispatching
- Database transactions

---

## 9. Events & Listeners

### 9.1 MemberJoined Event
**Location:** `app/Events/MemberJoined.php`

**Listeners:**
- Update unread counts for all group members
- Send real-time notifications

---

## 10. Dependencies

- GRP-001: Group Creation
- AUTH-001: User Registration
- Email service
- NOTIF-001: Real-Time Notifications

---

## 11. Related Documentation

- [GRP-001 Technical Spec](./GRP-001-technical.md) - Group Creation
- [GRP-003 Technical Spec](./GRP-003-technical.md) - Admin Role Management
- [NOTIF-001 Technical Spec](../notifications/NOTIF-001-technical.md) - Notifications
