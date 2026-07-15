# ProcureHub Design System

**Status:** Approved<br>
**Date:** 2026-07-14<br>
**Approved date:** 2026-07-15<br>
**Approved by:** Product Owner<br>
**Product:** ProcureHub<br>
**Design direction:** Modern Enterprise Dashboard<br>
Theme: Navy + Blue + Slate

## 1. Purpose

This document is the visual and interaction source of truth for ProcureHub. It applies to administration, procurement, approval, document-template, reporting, and notification interfaces.

Feature specifications define business behavior. This file defines how that behavior is presented consistently. When a feature specification and this document appear to conflict, business behavior follows the feature specification while visual and interaction decisions follow this design system unless the feature specification explicitly documents an exception.

The workflow-specific behavior is defined in:

```text
docs/superpowers/specs/2026-07-13-procurehub-workflow-design.md
```

## 2. Product Experience

ProcureHub should feel:

- Reliable
- Clear
- Efficient
- Professional
- Modern
- Organized
- Enterprise-ready

The interface must help users answer these questions quickly:

- What requires my action?
- What is the current document status?
- Who is the document waiting for?
- Why was the document returned or rejected?
- What changed since the previous revision?
- Is the task approaching or exceeding its SLA?
- Which company, branch, and department context am I viewing?

## 3. Core Design Principles

### 3.1 Clarity First

Critical information must be visible without opening tooltips or nested menus:

- Document number and type
- Company, branch, and owner department
- Current document status
- Current approval step and approver
- Amount and currency
- Due time or overdue state
- Return or rejection reason

### 3.2 Separate Different Kinds of Status

Document lifecycle, approval-task status, template status, and generated-file status are separate concepts and must not be presented as one badge.

Example:

```text
Document status: Pending Approval
Current task: Waiting for IT Division Head
PDF status: Not Generated
```

### 3.3 Action before Analytics

Dashboards prioritize actionable work, overdue tasks, returned documents, and failed operations before charts. Charts are used only when they support a decision.

### 3.4 Dense but Comfortable

ProcureHub is desktop-first and data-heavy. It should show enough information for daily work without cramped rows, excessive whitespace, or oversized cards.

### 3.5 Progressive Disclosure

Lists show essential information. Detailed history, attachments, revisions, and secondary metadata appear in detail pages, drawers, or expandable sections.

### 3.6 Consistent Workflow Language

The same action uses the same label throughout the system:

```text
Save Draft
Submit for Approval
Approve
Acknowledge
Return for Revision
Reject
Cancel
Generate PDF
View Timeline
```

Reject and Return for Revision are never used interchangeably.

## 4. Color System

### 4.1 Brand Palette

```css
:root {
  --brand-50: #eff6ff;
  --brand-100: #dbeafe;
  --brand-200: #bfdbfe;
  --brand-300: #93c5fd;
  --brand-400: #60a5fa;
  --brand-500: #3b82f6;
  --brand-600: #1d4ed8;
  --brand-700: #1e40af;
  --brand-800: #1e3a8a;
  --brand-900: #172554;
  --brand-950: #0b1739;

  --color-primary: #1d4ed8;
  --color-primary-hover: #1e40af;
  --color-primary-active: #1e3a8a;
  --color-primary-soft: #eff6ff;

  --color-accent: #0ea5e9;
  --color-accent-soft: #e0f2fe;
}
```

Primary blue communicates the main action, current workflow step, selected state, and navigational focus. Accent cyan is supplementary and must not compete with the primary action.

### 4.2 Neutral Palette

```css
:root {
  --slate-50: #f8fafc;
  --slate-100: #f1f5f9;
  --slate-200: #e2e8f0;
  --slate-300: #cbd5e1;
  --slate-400: #94a3b8;
  --slate-500: #64748b;
  --slate-600: #475569;
  --slate-700: #334155;
  --slate-800: #1e293b;
  --slate-900: #0f172a;
  --slate-950: #020617;

  --background-app: #f8fafc;
  --background-surface: #ffffff;
  --background-muted: #f1f5f9;

  --border-default: #e2e8f0;
  --border-strong: #cbd5e1;

  --text-primary: #0f172a;
  --text-secondary: #475569;
  --text-muted: #64748b;
  --text-disabled: #94a3b8;
}
```

