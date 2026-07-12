# Procurement & Purchase Approval System
### Built with Airtable | No-Code Automation 

---

## Project Overview

ABC Technologies was managing purchase requests through WhatsApp and email. Managers were missing requests, finance had no clear approval process, vendors were not tracked properly, and there was no visibility into spending.

I designed and built a complete Procurement and Purchase Approval System inside Airtable that automates the purchasing workflow — from employee request submission to purchase order generation.

---

## Business Problem

| Problem | Impact |
|---------|--------|
| Requests sent via WhatsApp and email | Managers missed requests regularly |
| No structured approval process | Finance had no visibility |
| Vendors not tracked properly | No vendor accountability |
| No spending visibility | Budget overruns went unnoticed |

---

## Solution

A fully automated no-code procurement system built entirely in Airtable with:

- 5 relational tables storing all procurement data
- An employee purchase request form
- 7 automations handling the approval workflow
- Automatic purchase order generation on full approval
- A two-page interface dashboard for tracking and submitting requests

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Airtable | Database, Forms, Automations, Interface Dashboard |
| Gmail | Automated email delivery via OAuth connection |

---

## Database Schema

The system uses 5 relational tables:

### 1. Departments

Stores department information and budget data.

| Field | Type | Description |
|-------|------|-------------|
| Department Name | Single line text | Name of department |
| Department Head | Single line text | Name of department head |
| Budget Limit | Currency | Maximum spend allowed |
| Budget Used | Rollup | Auto-calculated from Purchase Requests |
| Budget Remaining | Formula | Budget Limit minus Budget Used |

---

### 2. Employees

Stores employee details linked to departments.

| Field | Type | Description |
|-------|------|-------------|
| Employee Name | Single line text | Full name |
| Email | Email | Employee email address |
| Department | Linked record | Links to Departments table |
| Job Title | Single line text | Employee job title |
| Employee ID | Single line text | Unique employee identifier |
| Manager Name | Single line text | Name of direct manager |
| Manager Email | Email | Manager email for automation |

---

### 3. Vendors

Tracks all suppliers and vendor details.

| Field | Type | Description |
|-------|------|-------------|
| Vendor Name | Single line text | Supplier company name |
| Contact Email | Email | Vendor contact email |
| Phone Number | Phone | Vendor contact number |
| Products Supplied | Long text | What the vendor supplies |
| Vendor Status | Single select | Active or Inactive |
| Contact Person | Single line text | Name of vendor contact |
| Category | Single select | IT Equipment, Office Supplies, Software, Printing, Networking, Other |
| Preferred Vendor | Checkbox | Marks trusted vendors |

---

### 4. Purchase Requests

The heart of the system — every request lands here.

| Field | Type | Description |
|-------|------|-------------|
| Request ID | Autonumber | Auto-generated unique ID |
| Request Number | Formula | Formats as PR-1, PR-2 etc |
| Employee Name | Linked record | Links to Employees table |
| Email | Lookup | Auto-pulled from Employees |
| Manager Name | Lookup | Auto-pulled from Employees |
| Manager Email | Lookup | Auto-pulled from Employees |
| Department | Linked record | Links to Departments table |
| Item Requested | Single line text | What is being requested |
| Quantity | Number | How many units needed |
| Estimated Cost | Currency | Expected cost of purchase |
| Vendor | Linked record | Links to Vendors table |
| Business Justification | Long text | Why the item is needed |
| Request Date | Date | Request date |
| Status | Single select | Pending, Manager Approved, Fully Approved, Rejected |
| Approval Level | Formula | Auto-calculates based on cost |
| Manager Approval | Single select | Pending, Approved, Rejected |
| Finance Approval | Single select | Pending, Approved, Rejected |
| Notes | Long text | Additional comments |
| Purchase Order | Linked record | Links to Purchase Orders table |

#### Approval Level Formula

IF({Estimated Cost} < 500, "Manager Only", "Manager + Finance")

Requests with an Estimated Cost **below $500** require **Manager Only** approval.

Requests with an Estimated Cost of **$500 or above** require **Manager + Finance** approval.

---

### 5. Purchase Orders

Stores auto-generated purchase orders for fully approved requests.

| Field | Type | Description |
|-------|------|-------------|
| PO Number | Autonumber | Auto-generated unique ID |
| PO Reference | Formula | Formats as PO-1, PO-2 etc |
| Purchase Request | Linked record | Links to Purchase Requests |
| Approved By | Single line text | Manager name mapped from the request |
| Approval Date | Date | Actual automation run date |
| Total Amount | Currency | Estimated Cost mapped from the Purchase Request |
| PO Status | Single select | Issued, Sent to Vendor, Delivered, Cancelled |
| Notes | Long text | Notes mapped from the Purchase Request |

---

## Automation Workflow

The system runs 7 automations that handle the approval journey:

### Automation 1 — Send Confirmation Email to Employee

**Trigger:** When a form is submitted  
**Actions:** Send confirmation email to employee and update the new Purchase Request

The employee receives a confirmation email with the request details. The workflow initializes Status, Manager Approval and Finance Approval as Pending.

---

### Automation 2 — Send Approval Request to Manager

**Trigger:** When a form is submitted  
**Action:** Send approval request email to manager

The manager receives the request details and a prompt to log into Airtable to approve or reject.

---

### Automation 3 — Send Finance Approval Email

**Trigger:** When Manager Approval is updated  
**Condition:** Manager Approval is Approved

The automation handles the two approval paths:

- For Manager Only requests, Status is updated to Fully Approved.
- For Manager + Finance requests, Status is updated to Manager Approved and finance receives an approval request email.

---

### Automation 4 — Send Rejection Email to Employee

