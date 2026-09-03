Zylker Budget Approval — a Zoho Creator app for Atlas Precision Manufacturing Inc. with exactly three user-facing modules: (1) Request Intake Form for employees to raise spend requests, (2) Approvals for RM/Dept Head/Finance to review and decide, (3) Dashboard for leadership visibility. Supporting forms (Employee Directory, Request Catalog, Approval Log) are invisible backend tables that power these three modules.

## User_Profiles

Sets up the five background permission profiles — Requester, Reporting Manager, Department Head, Finance Controller, and Operations Leadership. These are invisible to end users but control who can access each module and what data they can see.

Write shell profile .ds files (name + type only, NO ModulePermissions yet).
ModulePermissions are added in the Permissions segment after all forms/reports exist.

1. Permissions/Profiles/Requester.ds
   name = Requester | type = Users_Permissions
   permissions = { Chat:false, Predefined:false, ApiAccess:false, PIIAccess:false, ePHIAccess:false }
   description = "Raises spend requests for their department"

2. Permissions/Profiles/Reporting_Manager.ds
   name = Reporting_Manager | type = Users_Permissions
   permissions = { Chat:false, Predefined:false, ApiAccess:false, PIIAccess:false, ePHIAccess:false }
   description = "First-level approver on 2-Person and 3-Person tier requests"

3. Permissions/Profiles/Department_Head.ds
   name = Department_Head | type = Users_Permissions
   permissions = { Chat:false, Predefined:false, ApiAccess:false, PIIAccess:false, ePHIAccess:false }
   description = "Second-level approver on 3-Person tier requests only"

4. Permissions/Profiles/Finance_Controller.ds
   name = Finance_Controller | type = Users_Permissions
   permissions = { Chat:false, Predefined:false, ApiAccess:false, PIIAccess:false, ePHIAccess:false }
   description = "Final approver on all non-auto tier requests"

5. Permissions/Profiles/Operations_Leadership.ds
   name = Operations_Leadership | type = Users_Permissions
   permissions = { Chat:false, Predefined:false, ApiAccess:false, PIIAccess:false, ePHIAccess:false }
   description = "Views consolidated spend activity and dashboard"

**Implementation notes:** Shell only. No ModulePermissions. Keep concise.

## Data_Backend

Creates the three invisible backend tables that power the BRD modules: an Employee Directory (for auto-resolving Requester ID and manager names), a Request Catalog (for department-filtered item dropdowns with auto-populated costs), and an Approval Log (for storing every decision as a timestamped activity entry). None appear in the end-user navigation.

ReadDoc BEFORE writing: forms/index, forms/forms, forms/fields, forms/fields/section_rules, forms/fields/choice, forms/fields/currency_codes, reports/index

--- FORM 1: Employee Directory ---
FILE: Components/Employee_Directory/Forms/Employee_Directory.ds

form Employee_Directory
{
  displayname = "Employee Directory"

  Section_1 (type=section, row=1, column=0)
  Employee_Name (type=text, must have, displayname="Employee Name", row=1)
  Department (type=picklist, must have, displayname="Department", row=1, column=2,
    values={"Production","Maintenance & Utilities","Quality Assurance","Procurement & Stores","IT & Systems","HR & Admin"})
  Requester_ID (type=text, displayname="Requester ID", row=1)
  Reporting_Manager (type=text, displayname="Reporting Manager", row=1)
  Department_Head (type=text, displayname="Department Head", row=1)
}

Sample records (6 rows from BRD Part II §12):
  Carlos Medina | Production | EMP-•••-3011 | Jason Miller | Henrik Larsson
  Wei Zhang | Maintenance & Utilities | EMP-•••-3024 | Fatima Al-Sayed | Aisha Bello
  Amara Okafor | Quality Assurance | EMP-•••-3037 | Marco Rossi | Takeshi Yamamoto
  Elena Petrova | Procurement & Stores | EMP-•••-3042 | Grace Kim | Isabella Fischer
  Daniel Cohen | IT & Systems | EMP-•••-3055 | Liam O'Connor | Noah Bergström
  Priya Nair | HR & Admin | EMP-•••-3068 | Sofia Martins | Chidi Eze

FILE: Components/Employee_Directory/Reports/All_Employees.ds
default list All_Employees {
  displayName = "All Employees"
  show all rows from Employee_Directory
  (Employee_Name, Department, Requester_ID, Reporting_Manager, Department_Head)
  sort by (Employee_Name ascending)
}

--- FORM 2: Request Catalog ---
FILE: Components/Request_Catalog/Forms/Request_Catalog.ds

form Request_Catalog
{
  displayname = "Request Catalog"

  Section_1 (type=section, row=1, column=0)
  Department (type=picklist, must have, displayname="Department", row=1,
    values={"Production","Maintenance & Utilities","Quality Assurance","Procurement & Stores","IT & Systems","HR & Admin"})
  Item_Description (type=text, must have, displayname="Item / Description", row=1, column=2)
  Catalog_Cost (type=USD, must have, displayname="Catalog Cost (USD)", row=1)
}

Sample catalog (18 items from BRD Part II §13):
Production: PPE restock — gloves & face shields ($850) | Conveyor sensor upgrade — Line 4 ($18,500) | CNC lathe machine — Unit 2 ($128,000)
Maintenance & Utilities: Bearing & lubricant spares ($620) | Industrial vacuum pump ($32,000) | Air compressor replacement (plant-wide) ($94,000)
Quality Assurance: Calibration gauge set ($780) | Digital vernier & micrometer set ($9,500) | Automated inspection system ($142,000)
Procurement & Stores: Packaging consumables restock ($340) | Pallet racking addition ($22,000) | Warehouse racking automation project ($86,000)
IT & Systems: Keyboard/mouse replacement batch 20 units ($560) | Departmental laptop refresh 10 units ($48,000) | Server & network infrastructure upgrade ($115,000)
HR & Admin: Stationery & printing restock ($210) | HRMS software license renewal ($11,000) | Cafeteria & facility renovation ($69,000)

FILE: Components/Request_Catalog/Reports/Catalog_Items.ds
default list Catalog_Items {
  displayName = "Catalog Items"
  show all rows from Request_Catalog
  (Department, Item_Description, Catalog_Cost)
  filters (Department)
  sort by (Department ascending)
}

--- FORM 3: Approval Log ---
FILE: Components/Approval_Log/Forms/Approval_Log.ds

form Approval_Log
{
  displayname = "Approval Log"

  Section_1 (type=section, row=1, column=0)
  Request_ID_Ref (type=text, must have, displayname="Request ID", row=1)
  Stage (type=picklist, must have, displayname="Stage", row=1, column=2,
    values={"System","Reporting Manager","Department Head","Finance Controller"})
  Approver_Name (type=text, must have, displayname="Approver Name", row=1)
  Decision (type=picklist, must have, displayname="Decision", row=1,
    values={"Submitted","Auto-Approved","Approved","On Hold","Rejected"})
  Decision_Timestamp (type=datetime, must have, displayname="Decision Date & Time", row=1)
  Comments (type=textarea, displayname="Comments", row=1, height=60px)
}

FILE: Components/Approval_Log/Reports/All_Decisions.ds
default list All_Decisions {
  displayName = "All Decisions"
  show all rows from Approval_Log
  (Request_ID_Ref, Stage, Approver_Name, Decision, Decision_Timestamp, Comments)
  sort by (Decision_Timestamp descending)
}