### 4.3 Sidebar

```css
:root {
  --sidebar-background: #071b36;
  --sidebar-background-deep: #041326;
  --sidebar-border: rgb(255 255 255 / 8%);
  --sidebar-text: #cbd5e1;
  --sidebar-text-active: #ffffff;
  --sidebar-icon: #94a3b8;
  --sidebar-item-hover: rgb(59 130 246 / 12%);
  --sidebar-item-active: #1d4ed8;
}
```

The sidebar may use this subtle two-stop background:

```css
background: linear-gradient(180deg, #071b36 0%, #041326 100%);
```

No bright, multicolor, or animated gradients are allowed.

### 4.4 Semantic Status Tokens

```css
:root {
  --status-draft-text: #475569;
  --status-draft-bg: #f1f5f9;
  --status-draft-border: #cbd5e1;

  --status-pending-text: #b45309;
  --status-pending-bg: #fffbeb;
  --status-pending-border: #fde68a;

  --status-approved-text: #15803d;
  --status-approved-bg: #f0fdf4;
  --status-approved-border: #bbf7d0;

  --status-returned-text: #c2410c;
  --status-returned-bg: #fff7ed;
  --status-returned-border: #fed7aa;

  --status-rejected-text: #b91c1c;
  --status-rejected-bg: #fef2f2;
  --status-rejected-border: #fecaca;

  --status-cancelled-text: #4b5563;
  --status-cancelled-bg: #f3f4f6;
  --status-cancelled-border: #d1d5db;

  --status-closed-text: #334155;
  --status-closed-bg: #e2e8f0;
  --status-closed-border: #cbd5e1;

  --status-overdue-text: #b91c1c;
  --status-overdue-bg: #fef2f2;
  --status-overdue-border: #fca5a5;

  --status-waiting-text: #475569;
  --status-waiting-bg: #f8fafc;
  --status-waiting-border: #cbd5e1;

  --status-acknowledged-text: #1d4ed8;
  --status-acknowledged-bg: #eff6ff;
  --status-acknowledged-border: #bfdbfe;

  --status-skipped-text: #64748b;
  --status-skipped-bg: #f8fafc;
  --status-skipped-border: #e2e8f0;

  --status-resolution-error-text: #b91c1c;
  --status-resolution-error-bg: #fef2f2;
  --status-resolution-error-border: #fca5a5;

  --status-generating-text: #1d4ed8;
  --status-generating-bg: #eff6ff;
  --status-generating-border: #bfdbfe;

  --status-generated-text: #15803d;
  --status-generated-bg: #f0fdf4;
  --status-generated-border: #bbf7d0;

  --status-failed-text: #b91c1c;
  --status-failed-bg: #fef2f2;
  --status-failed-border: #fecaca;

  --status-superseded-text: #475569;
  --status-superseded-bg: #f1f5f9;
  --status-superseded-border: #cbd5e1;

  --status-voided-text: #991b1b;
  --status-voided-bg: #fef2f2;
  --status-voided-border: #fca5a5;

  --color-success: #15803d;
  --color-warning: #c2410c;
  --color-danger: #b91c1c;
  --color-info: #1d4ed8;
  --focus-ring: rgb(59 130 246 / 35%);
  --overlay-background: rgb(15 23 42 / 55%);
}
```

Status is never represented by color alone. Every status includes text and, when useful, an icon.

## 5. Typography

### 5.1 Font Stack

```css
:root {
  --font-sans:
    "IBM Plex Sans Thai",
    "IBM Plex Sans",
    "Noto Sans Thai",
    "Noto Sans",
    system-ui,
    sans-serif;
}
```

