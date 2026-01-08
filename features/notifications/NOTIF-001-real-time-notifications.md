# NOTIF-001: Real-Time Notifications

**Feature ID:** NOTIF-001  
**Category:** Notifications & Communication  
**Priority:** P0 (Critical)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Users receive instant notifications for relevant group activities.

---

## Functional Requirements
- Notification types:
  - New pending expense (Admins only)
  - Expense approved/rejected (submitter)
  - New deposit (all members)
  - New member joined (all members)
  - Balance update (all members)
  - Admin role change (affected user)
- Delivery methods:
  - In-app notifications (real-time)
  - Email notifications (configurable per user)
  - Browser push notifications (opt-in)
- Notification center with:
  - Unread count badge
  - List of recent notifications
  - Mark as read functionality
  - Filter by notification type

---

## Technical Requirements
- WebSocket connection for real-time delivery
- Fallback to polling if WebSocket unavailable
- Notification persistence in database
- Notification expiration after 30 days

---

## Business Rules
- Notifications are user-specific
- Email notifications respect user preferences
- Push notifications require user opt-in
- Unread count updates in real-time
- Notifications expire after 30 days

---

## Acceptance Criteria
- ✅ Notifications delivered within 1 second
- ✅ Unread count accurate
- ✅ Email notifications sent within 60 seconds
- ✅ Notification center loads within 1 second
- ✅ Mark as read works correctly
- ✅ Filtering works as expected
- ✅ WebSocket fallback to polling works

---

## Technical Notes
- WebSocket: Socket.io or native WebSocket
- Fallback: Long polling or Server-Sent Events
- Database: Notifications table with indexes
- Email: Async job queue for email delivery
- Push: Service Worker for browser push

---

## Dependencies
- All feature modules (for notification triggers)
- Email service integration
- WebSocket server
- Database notifications table

---

## Related Features
- All features (notifications triggered by various actions)
- UI-002: Real-Time Updates

---

## Test Cases
1. **TC-NOTIF-001-01:** Real-time notification delivery
2. **TC-NOTIF-001-02:** Unread count accuracy
3. **TC-NOTIF-001-03:** Email notification delivery
4. **TC-NOTIF-001-04:** Notification center functionality
5. **TC-NOTIF-001-05:** Mark as read works
6. **TC-NOTIF-001-06:** Filtering by type works
7. **TC-NOTIF-001-07:** WebSocket fallback to polling
8. **TC-NOTIF-001-08:** Notification expiration after 30 days