**Implementation notes:** These forms are hidden from end-user navigation. They are referenced by the main Spend_Request form's workflows and by the Approvals/Dashboard page Deluge scripts.

## Spend_Request

Module 1 — the core Spend Request data form that employees submit. Captures all BRD fields: requester identity (auto-resolved from the Employee Directory), department, item/description (filtered by department from the catalog), estimated cost with live Approval Tier auto-calculation, priority, justification, and an optional attachment. A blueprint drives the full lifecycle from Submitted through each approval stage to Approved or Rejected.

ReadDoc BEFORE writing: forms/index, forms/forms, forms/fields, forms/fields/section_rules, forms/fields/choice, forms/fields/lookup, forms/fields/formula_autonumber, forms/fields/media, forms/fields/subform, forms/fields/currency_codes, workflows/formWorkflows, workflows/blueprint, workflows/overview, workflows/tasks

---
FILE: Components/Spend_Request/Forms/Spend_Request.ds

form Spend_Request
{
  displayname = "Spend Request"
  success message = "Your spend request has been submitted. You will be notified as it moves through the approval chain."

  blueprint components
  {
    stages = {"Submitted","Auto_Approved","Pending_RM","Pending_Dept_Head","Pending_Finance","Approved","Rejected","On_Hold"}
  }

  FIELDS (ReadDoc fields/section_rules first — first section must be row=1, column=0; all subsequent sections need visibility=true):

  Section_Requester_Details (type=section, row=1, column=0, displayname="Requester Details")
  Requester_Name (type=text, must have, displayname="Requester Name", row=1)
  Department (type=picklist, must have, displayname="Department", row=1, column=2,
    values={"Production","Maintenance & Utilities","Quality Assurance","Procurement & Stores","IT & Systems","HR & Admin"})
  Requester_ID (type=text, displayname="Requester ID", row=1
    — NOTE: do NOT mark private; must have is also NOT used here since it is auto-resolved. This field is auto-populated via on-user-input workflow.)
  Reporting_Manager_Name (type=text, displayname="Reporting Manager", row=1)
  Department_Head_Name (type=text, displayname="Department Head", row=1)

  Section_Request_Details (type=section, row=2, column=1, visibility=true, displayname="Request Details")
  Item_Description (type=text, must have, displayname="Item / Description", row=2
    — rendered as a dropdown in UI via on-user-input of Department that triggers a Deluge setOptions; declared as text since lookup picklist to Request_Catalog filters dynamically)
  Estimated_Cost (type=USD, must have, displayname="Estimated Cost (USD)", row=2, column=2)
  Approval_Tier (type=formula, displayname="Approval Tier", row=2,
    value = if(Estimated_Cost <= 1000, "Auto", if(Estimated_Cost <= 50000, "2-Person", "3-Person")),
    visibility=true)
  Priority (type=picklist, must have, displayname="Priority", row=2,
    values={"Low","Medium","High"}, initial value="Medium")
  Justification (type=textarea, displayname="Justification", row=2, height=80px)
  Attachment (type=upload file, displayname="Attachment (Vendor Quotation)", row=2, file count=1, browse=local_drive)

  Section_Status_Info (type=section, row=3, column=1, visibility=true, displayname="Status & Tracking")
  Request_ID (type=text, displayname="Request ID", row=3
    — auto-set on success to format BRQ-YYYY-NNNN using record ID; read-only in practice)
  Status (type=picklist, displayname="Status", row=3, column=2,
    values={"Submitted","Auto_Approved","Pending_RM","Pending_Dept_Head","Pending_Finance","Approved","Rejected","On_Hold"},
    initial value="Submitted")
  SLA_Due_Date (type=date, displayname="SLA Due Date", row=3)
  Submitted_Date (type=date, displayname="Submitted Date", row=3)

  Section_Cost_Breakdown (type=section, row=4, column=1, visibility=true, displayname="Cost Breakdown")
  Cost_Breakdown (type=grid, displayname="Cost Breakdown", row=4,
    values=Cost_Breakdown_Items.ID — this is a subform; ReadDoc fields/subform for correct syntax)
    Sub-fields of Cost_Breakdown: Line_Item (type=text, displayname="Line Item"), Amount (type=USD, displayname="Amount")

  actions
  {
    on add
    {
      Save_Draft (type=reset, displayname="Save Draft")
      Submit_Request (type=submit, displayname="Submit Request")
    }
    on edit
    {
      Update (type=submit, displayname="Update")
      Cancel (type=cancel, displayname="Cancel")
    }
  }
}

NOTE on subform: The Cost_Breakdown grid sub-fields (Line_Item, Amount) belong to a sub-form entity. ReadDoc fields/subform before writing this block to get correct syntax (values = <SubFormName>.ID and sub-field declarations inside).

---
FILE: Components/Spend_Request/FormWorkflows/Spend_Request_On_Success.ds

On-success (record event = on add) workflow:
- Sets Request_ID = "BRQ-" + zoho.currentyear + "-" + zoho.toString(input.ID) padded to 4 digits
- Sets Submitted_Date = zoho.currentdate
- Based on formula Approval_Tier value:
  * "Auto" → Status = "Auto_Approved", create Approval_Log entry (Decision="Auto-Approved", Approver_Name="System", Stage="System")
  * "2-Person" → Status = "Pending_RM", SLA_Due_Date = zoho.currentdate.addDay(60)
  * "3-Person" → Status = "Pending_RM", SLA_Due_Date = zoho.currentdate.addDay(60)
- For non-auto requests: create Approval_Log entry (Decision="Submitted", Approver_Name=input.Requester_Name, Stage="System", Decision_Timestamp=zoho.currentdatetime)
- Update the just-created record with these computed values using update record Deluge

---
FILE: Components/Spend_Request/FormWorkflows/Requester_Name_On_User_Input.ds

Record event = on add or edit
Sub-event: on user input of Requester_Name
Actions (custom deluge script):
- Fetch Employee_Directory record where Employee_Name == input.Requester_Name && Department == input.Department
- If found: set Requester_ID = emp.Requester_ID, set Reporting_Manager_Name = emp.Reporting_Manager, set Department_Head_Name = emp.Department_Head
- Use Deluge UI functions: input.Requester_ID = emp.Requester_ID; input.Reporting_Manager_Name = emp.Reporting_Manager; input.Department_Head_Name = emp.Department_Head

---
FILE: Components/Spend_Request/FormWorkflows/Department_On_User_Input.ds

Record event = on add or edit
Sub-event: on user input of Department
Actions (custom deluge script):
- Clear Item_Description: input.Item_Description = "";
- Clear Estimated_Cost: input.Estimated_Cost = 0;
- Fetch Employee_Directory[Employee_Name == input.Requester_Name && Department == input.Department]
- If found: auto-fill Requester_ID, Reporting_Manager_Name, Department_Head_Name
- Fetch Request_Catalog items for this department:
  catalog_items = Request_Catalog[Department == input.Department];
  Use setOptions(Item_Description, <list of item descriptions from catalog>)

---
FILE: Components/Spend_Request/FormWorkflows/Item_Description_On_User_Input.ds

Record event = on add or edit
Sub-event: on user input of Item_Description
Actions (custom deluge script):
- Fetch Request_Catalog[Department == input.Department && Item_Description == input.Item_Description]
- If found: input.Estimated_Cost = catalog_record.Catalog_Cost;