IBM Plex Sans Thai is the primary application font. Noto Sans Thai is the fallback. Do not mix display typefaces into normal application screens.

### 5.2 Weight

```text
Regular: 400
Medium: 500
Semibold: 600
Bold: 700
```

Weight 300 is not used for operational text.

### 5.3 Type Scale

```css
:root {
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;
  --text-3xl: 1.875rem;
}
```

Usage:

| Element | Size | Weight |
| --- | ---: | ---: |
| Page title | 24-30 px | 600 |
| Section title | 18-20 px | 600 |
| Card title | 14-16 px | 500-600 |
| Body | 14-16 px | 400 |
| Form label | 13-14 px | 500 |
| Table | 13-14 px | 400-500 |
| Helper text | 12 px | 400 |
| KPI value | 28-36 px | 600 |

## 6. Spacing, Radius, and Shadow

### 6.1 Spacing

Use a 4 px base unit.

```css
:root {
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-8: 32px;
  --space-10: 40px;
  --space-12: 48px;
}
```

Defaults:

- Icon to label: 8 px
- Compact control gap: 8-12 px
- Form field gap: 16-20 px
- Card padding: 16-24 px
- Page padding: 24-32 px
- Section gap: 24-32 px

### 6.2 Radius

```css
:root {
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-full: 9999px;
}
```

Usage:

- Button and input: 8 px
- Card and table container: 12 px
- Modal and drawer: 12-16 px
- Badge: full

### 6.3 Shadow

```css
:root {
  --shadow-xs: 0 1px 2px rgb(15 23 42 / 4%);
  --shadow-sm:
    0 1px 3px rgb(15 23 42 / 6%),
    0 1px 2px rgb(15 23 42 / 4%);
  --shadow-md: 0 6px 18px rgb(15 23 42 / 8%);
  --shadow-overlay: 0 20px 40px rgb(15 23 42 / 16%);
}
```

Cards usually use a border and `shadow-xs`. Heavy shadows, glow, and glass effects are prohibited.

## 7. Application Shell

```css
:root {
  --sidebar-width: 248px;
  --sidebar-collapsed-width: 72px;
  --topbar-height: 64px;
  --content-max-width: 1600px;
}
```

Structure:

```text
App Shell
├── Fixed Sidebar
├── Top Navigation
└── Main Content
```

### 7.1 Sidebar

- Full width on desktop, collapsible to icons
- Logo and ProcureHub wordmark at the top
- Menu grouped by user task rather than database entity
- Active item uses primary blue with white text
- Permission-filtered navigation
- Collapsed icon-only items always have accessible names and tooltips

Suggested information architecture:

```text
Dashboard

Procurement
- Purchase Requisitions
- Purchase Orders
- Receiving
- Vendors

Work
- My Approval Tasks
- Returned Documents
- My Documents

Documents
- Generated Documents
- Document Templates
- Contracts and Tag Sheets

Analytics
- Reports
- SLA Monitoring

Administration
- Companies and Branches
- Divisions and Departments
- Users and Positions
- Approval Workflows
- Document Numbering
- Work Calendars
- Settings
```

### 7.2 Top Navigation

Contains:

- Contextual page breadcrumb when needed
- Global search
- Notification center
- Company and branch selector
- User menu

Company and branch context must always be visible when the user can access more than one context.

Changing company or branch reloads data under the selected authorization context. If a page contains unsaved changes, context switching requires confirmation and must never silently discard user input.

### 7.3 Main Content

- App background uses `--background-app`
- Content aligns to a consistent page grid
- Wide tables may use the available viewport up to 1600 px
- Operational pages are not constrained to a narrow marketing-style column

## 8. Responsive Behavior

Breakpoints:

```css
:root {
  --breakpoint-sm: 640px;
  --breakpoint-md: 768px;
  --breakpoint-lg: 1024px;
  --breakpoint-xl: 1280px;
  --breakpoint-2xl: 1536px;
}
```

