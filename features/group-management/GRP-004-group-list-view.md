# GRP-004: Group List View

**Feature ID:** GRP-004  
**Category:** Group Management  
**Priority:** P0 (Critical)  
**Status:** Draft  
**Last Updated:** January 2026

---

## Description
Home screen displays all user's groups in a WhatsApp-style list view.

---

## Functional Requirements
- List sorted by most recent activity (last expense/deposit)
- Each list item displays:
  - Group name
  - Main Balance (formatted with currency)
  - Last activity timestamp
  - Unread notification badge (if applicable)
- Click/tap navigates to group detail view
- Pull-to-refresh functionality
- Search/filter groups by name

---

## UI Requirements
- Mobile-first design
- Swipe actions (future: archive, leave group)
- Empty state message if no groups exist
- Loading skeleton during data fetch
- Smooth scrolling and animations

---

## Business Rules
- Only groups where user is a member are displayed
- Sorting based on most recent transaction (deposit or expense)
- Search is case-insensitive
- Real-time updates when group activity occurs

---

## Acceptance Criteria
- ✅ List loads within 2 seconds
- ✅ Groups sorted correctly by activity
- ✅ Navigation to group detail works smoothly
- ✅ Search filters groups in real-time
- ✅ Pull-to-refresh updates list
- ✅ Empty state shown when no groups
- ✅ Unread notification badges accurate
- ✅ Main Balance formatted correctly with currency

---

## Technical Notes
- Data fetching: Optimized query with indexes
- Caching: Client-side cache with invalidation
- Real-time: WebSocket updates for activity changes
- Performance: Pagination for users with many groups (future)

---

## Dependencies
- GRP-001: Group Creation
- FIN-001: Deposit Submission
- FIN-002: Expense Submission
- NOTIF-001: Real-Time Notifications
- UI-001: Mobile-Responsive Design

---

## Related Features
- GRP-001: Group Creation
- UI-001: Mobile-Responsive Design
- UI-002: Real-Time Updates

---

## Test Cases
1. **TC-GRP-004-01:** Group list loads and displays correctly
2. **TC-GRP-004-02:** Groups sorted by most recent activity
3. **TC-GRP-004-03:** Search filters groups in real-time
4. **TC-GRP-004-04:** Pull-to-refresh updates list
5. **TC-GRP-004-05:** Empty state displayed when no groups
6. **TC-GRP-004-06:** Navigation to group detail works
7. **TC-GRP-004-07:** Unread notification badges accurate
8. **TC-GRP-004-08:** Main Balance formatting correct