---
FILE: Components/Spend_Request/FormWorkflows/BluePrints/Spend_Request_Blueprint.ds

Blueprint for Spend_Request full lifecycle:
type = blueprint
form = Spend_Request
start stage = "Submitted"

Stages (must match form blueprint components exactly):
  Submitted, Auto_Approved, Pending_RM, Pending_Dept_Head, Pending_Finance, Approved, Rejected, On_Hold

Transitions (every transition needs before{} and after{} — both mandatory even if empty):

1. Auto_Approve: Submitted → Auto_Approved
   type=normal | owner: Administrator profile
   after: update Status="Auto_Approved"

2. Route_To_RM_2Person: Submitted → Pending_RM
   type=normal | transition criteria: Approval_Tier == "2-Person" OR Approval_Tier == "3-Person"
   after: update Status="Pending_RM"

3. RM_Approve_2Person: Pending_RM → Pending_Finance
   type=normal | transition criteria: Approval_Tier == "2-Person"
   owner: Reporting_Manager profile
   during: update_fields visible fields=[Comments]
   after: custom deluge — update Status="Pending_Finance"; add Approval_Log entry (Stage="Reporting Manager", Decision="Approved")

4. RM_Approve_3Person: Pending_RM → Pending_Dept_Head
   type=normal | transition criteria: Approval_Tier == "3-Person"
   owner: Reporting_Manager profile
   during: update_fields visible fields=[Comments]
   after: custom deluge — update Status="Pending_Dept_Head"; add Approval_Log entry (Stage="Reporting Manager", Decision="Approved")

5. Dept_Head_Approve: Pending_Dept_Head → Pending_Finance
   type=normal | owner: Department_Head profile
   during: update_fields visible fields=[Comments]
   after: custom deluge — update Status="Pending_Finance"; add Approval_Log entry (Stage="Department Head", Decision="Approved")

6. Finance_Approve: Pending_Finance → Approved
   type=normal | owner: Finance_Controller profile
   during: update_fields visible fields=[Comments]
   after: custom deluge — update Status="Approved"; add Approval_Log entry (Stage="Finance Controller", Decision="Approved")

7. RM_Hold: Pending_RM → On_Hold
   type=normal | owner: Reporting_Manager profile
   during: update_fields visible fields=[Comments]
   after: update Status="On_Hold"; add Approval_Log entry (Decision="On Hold", Stage="Reporting Manager")

8. Dept_Hold: Pending_Dept_Head → On_Hold
   type=normal | owner: Department_Head profile
   during: update_fields visible fields=[Comments]
   after: same pattern

9. Finance_Hold: Pending_Finance → On_Hold
   type=normal | owner: Finance_Controller profile
   during: update_fields visible fields=[Comments]
   after: same pattern

