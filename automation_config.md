# Automation Configuration — Procurement & Purchase Approval System

This document describes the configuration of all 7 automations built in Airtable for the ABC Technologies Procurement & Purchase Approval System.

---

## Automation 1 — Send Confirmation Email to Employee

**Status:** ON

**Trigger Type:** When a form is submitted

**Table:** Purchase Requests

**Form:** ABC Technologies Purchase Request Form

### Action 1 — Gmail: Send Email

**To:** Email (from Employee Name) — Value

**Subject:** Purchase Request Received — [Item Requested]

**Message:**

Dear [Employee Name],

Your purchase request has been received and is under review.

Request Details:

Item: [Item Requested]

Quantity: [Quantity]

Estimated Cost: [Estimated Cost]

Request Number: [Request Number]

We will notify you once your manager reviews it.

ABC Technologies Procurement Team

### Action 2 — Update Record

**Table:** Purchase Requests

**Record:** Triggering record

**Fields Updated:**

- Status → Pending
- Manager Approval → Pending
- Finance Approval → Pending

This ensures every new purchase request begins with the correct workflow status.

---

## Automation 2 — Send Approval Request to Manager

**Status:** ON

**Trigger Type:** When a form is submitted

**Table:** Purchase Requests

**Form:** ABC Technologies Purchase Request Form

### Action — Gmail: Send Email

**To:** Manager Email (Lookup from Employee Name) — Value

**Subject:** Action Required: Approve Purchase Request — [Item Requested]

**Message:**

Dear [Manager Name],

An employee has submitted a purchase request that requires your approval.

Request Details:

Employee: [Employee Name]

Item: [Item Requested]

Quantity: [Quantity]

Estimated Cost: [Estimated Cost]

Justification: [Business Justification]

Request Number: [Request Number]

Approval Level: [Approval Level]

Please log in to Airtable to Approve or Reject this request.

ABC Technologies Procurement Team

---

## Automation 3 — Send Finance Approval Email

**Status:** ON

**Trigger Type:** When a record is updated

**Table:** Purchase Requests

**Field Watched:** Manager Approval

**Condition:** Manager Approval is Approved

The automation checks the Approval Level of the purchase request.

### Manager Only Path

**Condition:** Approval Level is Manager Only

### Action — Update Record

**Table:** Purchase Requests

**Record:** Triggering record

**Field Updated:**

- Status → Fully Approved

Requests below $500 require Manager Only approval. After the manager approves the request, the Status automatically changes to Fully Approved.

### Manager + Finance Path

**Condition:** Approval Level is Manager + Finance

### Action 1 — Update Record

**Table:** Purchase Requests

**Record:** Triggering record

**Field Updated:**

- Status → Manager Approved

### Action 2 — Gmail: Send Email

**To:** chitogodslove68@gmail.com (Finance Team Email)

**Subject:** Finance Approval Required — [Item Requested]

**Message:**

Dear Finance Team,

A purchase request has been approved by the manager and now requires your financial approval.

Request Details:

Employee: [Employee Name]

Item: [Item Requested]

Quantity: [Quantity]

Estimated Cost: [Estimated Cost]

Request Number: [Request Number]

Manager Approved By: [Manager Name]

Please log in to Airtable to Approve or Reject this request.

ABC Technologies Procurement Team

Requests of $500 or above follow the Manager + Finance approval path.

---

## Automation 4 — Send Rejection Email to Employee

**Status:** ON

**Trigger Type:** When a record is updated

**Table:** Purchase Requests

**Field Watched:** Manager Approval

**Condition:** Manager Approval is Rejected

### Action 1 — Gmail: Send Email

**To:** Email (from Employee Name) — Value

**Subject:** Purchase Request Rejected — [Item Requested]

**Message:**

Dear [Employee Name],

We regret to inform you that your purchase request has been rejected by your manager.

Request Details:

Item: [Item Requested]

Quantity: [Quantity]

Estimated Cost: [Estimated Cost]