### Desktop, 1280 px and above

- Full sidebar
- Four KPI cards per row when space permits
- Full data tables
- Two-column detail layouts when useful
- Workflow editing enabled

### Tablet, 768-1279 px

- Collapsed sidebar
- Two KPI cards per row
- One-column detail layout
- Filters use a drawer
- Workflow editing only in landscape when enough width exists

### Mobile, below 768 px

- Sidebar becomes a drawer
- One KPI card per row
- Tables use prioritized columns, card lists, or deliberate horizontal scroll
- Approval actions may use a sticky bottom action bar
- Workflow diagram is read-only
- Workflow editing is disabled

## 9. Buttons and Actions

### 9.1 Primary

Used for the main page action:

```css
background: #1d4ed8;
color: #ffffff;
border: 1px solid #1d4ed8;
```

Examples: Create, Submit for Approval, Save Changes, Generate PDF.

### 9.2 Secondary

```css
background: #ffffff;
color: #334155;
border: 1px solid #cbd5e1;
```

Examples: Save Draft, Preview, Download, View Details.

### 9.3 Semantic Workflow Actions

| Action | Visual treatment |
| --- | --- |
| Approve | Green filled button |
| Acknowledge | Primary blue button |
| Return for Revision | Orange filled or strongly outlined button |
| Reject | Red destructive button |
| Cancel or Void | Red destructive treatment with confirmation |

Rules:

- A page normally has no more than one or two primary-looking actions.
- Return and Reject never share the same color, icon, or confirmation message.
- Destructive actions require confirmation.
- Critical actions are not all hidden inside a three-dot menu.
- Icon-only controls are used only for familiar secondary actions and have accessible names.

## 10. Forms

### 10.1 Input

```css
height: 40px;
border: 1px solid #cbd5e1;
border-radius: 8px;
background: #ffffff;
color: #0f172a;
```

Focus:

```css
border-color: #3b82f6;
box-shadow: 0 0 0 3px rgb(59 130 246 / 15%);
```

Error:

```css
border-color: #dc2626;
box-shadow: 0 0 0 3px rgb(220 38 38 / 10%);
```

### 10.2 Form Layout

- Labels appear above controls
- Required fields use a visible marker and validation message
- Related fields may use two columns on desktop
- Descriptions, reasons, and notes use full width
- Validation appears next to the affected field and in a summary when submission fails
- Read-only fields remain selectable and legible
- Disabled fields are visually muted and not interactive

### 10.3 Money and Quantity

- Numeric values align right
- Calculation result is read-only and visibly separated from editable inputs
- Currency is always shown
- Thousand separators and decimal precision are consistent
- Never rely on placeholder text as the only label

## 11. Cards

Standard card:

```css
background: #ffffff;
border: 1px solid #e2e8f0;
border-radius: 12px;
box-shadow: var(--shadow-xs);
```

Cards are used for grouped information, not as a wrapper around every sentence. Avoid nested cards and oversized single-metric cards.

Card headers place title and context on the left and a secondary action on the right. Tables inside cards may use a divider below the header.

## 12. Data Tables

Tables are a primary ProcureHub component and must support:

- Search
- Typed filters
- Sort
- Pagination
- Column visibility
- Sticky header where appropriate
- Row action menu
- Empty, loading, and error state
- Optional bulk actions for safe workflows

Standard document columns:

```text
Document Number
Document Type
Company and Branch
Owner Department
Requester
Document Status
Current Step or Approver
Amount
Updated At
Actions
```

Styling:

- Header background: Slate 50
- Row background: White
- Row hover: Slate 50
- Dividers: Slate 200
- Row height: 48-56 px
- Numeric values: right aligned
- Document number: primary-colored link with medium weight
- Status: semantic badge with label

Table density supports two explicit modes:

- `comfortable`: 48-56 px rows; default for general users
- `compact`: 40-44 px rows; available for purchasing and administration users handling dense operational lists

Density changes spacing only. It must not hide data, reduce touch targets below accessibility requirements, or change permission behavior.

Filters show active-filter chips and provide one clear Reset action. Search, filter, and sort state should be reflected in the URL when practical.

## 13. Status and Badges

### 13.1 Document Lifecycle

```text
Draft
Pending Approval
Returned for Revision
Approved
Rejected
Cancelled
Closed
```

### 13.2 Approval Task

```text
Waiting
Pending
Approved
Acknowledged
Returned
Rejected
Skipped
Cancelled
Resolution Error
```

SLA is displayed as a separate task indicator such as `Due Soon` or `Overdue`. It does not replace the primary Approval Task status; for example, a task can be `Pending` and `Overdue` at the same time.

### 13.3 Template and File

```text
Template lifecycle: Draft, Active, Archived
Template validation: Pending, Valid, Invalid
Template preview: Not Generated, Generating, Succeeded, Failed
Generated file: Generating, Generated, Failed, Superseded, Voided
```

Badges use a light background, dark semantic text, a thin border, and optional icon. Dense tables avoid fully saturated badge backgrounds.

## 14. Dashboard

The dashboard starts with the user's responsibilities rather than organization-wide charts.

Recommended order:

1. My Approval Tasks
2. Returned Documents requiring correction
3. Tasks due soon or overdue
4. Recent documents
5. Failed PDF generation or workflow resolution errors for authorized users
6. Status and cycle-time summaries

Recommended KPI cards:

- Pending Approvals
- Returned for Revision
- Due Soon
- Overdue
- Failed PDF Generation
- Unread Notifications

Every KPI links to a filtered actionable list.

## 15. Document Detail Page

Structure:

```text
Page Header
├── Document number and type
├── Document status
├── Company, branch, and owner department
├── Primary action
└── Secondary action menu

Summary
Document Information
Item Table
Attachments
Approval Progress
Timeline
Generated PDFs
Revision History
```

State-specific actions:

| State | Primary actions |
| --- | --- |
| Draft | Edit, Save Draft, Submit for Approval |
| Pending Approval | View, View Approval Progress |
| Returned | Edit, View Return Reason, Resubmit |
| Approved | Generate or Download Final PDF |
| Rejected/Cancelled | View history; duplicate if authorized |

Returned reason and revision impact must appear near the top, not buried in the timeline.

## 16. Workflow Administration UI

Workflow editing follows the approved constrained sequential design.

### 16.1 Workflow List

Group by company, branch, and department. Show:

```text
Workflow Name
Document Type
Scope
Source: Inherited or Override
Version
Status
Step Count
Health
Last Activated
Created By
Actions
```

Health states:

```text
Healthy
Missing Approver
Organization Changed
Resolution Error
Requires Validation
```

### 16.2 Editor Layout

```text
Header
  Name, scope, version, status, Save, Validate, Simulate, Activate

Left Palette
  Start, Approval, Review, Acknowledge, Verify, End

Center Canvas
  Ordered nodes with automatic connections and add-step controls

Right Properties
  Step settings, resolver, conditional rule, return strategy, SLA

Bottom Panel
  Validation errors, warnings, and simulation result
```

The editor supports add-between, reorder, duplicate, disable, delete, undo, redo, autosave, and auto-layout. Users cannot draw arbitrary connections in version 1.

### 16.3 Workflow Nodes

Every node contains:

- Step icon and type
- Step name
- Resolver summary
- Conditional-step badge when present
- SLA badge when present
- Validation state

Visual state:

| State | Treatment |
| --- | --- |
| Selected | Primary border and soft blue background |
| Valid | Neutral border |
| Warning | Amber indicator |
| Error | Red border and error icon |
| Disabled | Muted with Disabled label |

Explicit decision diamonds are documented as a future capability and are not shown in version 1.

