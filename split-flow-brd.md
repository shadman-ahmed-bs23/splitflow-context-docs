# Business Requirements Document (BRD): SplitFlow

**Project Name:** SplitFlow  
**Version:** 1.0  
**Status:** Draft  

---

## 1. Project Overview
SplitFlow is a web-based group expense management application designed to simplify shared finances. It features a hierarchical approval system where Admins oversee the ledger, ensuring that only verified expenses impact the group's "Main Balance." The UI is modeled after modern messaging apps for ease of navigation.

## 2. User Roles & Permissions
The system utilizes a simple binary role structure:

| Feature | Admin | Member |
| :--- | :--- | :--- |
| Create Group | Yes | Yes |
| Add/Remove Admins | Yes | No |
| Submit Expense | Auto-approved | Requires Approval |
| Deposit Funds | Yes | No |
| View Group List | Yes | Yes |
| Generate Reports | Yes | No |
| View Settlement | Yes | Yes |

---

## 3. Functional Requirements

### 3.1 Group Management
* **Group Creation:** Any registered user can create a group. The creator is automatically assigned the **Admin** role.
* **Navigation:** A "WhatsApp-like" list view on the home screen displays all groups a user has joined.
* **Role Delegation:** Admins can promote any member to Admin status or demote existing Admins.
* **Admin Resignation:** An Admin can leave the Admin role or the group entirely, provided at least one other Admin remains to manage the group.

### 3.2 Expense & Deposit Workflow
* **Expense Submission:** Users provide `Amount`, `Date-Time`, and an optional `Description`.
* **The Approval Logic:**
    * **Admin Submissions:** Automatically approved and reflected in the balance.
    * **Member Submissions:** Remain "Pending" until an Admin clicks **Approve**.
* **Main Balance:** Each group maintains its own pool of funds. Only approved expenses and Admin-initiated deposits impact this balance.
* **Individual Balance Tracking:** Users can view their personal standing, calculated as:  
  `Individual Balance = (Approved Deposits) - (User's Share of Approved Expenses)`

### 3.3 Notifications & Real-time Recalculation
* **Notifications:** Admins receive an immediate alert/notification when a new expense is pending.
* **Calculation:** Upon approval, the system divides the expense amount by the total number of group members.
* **Settlement:** A "View Settlement" tab calculates the most efficient way to reach a zero balance (e.g., "User A owes User B $50").

### 3.4 Reporting
* **Admin Controls:** Admins can generate financial summaries based on:
    * Weekly cycles
    * Monthly cycles
    * Custom date ranges

---

## 4. Proposed "Good-to-Have" Features (Phase 2)

* **Receipt Upload (OCR):** Allow users to upload photos of receipts to auto-fill expense details.
* **Unequal Splits:** Option to split expenses by percentage or fixed amount rather than just equal shares.
* **Multi-Currency Support:** Automatic currency conversion for international group trips.
* **Export Data:** Ability for Admins to download reports in PDF or CSV format.
* **Categories:** Tagging expenses (e.g., Food, Travel, Rent) for better visual breakdown in reports.

---

## 5. Technical Considerations
* **Responsive Web Design:** The application must be fully functional on mobile and desktop browsers.
* **Real-time Updates:** Use of WebSockets (or similar) to ensure the "Main Balance" updates across all user screens without page refreshes.