10. RM_Reject: Pending_RM → Rejected
    type=normal | owner: Reporting_Manager profile
    during: update_fields visible fields=[Comments]
    after: update Status="Rejected"; add Approval_Log entry (Decision="Rejected", Stage="Reporting Manager")
    (BRD example: BRQ-2026-0314 rejected by Liam O'Connor)

11. Dept_Reject: Pending_Dept_Head → Rejected
    type=normal | owner: Department_Head profile
    during: update_fields visible fields=[Comments]
    after: same pattern
    (BRD example: BRQ-2026-0312 rejected by Isabella Fischer)

12. Finance_Reject: Pending_Finance → Rejected
    type=normal | owner: Finance_Controller profile
    during: update_fields visible fields=[Comments]
    after: same pattern
    (BRD example: BRQ-2026-0317 rejected by Olivia Bennett)

13. Resubmit_From_Hold: On_Hold → Submitted
    type=normal | owner: Requester profile
    after: update Status="Submitted"

---
REPORTS:

FILE: Components/Spend_Request/Reports/All_Requests.ds
default list All_Requests {
  displayName = "All Requests"
  show all rows from Spend_Request
  (Request_ID, Requester_Name, Department, Item_Description, Estimated_Cost, Approval_Tier, Status, Priority, SLA_Due_Date)
  filters (Status, Department, Approval_Tier, Priority)
  sort by (Submitted_Date descending)
}

FILE: Components/Spend_Request/Reports/Pending_Requests.ds
list Pending_Requests {
  displayName = "Pending Requests"
  show all rows from Spend_Request [Status == "Pending_RM" || Status == "Pending_Dept_Head" || Status == "Pending_Finance"]
  (Request_ID, Requester_Name, Department, Estimated_Cost, Approval_Tier, Status, SLA_Due_Date)
  filters (Status, Department)
  sort by (SLA_Due_Date ascending)
}

**Implementation notes:** ReadDoc fields/subform for the Cost_Breakdown grid. Use Deluge setOptions() for the Item_Description dropdown inside on-user-input of Department — this is the correct Creator approach for dynamic dropdowns rather than a lookup picklist. The blueprint stage names MUST exactly match the blueprint components stages block.

## Module_1_Request_Intake_Page

Module 1 — the polished Request Intake screen. Features a dark fixed left navigation, top-right utility icons, and a single-card form layout. The Requester Detail Row auto-shows the resolved Requester ID chip. The Item/Description dropdown filters by department. An inline colored Approval Tier pill appears beside the Estimated Cost field. An attachment uploader and Save Draft / Submit buttons complete the form.

ReadDoc BEFORE writing: page/index, page/htmlSnippet, page/dsOutputStructure, page/embeddedForm, page/pageOnloadScripts, icons/index, devices/index

This is a page-builder segment. Build the Module 1 UI using a .dshtml HTML snippet wrapping the Creator embedded form.

Page architecture:
- Pages/Request_Intake.ds — main page, no parameters
- Pages/Request_Intake/Nav_Sidebar.dshtml — dark left nav HTML sidebar (shared layout element)
- Pages/Request_Intake/Header_Bar.dshtml — top header with utility icons
- Pages/Request_Intake/Form_Card.dshtml — the single-card form wrapper with all BRD visual components

Design spec (inline pixel styles only, Lato via Google Fonts, no Tailwind, no ZCS components, no emoji):

OVERALL LAYOUT:
- Left nav: fixed, 220px wide, background #1A1F2E, full height, z-index 100
- Main area: margin-left 220px, background #F8FAFC, min-height 100vh
- Header bar: height 56px, white bg, border-bottom 1px solid #E5E7EB, display flex, align-items center, padding 0 24px, justify-content space-between; fixed top, left 220px, right 0
- Content area: padding-top 56px (header offset), padding 32px

LEFT NAV (Nav_Sidebar.dshtml):
- Logo area: top 20px, padding 20px, white text "Zylker" 16px bold + subtitle "Budget Approval" 11px #94A3B8
- Nav items (3): Request Intake, Approvals, Dashboard
  Each: display flex, align-items center, gap 10px, padding 11px 20px, font 14px, color #94A3B8
  Icons: Nucleo outline icons via zc-li-outline classes
    Request Intake: icon class="zc-li-outline ui-1-send-message"
    Approvals: icon class="zc-li-outline ui-1-check-list-3"
    Dashboard: icon class="zc-li-outline ui-1-chart-bar-33"
  Active state (Request Intake): color #FFFFFF, background rgba(37,99,235,0.15), border-left 3px solid #2563EB, padding-left 17px
  Hover: color #FFFFFF
- Nav items are <a> tags linking to the respective pages via Creator's # URL scheme
- Bottom section: version text "v1.0 — Plant 3, Ohio" in #4B5563 11px, padding 20px, position absolute bottom

HEADER BAR (Header_Bar.dshtml):
- Left: breadcrumb "Request Intake" in 14px #374151 bold
- Right: 3 icon buttons with 8px gap:
  Accessibility: class="zc-li-outline ui-1-eye" font-size 18px color #6B7280
  Notification: class="zc-li-outline ui-1-bell-53" font-size 18px color #6B7280
  Profile: class="zc-li-outline ui-1-single-02" font-size 18px color #6B7280
  Each wrapped in a 36px circle button: bg #F3F4F6, border-radius 50%, display flex, align-items center, justify-content center, border none, cursor pointer (use <button> tag styled)

FORM CARD (Form_Card.dshtml):
Page title area:
- "New Spend Request" — font 22px bold #111827, Lato
- Subtitle: "Atlas Precision Manufacturing Inc. · Plant 3, Ohio, USA" — 13px #6B7280, margin-top 4px
- Margin-bottom 24px

White card: background #FFFFFF, border-radius 12px, box-shadow 0 1px 4px rgba(0,0,0,0.08), padding 32px, max-width 900px

Inside the card — embed the Creator form using:
<div zc-component="form" zc-linkname="Spend_Request" zc-height="auto"></div>

The Creator form itself renders all the fields. Wrap it with the card styling. Add supplementary visual context above the zc-component embed:

INFO HEADER STRIP (above the form embed, inside the card):
- Small text row: icon (zc-li-outline ui-1-info-circle) + "Requester ID, Reporting Manager, and Department Head are auto-resolved when your name and department match our employee records." — 12px #6B7280 italic, background #F8FAFC, border-radius 6px, padding 10px 14px, margin-bottom 20px

APPROVAL TIER EXPLANATION (below the form embed but inside the card):
- Small legend showing the three tier thresholds:
  Three inline badges in a row:
  - "Auto" green badge: "Up to $1,000" — bg #DCFCE7, color #16AD78, border-radius 20px, padding 4px 12px, 12px font
  - "2-Person" amber badge: "$1,001 – $50,000" — bg #FEF9C3, color #D97706
  - "3-Person" blue badge: "$50,001+" — bg #EFF6FF, color #2563EB
  Label above: "Approval Tier Thresholds" in 12px #6B7280
  Margin-top 20px, border-top 1px solid #F3F4F6, padding-top 16px

NOTE: The actual form fields (Requester Name, Department, Item/Description, Estimated Cost with Approval Tier formula, Priority, Justification, Attachment, Save Draft / Submit buttons) are rendered natively by the Creator form embed. The Approval Tier pill beside Estimated Cost is the formula field rendered by Creator's native form UI — no hand-coding needed. The dynamic item filtering and requester ID resolution are handled by the on-user-input workflows in the backend form.

All three snippet files (Nav_Sidebar, Header_Bar, Form_Card) are assembled in Pages/Request_Intake.ds:
<page linkname="Request_Intake" displayname="Request Intake"/>
<content>
<zml>
  <layout>
    <row>
      <column width='100%'>
        <!-- nav sidebar -->
        <dsp id='nav_sidebar' elementName='Nav_Sidebar'>
          <![CDATA[htmlpage nav_sidebar() content <%{%> <file name:Nav_Sidebar.dshtml/> <%}%>]]>
        </dsp>
        <!-- header -->
        <dsp id='header_bar' elementName='Header_Bar'>
          <![CDATA[htmlpage header_bar() content <%{%> <file name:Header_Bar.dshtml/> <%}%>]]>
        </dsp>
        <!-- form card -->
        <dsp id='form_card' elementName='Form_Card'>
          <![CDATA[htmlpage form_card() content <%{%> <file name:Form_Card.dshtml/> <%}%>]]>
        </dsp>
      </column>
    </row>
  </layout>
</zml>
</content>

**Implementation notes:** page-builder agent. The form fields are rendered by the zc-component Creator embed. The dshtml snippets provide the shell (nav, header, card wrapper, info strip, tier legend). No JavaScript — all logic is in the Creator form's workflows. Lato via <link href='https://fonts.googleapis.com/css2?family=Lato:wght@400;700&display=swap' rel='stylesheet'> in each dshtml file. No emoji, Nucleo icons only.

## Module_2_Approvals_Page

Module 2 — the full Approvals screen for Reporting Managers, Department Heads, and Finance Controllers. Features stage tabs (All / RM / Dept Head / Finance) filtering the approval queue, an interactive table with requester badges and tier tags, and a side detail panel that opens when a request is clicked — showing the stage stepper, financial stat cards, cost breakdown, activity log, and the Approve/Hold/Reject action bar.

ReadDoc BEFORE writing: page/index, page/htmlSnippet, page/dsOutputStructure, page/pageParameters, page/pageOnloadScripts, page/pageVariables, icons/index

This is a page-builder segment. Hand-build the entire Approvals UI as a custom .dshtml HTML snippet with Deluge data injection. Do NOT use ZCS component library for this screen.

Page architecture:
- Pages/Approvals.ds — main page with parameters: string active_tab, string selected_id
- Pages/Approvals/Nav_Sidebar.dshtml — same dark nav as Module 1 but with Approvals item active
- Pages/Approvals/Header_Bar.dshtml — same top header pattern
- Pages/Approvals/Approvals_Content.dshtml — the full approvals UI (stage tabs + queue table + detail panel)

ONLOAD SCRIPT (in Pages/Approvals.ds <script> block):
Declare variables: string queue_json, string detail_json, string log_json, string active_tab, string selected_id

Script logic:
  active_tab = input.active_tab;
  if(active_tab == null || active_tab == "") { active_tab = "all"; }
  selected_id = input.selected_id;
  if(selected_id == null || selected_id == "") { selected_id = "BRQ-2026-0303"; }

  // Fetch queue based on tab
  if(active_tab == "rm") {
    queue_records = Spend_Request[Status == "Pending_RM"];
  } else if(active_tab == "dept_head") {
    queue_records = Spend_Request[Status == "Pending_Dept_Head"];
  } else if(active_tab == "finance") {
    queue_records = Spend_Request[Status == "Pending_Finance"];
  } else {
    queue_records = Spend_Request[Status != "Auto_Approved"];
  }
  queue_json = queue_records.toJSONList();

  // Fetch selected record detail
  detail_records = Spend_Request[Request_ID == selected_id];
  detail_json = detail_records.toJSONList();

  // Fetch activity log for selected request
  log_records = Approval_Log[Request_ID_Ref == selected_id];
  log_json = log_records.toJSONList();

Approvals_Content.dshtml layout (Lato, inline pixel styles, Cobalt Blue #2563EB, no Tailwind, no emoji, no JavaScript):

Full page wrapper:
<div style="margin-left:220px; padding-top:56px; background:#F8FAFC; min-height:100vh;">
  [Page Header + Tabs + Split Panel]
</div>

PAGE TITLE ROW (padding 24px 28px 0):
"Approvals" — 22px bold #111827
"Review and act on pending spend requests" — 13px #6B7280 margin-top 4px

STAGE TABS ROW (padding 0 28px, margin-top 20px):
Four tab links: All | Reporting Manager | Department Head | Finance
Each tab: <a href="#Approvals?active_tab=all&selected_id=<%=input.selected_id%>"> etc.
Active tab style: font 14px bold, color #2563EB, border-bottom 3px solid #2563EB, padding-bottom 10px
Inactive style: font 14px, color #6B7280, padding-bottom 10px, no border
Tab row container: border-bottom 1px solid #E5E7EB, display flex, gap 32px, margin-bottom 0

Tab counts (shown in parentheses beside each label — fetch counts from queue data):
All: total non-auto requests; RM: Pending_RM count; Dept Head: Pending_Dept_Head count; Finance: Pending_Finance count

SPLIT PANEL (padding 20px 28px, display flex, gap 20px, align-items flex-start):

--- LEFT: APPROVAL QUEUE TABLE ---
Container: flex 1; min-width 0; background #FFFFFF; border-radius 10px; box-shadow 0 1px 3px rgba(0,0,0,0.08); overflow hidden

Table header row: background #F9FAFB; padding 12px 16px; display grid; grid-template-columns: 40px 1fr 140px 100px 90px 120px 120px; gap 12px; font 12px bold #6B7280; border-bottom 1px solid #E5E7EB; text-transform uppercase; letter-spacing 0.5px
Columns: [badge] | REQUESTER | DEPARTMENT | AMOUNT | TIER | STATUS | SLA DUE

Queue rows — rendered via Deluge loop over queue_json:
<%
  queue_list = input.queue_json.toList();
  for each req in queue_list
  {
    req_map = req.toMap();
    req_id = req_map.get("Request_ID");
    req_name = req_map.get("Requester_Name");
    req_dept = req_map.get("Department");
    req_amount = req_map.get("Estimated_Cost");
    req_tier = req_map.get("Approval_Tier");
    req_status = req_map.get("Status");
    req_sla = req_map.get("SLA_Due_Date");
    // Compute initials from name
    initials = req_name.subString(0,1); // first letter; for two initials use split by space
    is_selected = (req_id == input.selected_id);
    is_overdue = false; // compare req_sla to 15-Sep-2026
    ...
  }
%>

Each row: <a href="#Approvals?active_tab=<%=active_tab%>&selected_id=<%=req_id%>">
  display grid, same grid-template-columns as header, padding 14px 16px, border-bottom 1px solid #F3F4F6
  Selected row: background #EFF6FF
  Hover: background #F9FAFB (CSS class via inline style trick — not possible without JS, so just show selected state)

  Requester initials badge: 36px circle, background #2563EB, color #FFFFFF, font 13px bold, display flex, align-items center, justify-content center, border-radius 50%

  Requester cell: 2-line — name 14px bold #111827 + Request_ID 12px #9CA3AF

  Department: 13px #374151

  Amount: 14px bold #111827 — prefix "$"

  Tier tag pill:
  Auto → bg #DCFCE7, color #16AD78
  2-Person → bg #FEF9C3, color #D97706
  3-Person → bg #EFF6FF, color #2563EB
  Style: border-radius 20px, padding 3px 10px, font 12px bold

  Status badge: same pill style with colors by status:
  Pending_RM/Pending_Dept_Head/Pending_Finance → bg #FEF9C3, color #D97706, text "Pending"
  Approved → bg #DCFCE7, color #16AD78
  Rejected → bg #FEE2E2, color #EF4444

  SLA Due: 13px — if overdue: color #EF4444, font-weight bold + " (Overdue)" suffix; else color #374151

QUEUE DATA from BRD §14 (also fetched live from DB — these are the BRD reference values for the default view):
All tab shows 18 requests (excluding Auto_Approved ones from view defaults)
Pending queue (6 entries):
  BRQ-2026-0311 | Elena Petrova | Procurement & Stores | $22,000 | 2-Person | Pending_RM | 15 Nov 2026
  BRQ-2026-0309 | Amara Okafor | Quality Assurance | $142,000 | 3-Person | Pending_RM | 05 Sep 2026 OVERDUE
  BRQ-2026-0303 | Carlos Medina | Production | $128,000 | 3-Person | Pending_Dept_Head | 20 Nov 2026
  BRQ-2026-0306 | Wei Zhang | Maintenance & Utilities | $94,000 | 3-Person | Pending_Dept_Head | 10 Sep 2026 OVERDUE
  BRQ-2026-0302 | Carlos Medina | Production | $18,500 | 2-Person | Pending_Finance | 10 Dec 2026
  BRQ-2026-0315 | Daniel Cohen | IT & Systems | $115,000 | 3-Person | Pending_Finance | 25 Nov 2026

--- RIGHT: DETAIL PANEL ---
Container: width 440px; flex-shrink 0; background #FFFFFF; border-radius 10px; box-shadow 0 1px 3px rgba(0,0,0,0.08); overflow hidden

Default selected record: BRQ-2026-0303 (Production, 3-Person, Pending Dept Head)

Render detail from detail_json. Parse map, extract all fields.

DETAIL PANEL HEADER:
Background #F9FAFB, border-bottom 1px solid #E5E7EB, padding 16px 20px
Request ID: 15px bold #111827 (e.g. "BRQ-2026-0303")
Item description: 13px #6B7280 below the ID

STAGE STEPPER (padding 20px, border-bottom 1px solid #F3F4F6):
Horizontal stepper showing stages relevant to the request's Approval_Tier:
2-Person: RM → Finance (2 circles)
3-Person: RM → Dept Head → Finance (3 circles)

Each step:
- Circle 36px: completed = bg #2563EB, white check icon (zc-li-outline ui-1-check-simple); active = border 2px solid #2563EB, text #2563EB; pending = border 2px solid #D1D5DB, text #9CA3AF
- Label below circle: 11px, completed=bold #2563EB, active=bold #2563EB, pending=#9CA3AF
- Connector line between circles: height 2px; completed segments = bg #2563EB; pending = bg #E5E7EB

For BRQ-2026-0303 (3-Person, currently at Dept Head):
  RM circle: completed (filled blue + check)
  →
  Dept Head circle: active (blue border)
  →
  Finance circle: pending (gray)

FINANCIAL STAT CARDS (3 cards, display flex, gap 12px, padding 16px 20px, border-bottom 1px solid #F3F4F6):
Card style: flex 1, border-radius 8px, padding 14px 16px
- Request Value: bg #EFF6FF — value "$128,000" (20px bold #111827) — label "Request Value" (11px #6B7280)
- Approval Tier: bg #FFF8E7 — value "3-Person" (16px bold #D97706) — label "Approval Tier" (11px #6B7280)
- SLA Due Date: bg #F0FDF4 (green if not overdue, #FEF2F2 red if overdue) — value "20 Nov 2026" — label "SLA Due Date"

REQUEST DETAILS (2-column grid, padding 16px 20px, border-bottom 1px solid #F3F4F6):
Grid: display grid, grid-template-columns 1fr 1fr, gap 12px
Each field: label (11px #6B7280 uppercase letter-spacing 0.5px) + value (13px bold #111827) stacked
Fields: Department, Requester, Priority, Submitted Date, Item/Description (full width, span 2 columns)
For BRQ-2026-0303: Production | Carlos Medina | High | 25 Aug 2026 | CNC lathe machine — Unit 2

COST BREAKDOWN TABLE (padding 16px 20px, border-bottom 1px solid #F3F4F6):
Section label: "Cost Breakdown" 13px bold #374151, margin-bottom 12px
Table: width 100%, border-collapse collapse
Header row: background #F9FAFB, font 11px bold #6B7280, padding 8px 12px, text-transform uppercase
Columns: LINE ITEM | AMOUNT (right-aligned)
Data rows: font 13px #374151, padding 8px 12px, border-bottom 1px solid #F3F4F6
Total row: background #F9FAFB, font 13px bold #111827, border-top 2px solid #E5E7EB

BRQ-2026-0303 breakdown:
  CNC lathe machine unit | $112,000
  Installation & calibration | $9,500
  Contingency | $6,500
  TOTAL | $128,000

BRQ-2026-0302 breakdown:
  Sensor units (Qty 4) | $14,000
  Installation & calibration | $3,000
  Contingency | $1,500
  TOTAL | $18,500

ACTIVITY LOG (padding 16px 20px, border-bottom 1px solid #F3F4F6):
Section label: "Activity Log" 13px bold #374151, margin-bottom 12px
Render from log_json sorted by timestamp:
Each entry: display flex, gap 12px, margin-bottom 14px
  Left column: vertical line (2px, color depends on decision) + colored circle dot 10px:
    Submitted → gray #9CA3AF
    Approved → green #16AD78
    On Hold → amber #F59E0B
    Rejected → red #EF4444
    Auto-Approved → blue #2563EB
  Right column:
    Timestamp: 11px #9CA3AF (e.g. "28 Aug 2026, 09:30 AM")
    Description: 13px #374151 (e.g. "Approved by Jason Miller — Reporting Manager")
    Comment if any: 12px italic #9CA3AF, margin-top 2px

BRQ-2026-0303 log entries:
  25 Aug 2026, 10:05 AM — Submitted by Carlos Medina (Production)
  28 Aug 2026, 09:30 AM — Approved by Jason Miller (Reporting Manager) — routed to Department Head
  Awaiting — Department Head Henrik Larsson (SLA due 20 Nov 2026)

ACTION BAR (padding 16px 20px, sticky bottom, background #FFFFFF, border-top 1px solid #E5E7EB):
3 equal-width buttons, solid fill, no icons, side by side with gap 10px:
Button style: flex 1, height 40px, border-radius 6px, font 14px bold Lato, border none, cursor pointer, rendered as <a> tags styled as buttons
- Approve: background #16AD78, color #FFFFFF, href links to Approvals page with decision pre-set OR links to Approval_Action form page
- Hold: background #F59E0B, color #FFFFFF
- Reject: background #EF4444, color #FFFFFF

APPROVAL ACTION LINK: clicking Approve/Hold/Reject navigates to a URL that opens the Approval_Action embedded form (or Creator's native approval transition) for that request. Use: href="/creator/[app]/Approval_Action?Request_ID_Input=<%=req_id%>&Approver_Stage=<%=current_stage%>" — the actual URL format depends on the Creator app URL scheme. For the page, link to a Creator form page for Approval_Action form, passing the Request ID and stage via URL.

All Deluge loops and variable injections use <% %> and <%= %>. No <script> tags, no onclick JavaScript.

**Implementation notes:** page-builder agent. Build Approval_Action form (simple form with Request_ID_Input, Approver_Name, Approver_Stage, Decision, Comments fields + on-success workflow that updates Spend_Request Status and creates Approval_Log entry) plus the full Approvals page HTML. The onload script parses page parameters and fetches live data. The detail panel defaults to BRQ-2026-0303 if no selected_id is provided.

## Module_3_Dashboard_Page

Module 3 — the leadership Dashboard. Displays four KPI stat cards (Total Requested, Total Approved, Pending, Rejected), an SVG donut chart for Approval Tier spend, a horizontal bar chart for department-wise spend, a stage-by-stage approval funnel, and an escalation list showing the two overdue requests — all powered by live Deluge data from the Spend Request form.

ReadDoc BEFORE writing: page/index, page/htmlSnippet, page/dsOutputStructure, page/pageOnloadScripts, page/pageVariables, icons/index

This is a page-builder segment. Hand-build the entire Dashboard as custom HTML/CSS using .dshtml snippets with Deluge data injection. Do NOT use ZCS/native Creator chart components — hand-build the SVG donut and CSS bars.

Page architecture:
- Pages/Dashboard.ds — main page, no parameters
- Pages/Dashboard/Nav_Sidebar.dshtml — dark nav, Dashboard item active
- Pages/Dashboard/Header_Bar.dshtml — same top header
- Pages/Dashboard/Dashboard_Content.dshtml — the full dashboard: stat cards + all charts + escalation list

ONLOAD SCRIPT (in Pages/Dashboard.ds <script> block):
Declare variables:
  string total_requested_value, string approved_value, string pending_value, string rejected_value
  string total_requests, string approved_count, string pending_count, string rejected_count
  string auto_value, string two_person_value, string three_person_value
  string auto_count, string two_person_count, string three_person_count
  string dept_json, string funnel_json, string escalation_json

Script logic (fetch from Spend_Request form; fall back to BRD sample values if count is 0):
  // Summary
  all_reqs = Spend_Request.count();
  if(all_reqs > 0) {
    total_requested_value = "$" + zoho.toString(Spend_Request.sum(Estimated_Cost));
    ...
  } else {
    // BRD fallback values
    total_requested_value = "$778,360";
    approved_value = "$113,860";
    pending_value = "$519,500";
    rejected_value = "$145,000";
    total_requests = "18";
    approved_count = "9";
    pending_count = "6";
    rejected_count = "3";
    auto_count = "6"; auto_value = "$3,360";
    two_person_count = "6"; two_person_value = "$141,000";
    three_person_count = "6"; three_person_value = "$634,000";
  }

Dashboard_Content.dshtml layout (Lato, inline pixel styles, Cobalt Blue #2563EB, no Tailwind, no emoji, no JavaScript):

MAIN WRAPPER: margin-left 220px; padding-top 56px; background #F8FAFC; min-height 100vh; padding 72px 28px 28px 248px

PAGE HEADER ROW (display flex, justify-content space-between, align-items flex-start, margin-bottom 24px):
  Left: "Spend & Approval Dashboard" 22px bold #111827 | subtitle "Atlas Precision Manufacturing · as of 15 Sep 2026" 13px #6B7280
  Right: period chip "Reporting Period: Sep 2026" — bg #F3F4F6, border-radius 20px, padding 6px 14px, 12px #374151

SUMMARY STAT ROW (display flex, gap 16px, margin-bottom 24px):
4 equal cards. Card style: flex 1; background #FFFFFF; border-radius 10px; box-shadow 0 1px 3px rgba(0,0,0,0.08); padding 20px 24px; position relative; overflow hidden; border-left 4px solid <accent-color>

Inside each card:
  - Big number: 24px bold #111827 (e.g. "$778,360")
  - Count badge below number: 12px #6B7280 (e.g. "18 requests")
  - Label: 12px #9CA3AF uppercase letter-spacing 0.5px, margin-top 8px
  - Background accent tint: ::after pseudo element OR a <div> absolutely positioned bottom-right with the accent color at 6% opacity and large font-size (decorative)

Cards (from BRD §15):
  1. Total Requested | $778,360 | 18 requests | accent #2563EB
  2. Total Approved | $113,860 | 9 requests | accent #16AD78
  3. Pending Value | $519,500 | 6 requests | accent #F59E0B
  4. Rejected Value | $145,000 | 3 requests | accent #EF4444

(Use Deluge variables <%=input.total_requested_value%> etc. for live data; show BRD fallback when DB empty)

ROW 2 (display flex, gap 20px, margin-bottom 24px, align-items flex-start):

LEFT CARD — APPROVAL TIER DONUT (width 38%, background #FFFFFF, border-radius 10px, box-shadow 0 1px 3px rgba(0,0,0,0.08), padding 24px):
Title: "Approval Tier Breakdown" 15px bold #111827, margin-bottom 20px

SVG Donut chart (hand-coded, 260x260px, viewBox="0 0 260 260"):
Circle center: cx=130, cy=130, r=80
Circumference = 2 * π * 80 = 502.65

Segment values (from BRD §15):
  Total = $778,360
  Auto = $3,360 → 3360/778360 = 0.43% → dasharray = 2.17 of 502.65
  2-Person = $141,000 → 18.12% → dasharray = 91.02
  3-Person = $634,000 → 81.45% → dasharray = 409.43

Draw three <circle> elements with stroke-dasharray and stroke-dashoffset:
  Base circle: fill=none, stroke=#F3F4F6, stroke-width=28 (shows track)
  Segment 1 (3-Person, largest — draw first with correct offset): stroke=#2563EB
  Segment 2 (2-Person): stroke=#F59E0B, offset = 3-Person length
  Segment 3 (Auto, tiny): stroke=#16AD78, offset = 3-Person + 2-Person length

Center text overlay: position absolute centered text "$778K" 18px bold #111827 + "Total" 11px #9CA3AF

Legend below SVG (3 rows):
  Each: colored dot (10px circle) + tier name (13px #374151) + value (13px bold #111827) + count (12px #9CA3AF) — display flex, align-items center, gap 10px, margin-bottom 8px
  Auto | $3,360 | 6 requests | dot #16AD78
  2-Person | $141,000 | 6 requests | dot #F59E0B
  3-Person | $634,000 | 6 requests | dot #2563EB

RIGHT CARD — DEPARTMENT SPEND BAR CHART (flex 1, background #FFFFFF, border-radius 10px, box-shadow 0 1px 3px rgba(0,0,0,0.08), padding 24px):
Title: "Department-Wise Spend" 15px bold #111827, margin-bottom 20px
Subtitle: "By total estimated cost (USD)" 12px #9CA3AF, margin-top -16px, margin-bottom 20px

Data (from BRD §15, descending order):
  IT & Systems: $163,560
  Quality Assurance: $152,280
  Production: $147,350
  Maintenance & Utilities: $126,620
  Procurement & Stores: $108,340
  HR & Admin: $80,210
  Max = $163,560

For each department, render a row:
  display flex, align-items center, gap 12px, margin-bottom 14px
  Dept name: 13px #374151, width 160px, flex-shrink 0
  Bar track: flex 1, background #F1F5F9, border-radius 4px, height 28px; inside: filled bar width=(value/163560*100)%, background #2563EB, border-radius 4px
  Value label: 12px bold #374151, width 70px, text-align right

  Vary bar tint by index (lighter for lower-ranked): use opacity 1.0, 0.85, 0.75, 0.65, 0.55, 0.45 on the same Cobalt Blue

ROW 3 (display flex, gap 20px, margin-bottom 24px, align-items flex-start):

LEFT CARD — APPROVAL FUNNEL (width 55%, background #FFFFFF, border-radius 10px, box-shadow 0 1px 3px rgba(0,0,0,0.08), padding 24px):
Title: "Approval Funnel" 15px bold #111827, margin-bottom 20px
"Stage progression from submission to resolution" 12px #9CA3AF, margin-top -16px, margin-bottom 20px

Data (from BRD §15):
  Submitted: 18 | Auto-Cleared: 6 | Sent to Chain: 12 | Pending RM: 2 | Pending Dept Head: 2 | Pending Finance: 2 | Approved: 3 | Rejected: 3
  Max = 18

For each stage row:
  display flex, align-items center, gap 12px, margin-bottom 12px
  Stage name: 13px #374151, width 160px, flex-shrink 0
  Bar track: flex 1, background #F1F5F9, border-radius 4px, height 24px
  Filled bar: width=(count/18*100)%, height 24px, border-radius 4px, color varies:
    Submitted → #9CA3AF
    Auto-Cleared → #16AD78
    Sent to Chain → #60A5FA
    Pending RM/DH/Finance → #F59E0B
    Approved → #16AD78
    Rejected → #EF4444
  Count label: 12px bold #374151, width 30px, text-align right

RIGHT CARD — ESCALATION LIST (flex 1, background #FFFFFF, border-radius 10px, box-shadow 0 1px 3px rgba(0,0,0,0.08), padding 24px):
Title area: display flex, justify-content space-between, align-items center, margin-bottom 20px
  Title: "Escalation — SLA Overdue" 15px bold #111827
  Badge: "2 overdue" — bg #FEE2E2, color #EF4444, border-radius 20px, padding 3px 10px, 12px bold

Data (from BRD §15 Escalation List as of 15 Sep 2026):
  BRQ-2026-0309 | Quality Assurance | Pending RM | 10 days overdue
  BRQ-2026-0306 | Maintenance & Utilities | Pending Dept Head | 5 days overdue

For each entry:
  background #FEF2F2, border-left 3px solid #EF4444, border-radius 8px, padding 14px 16px, margin-bottom 12px
  Top row: Request ID 14px bold #111827 (left) + "X days overdue" 13px bold #EF4444 (right, display flex justify-content space-between)
  Bottom row: Department 12px #6B7280 + " · " + Stage 12px #6B7280 (e.g. "Quality Assurance · Pending RM")

FOOTER ROW — AVERAGE TURNAROUND (margin-top 8px, display flex, gap 16px, align-items center):
Label: "Avg. Approval Turnaround" 13px #374151
Two badges (from BRD §15):
  2-Person: "2.4 days" — bg #F3F4F6, border-radius 20px, padding 5px 14px, 13px #374151, font-weight bold prefix "2-Person: "
  3-Person: "5.1 days" — same style

Assemble all three snippet files in Pages/Dashboard.ds:
<page linkname="Dashboard" displayname="Dashboard"/>
<content>
<zml>
  <layout>
    <row>
      <column width='100%'>
        <dsp id='nav_sidebar' elementName='Nav_Sidebar'> <![CDATA[htmlpage nav_sidebar() content <%{%> <file name:Nav_Sidebar.dshtml/> <%}%>]]> </dsp>
        <dsp id='header_bar' elementName='Header_Bar'> <![CDATA[htmlpage header_bar() content <%{%> <file name:Header_Bar.dshtml/> <%}%>]]> </dsp>
        <dsp id='dashboard_content' elementName='Dashboard_Content'> <![CDATA[htmlpage dashboard_content() content <%{%> <file name:Dashboard_Content.dshtml/> <%}%>]]> </dsp>
      </column>
    </row>
  </layout>
</zml>
</content>
<script>
  [onload script here]
</script>

**Implementation notes:** page-builder agent. The SVG donut uses hardcoded segment lengths computed from BRD values (they won't change with live data since the total changes; ideally recompute in Deluge as percentages of the live sum — compute dasharray dynamically in the onload script and pass as string variables). For the bar charts, compute bar widths as percentages in Deluge and pass as string variables (e.g. it_width = '100%', qa_width = '93%', etc.) that are injected via <%=input.it_width%>. No JavaScript. No emoji. Nucleo icons via zc-li-outline where used.

## Permissions

Adds full access permissions to each profile — controlling who can submit requests, who can approve, and who can view the dashboard. Requesters see only their own data. Managers get approval scope. Leadership gets read-only dashboard access.

ReadDoc BEFORE writing: permissions/index, permissions/profiles

Edit each profile shell file to ADD ModulePermissions. Forms in scope: Spend_Request, Employee_Directory, Request_Catalog, Approval_Log, Approval_Action. Pages in scope: Request_Intake, Approvals, Dashboard.

Requester profile (Permissions/Profiles/Requester.ds):
  Spend_Request: enabled=Create,View,Edit,Tab,Search | allFieldsVisible=true
    ReportPermissions: All_Requests={View}, Pending_Requests={View}
  Employee_Directory: enabled=View,Search | allFieldsVisible=true
    ReportPermissions: All_Employees={View}
  Request_Catalog: enabled=View,Search | allFieldsVisible=true
    ReportPermissions: Catalog_Items={View}
  Approval_Log: enabled=View,Search | allFieldsVisible=true
    ReportPermissions: All_Decisions={View}
  Pages: Request_Intake={tab} — No Approvals page, No Dashboard

Reporting_Manager profile (Permissions/Profiles/Reporting_Manager.ds):
  Spend_Request: enabled=Create,View,Viewall,Edit,Modifyall,Tab,Search,Export | allFieldsVisible=true
    ReportPermissions: All_Requests={View,Edit,Delete}, Pending_Requests={View,Edit,Delete}
  Employee_Directory: enabled=View,Viewall,Search | allFieldsVisible=true
    ReportPermissions: All_Employees={View}
  Request_Catalog: enabled=View,Viewall,Search | allFieldsVisible=true
    ReportPermissions: Catalog_Items={View}
  Approval_Log: enabled=Create,View,Viewall,Tab,Search | allFieldsVisible=true
    ReportPermissions: All_Decisions={View}
  Approval_Action: enabled=Create,View,Viewall,Tab | allFieldsVisible=true
    ReportPermissions: Approval_Actions={View}
  Pages: Request_Intake={tab}, Approvals={tab} — No Dashboard

Department_Head profile (Permissions/Profiles/Department_Head.ds):
  Same as Reporting_Manager above (same access scope)
  Pages: Request_Intake={tab}, Approvals={tab} — No Dashboard

Finance_Controller profile (Permissions/Profiles/Finance_Controller.ds):
  Same as Reporting_Manager above but also:
  Pages: Request_Intake={tab}, Approvals={tab}, Dashboard={tab}

Operations_Leadership profile (Permissions/Profiles/Operations_Leadership.ds):
  Spend_Request: enabled=View,Viewall,Tab,Search,Export | allFieldsVisible=true
    ReportPermissions: All_Requests={View}, Pending_Requests={View}
  Employee_Directory: enabled=View,Viewall,Search | allFieldsVisible=true
    ReportPermissions: All_Employees={View}
  Request_Catalog: enabled=View,Viewall,Search | allFieldsVisible=true
    ReportPermissions: Catalog_Items={View}
  Approval_Log: enabled=View,Viewall,Tab,Search | allFieldsVisible=true
    ReportPermissions: All_Decisions={View}
  Pages: Dashboard={tab}

**Implementation notes:** Convergence segment. Read the already-written profile shell files first, then add ModulePermissions.

## UI_Configuration

Sets up the web navigation menu linking all three visible modules (Request Intake, Approvals, Dashboard), form label placements, and quickview/detailview report layouts — ensuring Creator's native navigation is consistent with the polished custom page UIs.

ReadDoc BEFORE writing: devices/index, devices/devices.md (or devices/webDeviceConfig.md), devices/menu.md, devices/deviceReports.md, devices/details/menu_examples, devices/details/report_examples, icons/index

FILE: UI/web/forms.ds
Form label placements:
  Spend_Request: field alignment = top
  Employee_Directory: field alignment = left
  Request_Catalog: field alignment = left
  Approval_Log: field alignment = left
  Approval_Action: field alignment = top

FILE: UI/web/menu.ds
Web navigation with pages as primary entry points:

space "Zylker Budget Approval"
{
  section "Modules"
  {
    page Request_Intake { icon = "<valid-icon>" displayname = "Request Intake" }
    page Approvals { icon = "<valid-icon>" displayname = "Approvals" }
    page Dashboard { icon = "<valid-icon>" displayname = "Dashboard" }
  }
  section "Data Management"
  {
    form Spend_Request { icon = "<valid-icon>" displayname = "All Requests" }
    form Employee_Directory { icon = "<valid-icon>" displayname = "Employees" }
    form Request_Catalog { icon = "<valid-icon>" displayname = "Request Catalog" }
    form Approval_Log { icon = "<valid-icon>" displayname = "Approval Log" }
  }
  section "Analytics"
  {
    type = shared_user_report_section
  }
}

NOTE: ReadDoc icons/index and relevant icon category docs to find valid icon identifiers before writing menu.ds.

FILE: UI/web/customize.ds
Font: Lato (via Google Fonts or platform font setting)
Accent/theme color: #2563EB

DEVICE REPORT LAYOUTS (web_ prefix, co-located with each report):

FILE: Components/Spend_Request/Reports/DeviceUI/web_All_Requests.ds
quickview: Request_ID, Requester_Name, Department, Estimated_Cost, Approval_Tier, Status, Priority, SLA_Due_Date
detailview: all above fields plus Item_Description, Justification, Submitted_Date

FILE: Components/Spend_Request/Reports/DeviceUI/web_Pending_Requests.ds
quickview: Request_ID, Requester_Name, Department, Estimated_Cost, Approval_Tier, Status, SLA_Due_Date
detailview: all above fields plus Priority, Item_Description, Submitted_Date

FILE: Components/Employee_Directory/Reports/DeviceUI/web_All_Employees.ds
quickview: Employee_Name, Department, Requester_ID
detailview: Employee_Name, Department, Requester_ID, Reporting_Manager, Department_Head

FILE: Components/Request_Catalog/Reports/DeviceUI/web_Catalog_Items.ds
quickview: Department, Item_Description, Catalog_Cost
detailview: Department, Item_Description, Catalog_Cost

FILE: Components/Approval_Log/Reports/DeviceUI/web_All_Decisions.ds
quickview: Request_ID_Ref, Stage, Approver_Name, Decision, Decision_Timestamp
detailview: Request_ID_Ref, Stage, Approver_Name, Decision, Decision_Timestamp, Comments

FILE: Components/Approval_Action/Reports/DeviceUI/web_Approval_Actions.ds
quickview: Request_ID_Input, Approver_Name, Approver_Stage, Decision
detailview: Request_ID_Input, Approver_Name, Approver_Stage, Decision, Comments

**Implementation notes:** Convergence segment. ReadDoc icons/index before writing menu to find valid icon link names. The Analytics section must use type = shared_user_report_section. Every report must have a web_ DeviceUI file.