# SplitFlow: Product Concept & Business Requirements Document (BRD)

**Project Name:** SplitFlow  
**Document Version:** 1.0  
**Date:** January 2026  
**Status:** Finalized Concept

---

## 1. Executive Summary
SplitFlow is a streamlined, web-based financial coordination tool designed for groups that require a layer of oversight in shared spending. Unlike traditional "equal split" apps, SplitFlow introduces an **Admin-verified ledger system**, ensuring that group funds are managed with accountability, mimicking a digital "petty cash" box for the modern era.

---

## 2. Problem Statement
Managing group finances (trips, shared households, or project teams) currently suffers from three primary friction points:

* **The Trust & Verification Gap:** Most apps allow any user to input data that instantly affects everyone’s balance, leading to errors, duplicates, or disputes over unverified receipts.
* **Complexity Overload:** Many tools are too complex for a simple weekend trip, making the barrier to entry high.
* **Lack of Centralized Funding:** Most apps track *debt* (who owes what) but fail to track a *pre-collected pool of money* (deposits) against real-time spending.

---

## 3. The Solution (Product Concept)
SplitFlow solves these issues by combining a familiar social interface with a **Gatekeeper Workflow**.

* **Controlled Transparency:** The Admin role ensures every dollar leaving the "Main Balance" is vetted, eliminating accidental or fraudulent entries.
* **The "Deposit-First" Philosophy:** SplitFlow focuses on the **Main Balance**. By tracking deposits, the app maintains a live "Group Purse."
* **Frictionless Settlement:** At the end of an event, the system calculates the net difference between deposits and approved spending to provide a clear path to a zero balance.

---

## 4. User Roles & Permissions
The system operates on a simple binary role structure to maintain ease of use:

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

## 5. Functional Requirements

### 5.1 Group Management
* **Group Creation:** Any user can create a group and becomes the primary Admin.
* **Navigation:** A "WhatsApp-style" list view on the home screen displays all active groups.
* **Role Delegation:** Admins can promote members to Admin or revoke Admin status.
* **Admin Resignation:** An Admin can step down if at least one other Admin remains.

### 5.2 Expense & Deposit Workflow
* **Submission:** Users enter `Amount`, `Date-Time`, and an optional `Description`.
* **Approval Logic:** * Admin expenses are instantly added to the ledger.
    * Member expenses enter a "Pending" state until an Admin verifies the note/receipt.
* **Main Balance:** Each group has a unique balance. Only approved expenses and Admin deposits affect this total.
* **Individual Balance:** Users see their personal standing: `(Deposit) - (Approved Share of Expenses)`.

### 5.3 Notifications & Settlement
* **Alerts:** Admins are notified of new pending expenses immediately.
* **Real-time Re-calculation:** Upon approval, the system splits the amount among all members and updates "Owed" balances in real-time.
* **Settlement Tab:** A dedicated view showing exactly who needs to pay whom to reach a zero balance.

### 5.4 Reporting
* **Admin Controls:** Admins can generate summaries based on Weekly, Monthly, or Custom Date Ranges to track spending trends.

---

## 6. Technical & UI Requirements
* **Mobile-Responsive Web:** The UI must prioritize a "mobile-first" chat-list style.
* **Real-time Logic:** Use of WebSockets (or similar) to ensure the Main Balance updates across all user screens without page refreshes.

---

## 7. Future Roadmap ("Good-to-Have")

| Feature | Description |
| :--- | :--- |
| **Receipt OCR** | AI extraction of amount and date from photos. |
| **Unequal Splits** | Options to split by percentage or specific amount. |
| **Multi-Currency** | Real-time exchange rates for international travel. |
| **Export Data** | Downloadable PDF/CSV reports for external record-keeping. |
| **Payment Integration** | "Settle Up" buttons linked to Venmo, PayPal, or UPI. |

---

## 8. Business Context & KPIs
* **Target Audience:** Travelers, roommates, and small event organizers.
* **Value Proposition:** Accountability for Admins, transparency for Members.
* **Success Metrics:** * **Retention:** Users creating multiple groups over time.
    * **Accuracy:** Reduction in disputed entries via the Admin approval gate.