**Trigger:** When Manager Approval is updated  
**Condition:** Manager Approval is Rejected  
**Actions:** Send rejection notification email and update Status to Rejected

If the manager rejects the request, the employee receives a notification and the request Status is automatically changed to Rejected.

---

### Automation 5 — Generate Purchase Order

**Trigger:** When Status is updated to Fully Approved  
**Condition:** Status is Fully Approved  
**Action:** Create a new record in the Purchase Orders table

The Purchase Order is created with the linked Purchase Request, Approved By, Approval Date, Total Amount, PO Status and Notes.

- Approval Date uses Airtable's Actual run time
- Total Amount maps from Estimated Cost
- Notes maps from Purchase Request Notes
- PO Status is set to Issued

---

### Automation 6 — Manager 3-Day Reminder

**Trigger:** Every day at 8:00am WAT  
**Action:** Find records where Manager Approval is Pending and Request Date is before 3 days ago, repeat through the matching records, and send reminder emails

Prevents pending manager approvals from being forgotten.

---

### Automation 7 — Complete Finance Approval

**Trigger:** When Finance Approval is updated  
**Condition:** Finance Approval is Approved  
**Action:** Update Status to Fully Approved

When finance approves a Manager + Finance request, the Status automatically changes to Fully Approved. This then triggers Automation 5 to generate the Purchase Order.

---

## Approval Flow Diagram

Employee submits form  
↓  
Request saved in Purchase Requests  
↓  
Approval Level calculated automatically  
↓  
Confirmation email sent (Auto 1)  
Manager approval request sent (Auto 2)  
↓  
Manager reviews request  
↓  
Rejected → Status changes to Rejected and employee is notified (Auto 4)  
OR  
Approved → Approval path is checked  
↓  
Manager Only → Fully Approved → Purchase Order generated (Auto 5)  
OR  
Manager + Finance → Manager Approved → Finance email sent (Auto 3)  
↓  
Finance Approval = Approved  
↓  
Fully Approved (Auto 7)  
↓  
Purchase Order generated (Auto 5)

Every day at 8:00am WAT, pending manager approvals older than 3 days are checked for reminders (Auto 6).

---

## Interface Dashboard

The system includes a two-page Airtable Interface:

**Page 1 — Purchase Requests**

Shows submitted requests with Request ID, Employee Name, Item Requested, Estimated Cost, Status, Request Date and Department. Supports filtering, sorting and searching.

**Page 2 — Submit a Purchase Request**

A clean employee form containing Employee Name, Department, Item Requested, Quantity, Estimated Cost, Vendor, Business Justification and Request Date.

The Status field is not exposed on the employee form because approval status is controlled by the workflow.

---

## Key Features

- Automatic approval level calculation based on request cost
- Below $500 requires Manager Only approval
- $500 and above requires Manager + Finance approval
- Automated confirmation, manager, finance and rejection emails
- Automatic status updates through the approval workflow
- Auto-generated Purchase Order numbers in PO-1, PO-2 format
- Auto-generated Request Numbers in PR-1, PR-2 format
- Purchase Order Approval Date captured from Actual run time
- Estimated Cost mapped to Purchase Order Total Amount
- Purchase Request Notes transferred to Purchase Order Notes
- Daily 8am reminder for pending manager approvals older than 3 days
- Budget tracking per department with automatic rollup calculations
- Vendor management with preferred vendor flagging
- Relational database design connecting all 5 tables

---

## Challenges & Solutions

| Challenge | Solution |
|-----------|----------|
| Airtable free plan email restriction — only sends to collaborators | Switched to Gmail integration via OAuth which sends workflow emails |
| Lookup fields returning nested values in automations | Selected the correct nested Value when mapping lookup fields |
| Link direction between Purchase Orders and Purchase Requests was wrong | Identified and corrected the link direction |
| New requests needed consistent initial workflow values | Updated Automation 1 to initialize Status, Manager Approval and Finance Approval as Pending |
| Manager Only and Manager + Finance requests needed different status paths | Corrected Automation 3 to route each approval level correctly |
| Rejected requests were not updating the overall Status | Updated Automation 4 to set Status to Rejected |
| Purchase Order fields were incomplete | Updated Automation 5 to map Actual run time, Estimated Cost and Notes |
| Finance approval did not automatically complete the request | Built Automation 7 to change Status to Fully Approved after Finance Approval is Approved |

---

## What I Learned

- How to design a relational database from scratch thinking about business process first
- How lookup fields, rollup fields and formula fields work together
- How to build conditional and multi-stage approval logic in Airtable
- How to connect Gmail via OAuth for automated email delivery
- How to map dynamic values between Airtable records
- How to debug automation conditions, status conflicts and field mapping issues
- How to test an approval workflow end to end using a fresh request

---

## End-to-End Test

The final workflow was tested using a fresh request for Brian Nelson in Engineering.

- Item Requested: Engineering Monitor
- Quantity: 2
- Estimated Cost: $800
- Approval Level: Manager + Finance
- Initial Status: Pending
- Manager Approval: Pending
- Finance Approval: Pending

After Manager Approval was changed to Approved, Status automatically changed to Manager Approved.

After Finance Approval was changed to Approved, Status automatically changed to Fully Approved.

A Purchase Order was automatically generated with:

- PO Status: Issued
- Approval Date: 7/12/2026
- Total Amount: $800
- Notes: Final end-to-end workflow test

The fresh end-to-end workflow test passed successfully.

---

## Potential Improvements

- Add Slack notifications alongside email alerts for faster manager response
- Build a vendor performance rating system after each delivery
- Add budget alert automation when department spending reaches 80% of limit
- Integrate with accounting software for automatic invoice generation
- Build a more mobile-friendly interface for on-the-go request submission

---

*Built by Nnoli Godslove Chito | AI Automation