### 16.4 Conditional Step Builder

Administrators use a form such as:

```text
Run this step when
[Amount] [is greater than or equal to] [100,000]
```

JSON is never exposed. If the condition is false at runtime, the step appears as Skipped with a human-readable reason.

### 16.5 Validation and Simulation

Errors block activation. Warnings require review but do not always block. Clicking an issue focuses the affected node.

Simulation shows:

- Resolved workflow and fallback source
- Company, branch, and department context
- Applicable and skipped steps
- Current resolved approver
- Missing organization assignments
- SLA examples
- Final ordered path

### 16.6 Activation

Activation UI must show:

- Validation result
- Last successful simulation
- Version diff
- Scope affected
- Parent fallback behavior
- Running instances that remain on the old version
- Required activation note

## 17. Runtime Approval UI

### 17.1 Approval Stepper

Runtime users see a read-only ordered stepper.

| Status | Visual |
| --- | --- |
| Completed | Green check |
| Current | Blue ring and highlighted card |
| Waiting | Slate gray |
| Skipped | Muted dash with reason |
| Returned | Orange return icon |
| Rejected | Red X |
| Reassigned | User-switch indicator |
| Delegated | Acting-user indicator |
| Overdue | Red clock and label |
| Resolution Error | Red warning with repair owner |

Do not use document status names such as `Pending IT Head`. The document remains Pending Approval while current-task details explain the exact step.

### 17.2 Approval Task Card

Shows:

- Document number and type
- Requester and owner department
- Company and branch
- Amount and currency
- Submitted time
- SLA due time
- Current required action
- Latest relevant comment

### 17.3 Return Dialog

- Orange warning treatment
- Reason is mandatory
- Explain that the document becomes editable
- Show configured resubmit strategy
- Explain that material changes can force a restart

### 17.4 Reject Dialog

- Red destructive treatment
- Reason is mandatory
- Explicitly state that rejection ends the workflow
- Never reuse the Return dialog text

## 18. Timeline and Revision UI

Timeline combines:

- Document lifecycle
- Approval actions
- Return and resubmit
- Revision creation
- Approver reassignment
- Delegation
- SLA reminders
- Attachment changes
- PDF generation
- Cancellation or voiding

Newest events appear first in the activity timeline. Approval step order remains a separate stepper.

Each event includes icon, title, explanation, actor, date and time, and optional structured metadata.

Revision comparison highlights:

- Added values
- Removed values
- Changed values
- Material approval impact
- Resulting restart or resume decision

## 19. Document Template Management UI

Main sections:

```text
Registered Contracts
Tag Sheet
Template Upload
Validation Result
Preview PDF
Version History
Active Template
```

Tag Sheet rows show label, copyable tag, path, group, required state, helper state, description, and Copy action.

Validation separates:

```text
Detected Tags
Known Tags
Unknown Tags
Missing Required Tags
Warnings
Preview Result
```

Unknown tags use a readable suggestion. Activation remains disabled until validation, contract compatibility, and preview requirements pass.

Template state is displayed in three separate fields:

```text
Lifecycle: Draft | Active | Archived
Validation: Pending | Valid | Invalid
Preview: Not Generated | Generating | Succeeded | Failed
```

Do not present `Invalid` as a lifecycle status. An invalid template remains a Draft whose validation failed.

## 20. Notification Center

Topbar notification center shows unread count and recent notifications. Full notification page supports:

- Unread and all filters
- Notification type filter
- Mark one or all as read
- Link to the relevant authorized document or task
- Failed-delivery administration view for System Admin

Notification labels distinguish:

```text
Action Required
Reminder
Returned
Approved
Rejected
System Error
```

In-app and email presentation may differ, but they originate from the same event and use consistent terminology.

## 21. Empty, Loading, Error, and Success States

### Empty

Include a simple icon, title, explanation, and one relevant action. Do not show decorative illustrations that dominate operational pages.

