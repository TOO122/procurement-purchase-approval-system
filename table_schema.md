# Table Schema — Procurement & Purchase Approval System

This document describes the complete database schema for the ABC Technologies Procurement & Purchase Approval System built in Airtable.

---

## Overview

The system uses 5 relational tables that are linked together to create a complete procurement database:

Departments → Employees → Purchase Requests → Vendors

Purchase Requests → Purchase Orders

---

## Table 1 — Departments

**Purpose:** Stores department information and tracks budget usage automatically.

| Field Name | Field Type | Description |
|------------|------------|-------------|
| Department Name | Single line text | Primary field — name of department |
| Department Head | Single line text | Name of person heading the department |
| Budget Limit | Currency | Maximum amount department can spend |
| Employees | Linked record | Auto-populated when employees link to this table |
| Purchase Requests | Linked record | Auto-populated when requests link to this table |
| Budget Used | Rollup | SUM of Estimated Cost from linked Purchase Requests |
| Budget Remaining | Formula | Budget Limit minus Budget Used |

### Budget Remaining Formula

`{Budget Limit} - {Budget Used}`

### Sample Data

| Department Name | Department Head | Budget Limit |
|-----------------|-----------------|--------------|
| Engineering | Sarah Johnson | $50,000 |
| Finance | Michael Chen | $30,000 |
| Human Resources | Amara Osei | $20,000 |
| Marketing | David Okafor | $25,000 |
| Operations | Ifenkili Joseph | $35,000 |

---

## Table 2 — Employees

**Purpose:** Stores employee details and links each employee to their department.

| Field Name | Field Type | Description |
|------------|------------|-------------|
| Employee Name | Single line text | Primary field — full name |
| Email | Email | Employee email address |
| Department | Linked record | Links to Departments table |
| Job Title | Single line text | Employee's role |
| Employee ID | Single line text | Unique employee identifier, e.g. EMP001 |
| Manager Name | Single line text | Name of direct manager |
| Manager Email | Email | Manager email used in automations |

### Sample Data

| Employee Name | Department | Job Title | Employee ID |
|---------------|------------|-----------|-------------|
| James Oti | Marketing | Marketing Lead | EMP001 |
| Adaugo Okeke | Finance | Financial Analyst | EMP002 |
| Brian Nelson | Engineering | Software Engineer | EMP003 |
| Ngozi Ifeji | Human Resources | HR Officer | EMP004 |
| Elizabeth Daniel | Operations | Operations Manager | EMP005 |

---

## Table 3 — Vendors

**Purpose:** Tracks all suppliers ABC Technologies works with.

| Field Name | Field Type | Description |
|------------|------------|-------------|
| Vendor Name | Single line text | Primary field — supplier name |
| Contact Email | Email | Vendor contact email |
| Phone Number | Phone | Vendor contact number |
| Products Supplied | Long text | Description of what vendor supplies |
| Vendor Status | Single select | Active or Inactive |
| Contact Person | Single line text | Name of vendor representative |
| Category | Single select | Vendor product category |
| Preferred Vendor | Checkbox | Checked if vendor is a preferred supplier |

### Single Select Options — Vendor Status

- Active
- Inactive

### Single Select Options — Category

- IT Equipment
- Office Supplies
- Software
- Printing
- Networking
- Other

---

## Table 4 — Purchase Requests

**Purpose:** The heart of the system. Every employee request is stored here and drives the approval workflow.

| Field Name | Field Type | Description |
|------------|------------|-------------|
| Request ID | Autonumber | Auto-generated unique number |
| Request Number | Formula | Formats the request number as PR-1, PR-2, etc. |
| Employee Name | Linked record | Links to Employees table |
| Email | Lookup | Pulled from Employee Name → Email |
| Manager Name | Lookup | Pulled from Employee Name → Manager Name |
| Manager Email | Lookup | Pulled from Employee Name → Manager Email |
| Department | Linked record | Links to Departments table |
| Item Requested | Single line text | What the employee needs |
| Quantity | Number | Number of units requested |
| Estimated Cost | Currency | Expected purchase cost |
| Vendor | Linked record | Links to Vendors table |
| Business Justification | Long text | Why the item is needed |
| Request Date | Date | Date of submission |
| Status | Single select | Overall request status |
| Approval Level | Formula | Automatically determines the required approval route |
| Manager Approval | Single select | Manager decision |
| Finance Approval | Single select | Finance decision |
| Notes | Long text | Comments from approvers |
| Purchase Order | Linked record | Links to Purchase Orders table |

### Approval Level Formula

`IF({Estimated Cost} < 500, "Manager Only", "Manager + Finance")`

### Approval Rules

- Requests below $500 require Manager Only approval.
- Requests of $500 or above require Manager + Finance approval.

### Examples

- $499 → Manager Only
- $500 → Manager + Finance
- $800 → Manager + Finance

### Request Number Formula

`"PR-" & {Request ID}`

### Single Select Options — Status

- Pending
- Manager Approved
- Finance Approved
- Fully Approved
- Rejected

### Single Select Options — Manager Approval

- Pending
- Approved
- Rejected

### Single Select Options — Finance Approval

- Pending
- Approved
- Rejected

---

## Table 5 — Purchase Orders

**Purpose:** Stores purchase orders automatically generated for fully approved requests.

| Field Name | Field Type | Description |
|------------|------------|-------------|
| PO Number | Autonumber | Auto-generated unique number |
| PO Reference | Formula | Formats the PO reference as PO-1, PO-2, etc. |
| Purchase Request | Linked record | Links back to Purchase Requests table |
| Approved By | Single line text | Name of final approver |
| Approval Date | Date | Date of final approval |
| Total Amount | Currency | Final confirmed purchase amount |
| PO Status | Single select | Current purchase order status |
| Notes | Long text | Additional PO comments |

### PO Reference Formula

`"PO-" & {PO Number}`

### Single Select Options — PO Status

- Issued
- Sent to Vendor
- Delivered
- Cancelled

---

## Table Relationships

| From Table | Link Field | To Table | Relationship |
|------------|------------|----------|--------------|
| Employees | Department | Departments | Many employees → One department |
| Purchase Requests | Employee Name | Employees | Many requests → One employee |
| Purchase Requests | Department | Departments | Many requests → One department |
| Purchase Requests | Vendor | Vendors | Many requests → One vendor |
| Purchase Orders | Purchase Request | Purchase Requests | One PO → One request |

---

*Built by Nnoli Godslove Chito | AI Automation*
