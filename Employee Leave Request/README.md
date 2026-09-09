## Application Overview
The Employee Leave Request app manages employee records, leave policies, annual balances, requests, manager approval, and HR balance confirmation. It provides role-aware queues, audit history, an approved-leave calendar, and a summary dashboard for Employees, Managers, and HR.

## Forms
| Form Name | Purpose | Key Lookups |
|---|---|---|
| Employees | Employee directory and reporting lines | Manager → Employees |
| Leave Types | Leave policies and allocation defaults | None |
| Leave Balances | Annual entitlement and usage | Employee → Employees; Leave Type → Leave Types |
| Leave Requests | Request and approval lifecycle | Employee, Manager → Employees; Leave Type → Leave Types |
| Audit Log | Immutable workflow history | Leave Request → Leave Requests |

## Reports
| Report Name | Type | Source Form |
|---|---|---|
| All Employees | List | Employees |
| All Leave Types | List | Leave Types |
| All Leave Balances | List | Leave Balances |
| My Leave Balances | List | Leave Balances |
| Team Leave Balances | List | Leave Balances |
| All Leave Requests | List | Leave Requests |
| My Leave Requests | List | Leave Requests |
| Manager Review Queue | List | Leave Requests |
| HR Processing Queue | List | Leave Requests |
| Leave Request Board | Kanban | Leave Requests |
| Approved Leave Calendar | Calendar | Leave Requests |
| Leave Audit Log | List | Audit Log |

## Pages
| Page Name | Purpose | Key Components |
|---|---|---|
| Leave Dashboard | Role-aware leave overview | KPI cards, status donut, balances, upcoming leave, report links |

## Design Decisions
- Assigned managers control first-stage approval; HR explicitly confirms balance deduction.
- Balances are maintained by employee, leave type, and year.
- Requests cannot cross calendar years; duration can include or exclude weekends by leave type.
- Only completed requests appear on the approved-leave calendar.
- Blueprint stages are Submitted, Pending HR, Completed, and Rejected.
- Custom profiles are Employee, Manager, and HR; no role hierarchy is required.