### Loading

- Skeleton for tables, cards, and page content
- Spinner for short button actions
- Progress state for document generation or upload
- Never render a blank page during normal loading

### Error

Errors explain:

- What failed
- What remains safe
- What the user can do next
- Error or correlation ID for server failures

Never display a stack trace to end users.

### Success

Use a toast for confirmation and update the affected page state immediately. Toast alone is not sufficient for workflow status changes.

## 22. Modals, Drawers, and Pages

Use a modal for:

- Confirmation
- Short required reason
- Small self-contained task

Use a drawer for:

- Timeline preview
- Approval task details
- Filter controls on smaller screens
- Supporting data without leaving a list

Use a full page for:

- Creating or editing business documents
- Workflow editing
- Template validation and preview
- Organization and numbering configuration

Modal focus must be trapped, Escape closes when safe, and the invoking control regains focus.

## 23. Icons

Use one outline icon family consistently. Lucide Icons is the preferred family if it matches the selected frontend stack.

Standard sizes:

```text
16 px: dense table and button icons
18-20 px: navigation and common controls
24 px: page-level status and empty state
```

Important actions include labels. Filled and outline icon styles are not mixed without a documented semantic reason.

## 24. Motion

```css
:root {
  --duration-fast: 120ms;
  --duration-normal: 180ms;
  --duration-slow: 240ms;
}
```

Motion is limited to hover, dropdown, drawer, modal, sidebar collapse, and small status transitions. Avoid bounce, strong springs, long animations, and large card movement.

Respect `prefers-reduced-motion`.

## 25. Accessibility

Minimum target: WCAG 2.2 AA.

Requirements:

- Normal text contrast of at least 4.5:1
- Large text contrast of at least 3:1
- Full keyboard navigation
- Visible focus indication
- Semantic HTML
- Accessible names for icon-only controls
- Error messages programmatically associated with fields
- Status communicated by text, not only color
- Correct heading order
- Modal focus management
- Table headers use proper header semantics
- Buttons use button elements
- Touch target size appropriate for mobile approval actions
- Reduced-motion support

## 26. Localization and Formatting

The UI supports Thai and English content architecture even if the first release is Thai-first.

Rules:

- Store timestamps in UTC and display in the relevant company/user timezone
- Default timezone: Asia/Bangkok
- Avoid ambiguous numeric dates
- Support Buddhist Era and Gregorian presentation through locale configuration
- Always display the currency carried by the document; never infer currency only from company or branch
- Use one currency format throughout a context
- Align monetary values right
- Keep document numbers unmodified and easy to copy

Examples:

```text
13 ก.ค. 2569 เวลา 10:30 น.
13 Jul 2026, 10:30
125,000.00 THB
3,500.00 USD
PR-HQ-2026-0001
```

Dark mode is out of scope for version 1. Do not create an alternate dark theme until it has a separate approved design specification.

## 27. React Email Visual Standard

React Email templates follow the ProcureHub visual identity but use email-safe simplicity.

Guidelines:

- One central ProcureHub sender identity
- Company logo and company name inside the content
- Maximum content width around 600 px
- White content surface on a light slate background
- Primary blue call-to-action that opens ProcureHub
- Short inbox preview text
- Clear document number, status, action, and due time
- Footer with company support contact and ProcureHub identity
- No direct Approve or Reject action in version 1
- No sensitive PDF attachment by default
- HTML and plain-text alternatives
- Test with Microsoft Outlook and Gmail

Email branding accepts safe typed values only. Arbitrary HTML and CSS from the database are prohibited.

## 28. Design Tokens as Source of Truth

The implementation must map these tokens into the existing styling system. If Tailwind is selected, map CSS variables into the Tailwind theme or utilities instead of duplicating hex values throughout components.

Required token groups:

```text
Color
Typography
Spacing
Radius
Shadow
Motion
Layout size
Semantic status
```

Do not scatter hard-coded brand or status colors across page components.

