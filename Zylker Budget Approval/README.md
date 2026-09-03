# Zylker Budget Approval

## Application Overview

Zylker Budget Approval is a Zoho Creator application for Atlas Precision Manufacturing Inc. that manages spend request intake, multi-level approval routing, and leadership dashboards. Employees raise requests, which are automatically routed through a 1-, 2-, or 3-person approval chain based on cost thresholds, with full audit logging and an operations dashboard for leadership visibility.

## Forms

| Form Name | Purpose | Key Lookups |
|---|---|---|
| Spend_Request | Core form for raising spend requests; captures requester, item, cost, justification, and attachment | Looked up by Approval_Log, Approval_Action |
| Employee_Directory | Backend catalog of employees with department, masked ID, RM, and Dept Head | Referenced by Spend_Request on-user-input workflows |
| Request_Catalog | Catalog of department-scoped items with catalog costs | Referenced by Spend_Request for item dropdown and cost auto-fill |
| Approval_Log | Timestamped record of every approval decision | References Spend_Request via Request_ID_Ref text field |
| Approval_Action | Simple decision form for approvers (Approve/Hold/Reject) | Updates Spend_Request.Status via on-success workflow |

## Reports

| Report Name | Type | Source Form |
|---|---|---|
| All_Requests | List (default) | Spend_Request |
| Pending_Requests | List | Spend_Request (filtered to pending statuses) |
| All_Employees | List (default) | Employee_Directory |
| Catalog_Items | List (default) | Request_Catalog |
| All_Decisions | List (default) | Approval_Log |
| Approval_Actions | List (default) | Approval_Action |

## Pages

| Page Name | Purpose | Key Components |
|---|---|---|
| Request_Intake | Module 1: spend request intake screen | Dark nav sidebar, top header, embedded Spend_Request form via zc-component |
| Approvals | Module 2: approval queue with stage tabs and detail panel | Stage tabs (All/RM/Dept Head/Finance), queue table, stage stepper, cost breakdown, activity log, Approve/Hold/Reject action bar |
| Dashboard | Module 3: leadership analytics dashboard | KPI stat cards, SVG donut (tier split), dept bar chart, approval funnel, escalation list, turnaround badges |

## Design Decisions

- **Approval Tier is a formula field** on Spend_Request (Auto ≤ $1,000 / 2-Person ≤ $50,000 / 3-Person above), so the live indicator on the intake form updates as the user changes the Estimated Cost field.
- **Employee_Directory and Request_Catalog are invisible backend tables** — they are not surfaced in primary navigation; they power the dynamic dropdown options and auto-fill logic via on-user-input Deluge workflows.
- **Blueprint drives the approval lifecycle** — stages and transitions for all 8 states (Submitted, Auto_Approved, Pending_RM, Pending_Dept_Head, Pending_Finance, Approved, Rejected, On_Hold) are defined in the blueprint, with `zoho.currenttime` for all timestamps.
- **Dashboard data has a BRD fallback** — when the Spend_Request form has no records (fresh install), the Dashboard shows BRD Part II sample values so the screen is always presentation-ready.
- **Approvals page uses page parameters** — active_tab and selected_id URL parameters drive tab selection and detail panel population; default selected record is BRQ-2026-0303.
- **No JavaScript anywhere** — all interactivity is via Deluge `<% %>` blocks, page parameter navigation, and Creator's native form rendering.