Request Number: [Request Number]

If you have any questions, please speak with your manager directly.

ABC Technologies Procurement Team

### Action 2 — Update Record

**Table:** Purchase Requests

**Record:** Triggering record

**Field Updated:**

- Status → Rejected

This ensures the overall Status automatically reflects the manager's rejection decision.

---

## Automation 5 — Generate Purchase Order

**Status:** ON

**Trigger Type:** When a record is updated

**Table:** Purchase Requests

**Field Watched:** Status

**Condition:** Status is Fully Approved

### Action — Create Record

**Table:** Purchase Orders

**Fields Created:**

| Field | Value |
|-------|-------|
| Purchase Request | Linked to triggering Purchase Request |
| Approved By | Manager Name (from Employee Name) — Value |
| Approval Date | Actual run time |
| Total Amount | Estimated Cost |
| PO Status | Issued |
| Notes | Notes from Purchase Request |

When a request reaches Fully Approved status, Airtable automatically creates a Purchase Order.

The PO Reference is generated automatically by the Purchase Orders formula field.

---

## Automation 6 — 3-Day Manager Reminder

**Status:** ON

**Trigger Type:** At a scheduled time

**Schedule:** Every 1 day at 8:00am WAT

### Action 1 — Find Records

**Table:** Purchase Requests

**Conditions:**

- Manager Approval is Pending
- Request Date is before 3 days ago

**Maximum Records:** 50

### Action 2 — Repeat for Each Record

The automation repeats the reminder action for every matching Purchase Request.

### Action 3 — Gmail: Send Email

**To:** chitogodslove68@gmail.com (Manager Email)

**Subject:** REMINDER: Purchase Request Awaiting Your Approval — [Item Requested]

**Message:**

Dear [Manager Name],

This is a reminder that a purchase request has been waiting for your approval for more than 3 days.

Request Details:

Employee: [Employee Name]

Item: [Item Requested]

Quantity: [Quantity]

Estimated Cost: [Estimated Cost]

Request Number: [Request Number]

Request Date: [Request Date]

Please log in to Airtable to Approve or Reject this request as soon as possible.

ABC Technologies Procurement Team

---

## Automation 7 — Complete Finance Approval

**Status:** ON

**Trigger Type:** When a record is updated

**Table:** Purchase Requests

**Field Watched:** Finance Approval

**Condition:** Finance Approval is Approved

### Action — Update Record

**Table:** Purchase Requests

**Record:** Triggering record

**Field Updated:**

- Status → Fully Approved

When Finance Approval changes to Approved, the Purchase Request Status automatically changes to Fully Approved.

The Fully Approved status then triggers Automation 5, which automatically generates the Purchase Order.

---

## Approval Level Logic

The Approval Level field automatically determines the required approval route based on Estimated Cost.

**Formula:**

```text
IF(
  {Estimated Cost} < 500,
  "Manager Only",
  "Manager + Finance"
)
```

### Approval Rules

- Requests below $500 require Manager Only approval.
- Requests of $500 or above require Manager + Finance approval.

### Examples

- $499 → Manager Only
- $500 → Manager + Finance
- $800 → Manager + Finance

---

## Verified End-to-End Workflow Test

A fresh end-to-end workflow test was completed using the following request:

- Employee: Brian Nelson
- Department: Engineering
- Item Requested: Engineering Monitor
- Quantity: 2
- Estimated Cost: $800
- Approval Level: Manager + Finance
- Initial Status: Pending
- Manager Approval: Pending
- Finance Approval: Pending

After Manager Approval was changed to Approved, the request Status automatically changed to Manager Approved.

After Finance Approval was changed to Approved, the request Status automatically changed to Fully Approved.

A Purchase Order was automatically generated with:

- PO Status: Issued
- Approval Date: 7/12/2026
- Total Amount: $800
- Notes: Final end-to-end workflow test

The fresh end-to-end workflow test passed successfully.

---

*Built by Nnoli Godslove Chito | AI Automation*