## 29. Reusable Component Inventory

Foundation:

```text
AppShell
Sidebar
TopNavigation
PageHeader
Breadcrumbs
CompanyBranchSelector
```

Data display:

```text
MetricCard
StatusBadge
DocumentTypeBadge
SlaBadge
DataTable
TableToolbar
Pagination
ColumnSelector
DocumentSummary
```

Forms:

```text
FormField
MoneyInput
DatePicker
CompanySelector
BranchSelector
DepartmentSelector
UserSelector
AttachmentUploader
```

Workflow:

```text
WorkflowEditor
WorkflowNode
WorkflowPropertiesPanel
WorkflowValidationPanel
WorkflowSimulator
ApprovalStepper
ApprovalTaskCard
ApprovalActionBar
ReturnRevisionDialog
RejectDialog
RevisionDiff
DocumentTimeline
```

Document platform:

```text
TagSheet
TemplateUploader
TemplateValidationPanel
TemplatePreviewViewer
TemplateVersionHistory
GeneratedPdfList
AttachmentList
```

Feedback:

```text
EmptyState
LoadingSkeleton
ErrorState
ConfirmDialog
Toast
NotificationCenter
```

Reusable components must support loading, disabled, error, empty, and permission states where applicable.

## 30. Anti-Patterns

Do not:

- Use several competing primary colors
- Use bright gradients, glassmorphism, neon, or heavy shadow
- Turn every section into a large card
- Hide critical actions in a three-dot menu
- Use color as the only status signal
- Treat Reject and Return as the same action
- Encode the current approver into the document status name
- Use unrestricted JSON editors for workflow conditions
- Allow mobile workflow-diagram editing
- Use multiple font families across application screens
- Use radius larger than 16 px throughout the product
- Build tables without search, filtering, loading, and empty states
- Show raw backend errors or stack traces
- Let frontend data become the source of truth for document generation
- Add animation that delays operational work
- Include fake production metrics or decorative charts without meaning

## 31. Screen Specification Process

This design system is intentionally feature-neutral. Detailed screens are documented when a module enters design.

Recommended future files:

```text
docs/design/screens/WORKFLOW_ADMIN_SCREENS.md
docs/design/screens/DOCUMENT_TEMPLATE_SCREENS.md
docs/design/screens/PR_SCREENS.md
docs/design/screens/PO_SCREENS.md
```

Each screen specification should define:

- User goal and permission
- Entry points
- Layout and information hierarchy
- Fields and validation
- Actions by lifecycle status
- Loading, empty, error, and success states
- Responsive behavior
- Accessibility behavior
- Analytics or audit events

## 32. Design Review Checklist

Before accepting a screen:

- Is the company, branch, and department context clear?
- Are document status and current task shown separately?
- Is the main action obvious?
- Are Return and Reject clearly different?
- Can the user understand why an action is disabled?
- Are loading, empty, error, and permission states designed?
- Does the table remain usable on smaller screens?
- Is keyboard navigation possible?
- Are text and controls WCAG AA compliant?
- Does the screen use shared components and tokens?
- Are date, money, and document numbers formatted consistently?
- Does the mobile view preserve essential approval actions?

## 33. Definition of Done

UI implementation conforms to this system when:

- ProcureHub uses the approved Navy + Blue + Slate direction.
- Design tokens are centralized and reused.
- App shell is responsive and permission-aware.
- Company and branch context is visible.
- Tables, forms, cards, and dialogs share consistent states.
- Document and approval statuses are distinct and accessible.
- Workflow editing is constrained, validated, and desktop-first.
- Runtime approval progress is understandable without reading raw history.
- Return and Reject have distinct language and visual treatment.
- Template management clearly communicates tag validation and version status.
- In-app notification and React Email presentation use consistent terminology.
- Thai and English typography render correctly.
- No critical accessibility issue remains.
- Feature-specific screen specs document intentional exceptions.
