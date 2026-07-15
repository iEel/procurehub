# ProcureHub Workflow, SLA, and Notification Design

**Status:** Approved<br>
**Date:** 2026-07-13<br>
**Approved date:** 2026-07-15<br>
**Approved by:** Product Owner<br>
**Product:** ProcureHub<br>
Scope: Generic document workflow foundation; no PR/PO-specific business logic is implemented by this specification.

## 1. Purpose

ProcureHub needs a configurable approval workflow engine that supports multiple companies, branches, and departments. Workflow administrators must be able to create and edit an approval diagram without writing code. Business users must see the current approval progress, act on assigned tasks, and understand why a document is waiting, returned, rejected, or overdue.

The engine must remain generic. PR, PO, and future document types integrate through document contracts, workflow context, approver resolvers, and lifecycle adapters rather than hard-coded workflow logic.

## 2. Confirmed Design Decisions

The following decisions were explicitly accepted:

- Approval is sequential in version 1. Parallel approval is out of scope.
- Workflow editing uses a constrained visual diagram rather than unrestricted BPMN.
- Only central System Administrators can create, edit, simulate, activate, archive, and roll back workflow definitions.
- The administrator who created a draft may activate it after all mandatory checks pass.
- Exactly one active workflow is allowed per workflow scope.
- Workflow selection uses hierarchical fallback and selects one complete definition; steps are never merged across definitions.
- Department-specific workflows are supported because departments can have different step structures and different approvers.
- Version 1 uses conditional steps. Explicit Yes/No decision branches are a future capability.
- Submission is blocked when required approvers cannot be resolved during preflight validation.
- Future waiting steps resolve their actual approver when the step becomes active.
- Pending tasks are reassigned automatically when the relevant organization assignment changes.
- Return/resubmit behavior is configurable per step, with restart from the first step as the default.
- Approval impact rules force a full restart when material document fields change.
- SLA supports calendar hours or business hours per step.
- SLA sends reminders only. It does not escalate, reassign, or notify the approver's manager.
- Notification channels in version 1 are in-app notification and email.
- Email templates are code-owned React Email components.
- Email is delivered through Microsoft Graph using one central Microsoft 365 mailbox.

## 3. Scope and Non-Goals

### 3.1 Included

- Versioned workflow definitions and steps
- Department-aware hierarchical workflow resolution
- Constrained visual workflow editor
- Sequential approval execution
- Conditional steps with typed conditions
- Approver resolver registry
- Preflight validation and workflow simulation
- Dynamic approver resolution at step activation
- Automatic reassignment after organization changes
- System Admin-managed approval delegation with effective dates
- Approve, acknowledge, reject, and return actions
- Return for revision, document revisions, and approval rounds
- Approval impact evaluation
- Work calendars, SLA due dates, and reminders
- In-app notifications
- React Email rendering
- Microsoft Graph app-only email delivery
- Audit history and unified document timeline events

### 3.2 Excluded from version 1

- Parallel approval or approval quorum
- Free-form BPMN editing or BPMN import/export
- Explicit Yes/No decision nodes
- Sub-workflows and reusable workflow fragments
- Automatic SLA escalation or SLA-based reassignment
- LINE OA and Microsoft Teams notifications
- Administrator-editable React Email templates
- Approval directly from an email link
- Automatic PDF attachment to notification email
- PR/PO-specific fields, calculations, or workflows in the generic core

## 4. Architecture

The workflow subsystem is separated into bounded modules:

```text
Workflow Definition
  Owns definitions, versions, visual layout, validation, simulation, and activation.

Workflow Resolution
  Selects the active definition using document context and hierarchical fallback.

Approver Resolution
  Converts resolver codes and organization context into current users, then applies active delegation.

Workflow Execution
  Owns workflow instances, sequential task activation, and task actions.

Document Lifecycle
  Owns high-level document status, edit locking, revisions, and approval rounds.

Approval Impact
  Compares revision snapshots and decides whether prior approvals remain valid.

SLA and Calendar
  Calculates due dates and schedules reminder events.

Notification
  Creates in-app notifications and transactional email jobs.

Audit and Timeline
  Records immutable workflow, task, reassignment, revision, SLA, and notification events.
```

Modules communicate through typed interfaces and domain events. Email delivery failure must never roll back an approval action or document lifecycle transaction.

## 5. Workflow Scope and Resolution

### 5.1 Scope

A workflow scope consists of:

```text
documentType
companyId nullable
branchId nullable
departmentId nullable
```

Only one active workflow may exist for the same complete scope tuple.

### 5.2 Resolution Order

For a document with company, branch, and owner department, resolution uses this exact order:

1. Company + Branch + Department
2. Company + Department
3. Company + Branch
4. Company default
5. Global default

The first active match wins. The engine never merges a parent definition into an override.

The resolved workflow ID, version, scope, and resolution explanation are saved on the workflow instance. Existing instances do not move to a newly activated definition.

### 5.3 Override Behavior

The administration UI shows whether a workflow is inherited or overridden. Creating an override clones the resolved parent definition into a new draft. Archiving an override shows which parent definition will become effective for new documents before confirmation.

## 6. Workflow Definition and Versioning

A workflow definition has immutable published versions.

Statuses:

```text
draft
active
archived
```

An active version cannot be edited. Editing clones it into the next draft version. Activating a draft archives the previous active version for the same scope. Rollback reactivates a previous immutable definition and creates a new activation audit event; running instances keep their original definition snapshot.

Activation requires:

- Successful structural validation
- No unresolved validation errors
- Successful workflow simulation
- At least one saved simulation case
- A version diff review
- An activation note
- Confirmation of the affected scope and fallback behavior

The draft creator may activate the draft because maker-checker separation is not required for version 1.

## 7. Visual Workflow Editor

The editor is desktop-first and constrained. Mobile users can view workflow progress but cannot edit diagrams.

Layout:

```text
Header
  Workflow name, scope, version, status, Save, Validate, Simulate, Activate

Left palette
  Start, Approval, Review, Acknowledge, Verify, End

Center canvas
  Ordered diagram with automatic layout and add-step controls

Right properties panel
  Selected step settings, resolver, conditions, return policy, and SLA

Bottom validation panel
  Errors, warnings, and simulation results
```

Version 1 does not allow unrestricted edge drawing. Users add a step between existing steps, drag to reorder, duplicate, disable, or delete. The system creates sequential connections automatically.

Editor capabilities:

- Autosave draft
- Undo and redo
- Drag to reorder
- Duplicate step
- Auto-layout
- Unsaved-change warning
- Inline validation markers
- Click an error to focus the affected step
- Clone workflow and create scope override
- Compare with the current active version
- Read-only runtime view highlighting completed, current, waiting, skipped, returned, rejected, reassigned, and overdue steps

## 8. Step Types and Conditions

Supported action types:

```text
review
approve
acknowledge
verify
```

Each step defines:

- Stable step ID and step code
- Name and description
- Action type
- Approver resolver code and resolver configuration
- Resolver scope policy from the approved organization-model policy registry
- Allow reject
- Allow return
- Require comment
- Return/resubmit strategy
- Optional typed condition
- SLA settings
- Active flag

The step action type controls user-facing language. Version 1 keeps `approve` as the canonical backend command and `approved` as the terminal task result for `approve`, `review`, and `verify` steps, but the UI labels the command according to the configured step type:

```text
approve -> Approve / อนุมัติ
review  -> Complete Review / ตรวจสอบเสร็จสิ้น
verify  -> Verify / ยืนยันการตรวจสอบ
```

Approval history stores both the canonical command and the configured step action type. A completed `review` or `verify` step must not be presented as an approval decision in the timeline or generated-document approval snapshot.

### 8.1 Conditional Steps

Conditions are built using a form; administrators never edit JSON directly. Allowed fields come from a workflow-context registry for the document type. Operators are type-aware.

Initial operators:

```text
==, !=, >, >=, <, <=, in, not_in
```

Version 1 supports one condition group with `all` or `any` matching. If the condition is false, the task is recorded as `skipped` with a human-readable reason and the engine advances to the next step.

### 8.2 Future Decision Branch Note

The data model uses stable node IDs and a node-type discriminator so an explicit decision node and multiple outgoing edges can be added later. The version 1 editor and execution engine must not expose or execute this capability.

## 9. Validation and Simulation

### 9.1 Structural Validation

Activation is blocked when any of these checks fail:

- Exactly one Start
- At least one End
- At least one actionable step
- No duplicate step code
- Every actionable step has a resolver
- All condition fields and operators are valid for the document type
- All enabled steps are reachable in sequential order
- No unsupported node or capability is present
- Workflow scope is valid
- No second active workflow exists for the same scope

### 9.2 Simulation

The simulator accepts a workflow context such as company, branch, department, requester, amount, category, and currency. It shows:

- The selected definition and fallback explanation
- Applicable and skipped steps
- Current resolved approver for every applicable step
- Self-approval policy results
- Missing organization assignments
- SLA due date examples
- Final ordered path

Saved cases should cover representative departments, thresholds, branches, missing approver scenarios, and self-approval scenarios.

## 10. Submission and Execution

### 10.1 Preflight

Before submission:

1. Resolve the active workflow using the approved hierarchy.
2. Validate the document lifecycle permits submission.
3. Evaluate conditional steps using the locked document data.
4. Resolve every currently applicable approver to verify organization completeness.
5. Validate self-approval and required attachment policies.
6. Block submission if any applicable step lacks an approver.
7. Create the workflow instance and first approval round transactionally.

If preflight fails, no partial workflow instance is created. The response identifies the failed step, resolver, company, branch, and department and creates a System Admin alert.

Approver resolution follows the step's explicit `resolverScopePolicy`. Workflow-definition fallback selects a complete workflow; it does not authorize an approver resolver to widen its organization scope.

### 10.2 Task Creation

All applicable and skipped steps are represented in the approval round so users can see the complete expected path:

- First applicable step: `pending`, with a resolved assignee snapshot
- Future applicable steps: `waiting`, storing resolver and workflow context
- Non-applicable steps: `skipped`, storing the condition result

### 10.3 Step Activation

When a waiting step becomes current, the engine resolves its approver again from the latest effective organization data. The preflight assignee is not treated as final for a future step.

If no approver can be resolved at activation:

- Workflow instance remains running with `resolution_error`
- Document remains `pending_approval` and locked
- No step is skipped
- System Admin receives an in-app and email alert
- `Retry Resolve Approver` becomes available after organization data is corrected

### 10.4 Organization Changes

An effective organization-assignment change triggers reconciliation for affected `pending` and `waiting` tasks:

- Completed task history never changes.
- Waiting steps retain their resolver and will resolve normally on activation.
- Pending steps are resolved immediately against the new organization state.
- If the resolved user changes, the old assignment is revoked and the new user is assigned.
- Old reminders are cancelled and new notifications are issued.
- If resolution fails, the task enters `resolution_error`.
- Reassignment and approval use transaction or optimistic-lock protection to prevent races.

Every reassignment records old user, new user, resolver, organization effective date, reason, actor/source, and timestamp.

### 10.5 Approval Delegation

Version 1 includes delegation managed by System Administrators. Approvers cannot create their own delegation from the user interface yet.

A delegation defines:

```text
fromUserId
toUserId
documentType nullable
companyId nullable
branchId nullable
departmentId nullable
effectiveFrom
effectiveTo
reason
active
```

Resolver behavior is deterministic:

1. Resolve the current position holder from organization data.
2. Find the most specific active delegation applicable to that resolved user and workflow context.
3. Assign the task to the delegate while preserving the original approver in the assignment snapshot.

Delegation rules:

- Delegation cannot target the same user.
- Delegation cycles are rejected.
- Overlapping delegations with equal specificity are rejected.
- Starting or ending an effective delegation triggers the same pending-task reconciliation used for organization changes.
- Completed tasks never change.
- The timeline shows both the original approver and acting delegate.
- Delegation does not change the workflow definition or document lifecycle status.

## 11. Task Actions and Lifecycle

Task statuses:

```text
waiting
pending
approved
acknowledged
rejected
returned
skipped
cancelled
resolution_error
```

Rules:

- Only the currently assigned user may act on a pending task.
- The canonical `approve` command completes an `approve`, `review`, or `verify` step; the UI label and audit description come from the configured step action type.
- Acknowledge completes an `acknowledge` step.
- Reject requires a reason and ends the workflow and document lifecycle as rejected.
- Return requires a reason and sends the document back for editing.
- Every action is idempotent and writes immutable approval history.
- Task completion activates the next applicable step transactionally.

Document lifecycle remains high-level:

```text
draft
pending_approval
returned_for_revision
approved
rejected
cancelled
closed
```

Current approver and current step come from ApprovalTask, not from document status names.

## 12. Return, Revision, and Approval Impact

### 12.1 Return

When a pending task is returned:

- A reason is mandatory.
- Current task becomes `returned`.
- Workflow instance becomes returned.
- Document lifecycle becomes `returned_for_revision`.
- Document edit lock is removed for the owner.
- Remaining waiting/pending tasks in the round are cancelled.
- The pre-edit document data is saved as a revision snapshot.
- The owner receives in-app and email notification.

### 12.2 Resubmit Strategy

Each step can configure:

```text
restart_from_first_step
resume_from_returned_step
```

Default is `restart_from_first_step`.

Every resubmission creates a new approval round. The document revision number increments when data changed. Previous rounds and tasks remain immutable.

### 12.3 Approval Impact Rules

Each document contract defines approval-sensitive paths and impact rules. Typical material paths include:

```text
companyId
branchId
ownerDepartmentId
amount
currency
vendorId
items
budgetCode
paymentTerms
deliveryTerms
```

The engine compares previous and current data snapshots:

- No material change: follow the step's configured strategy.
- Material change: force restart from the first step.
- Workflow scope change: resolve a new workflow and force restart from the first step.

The resubmit confirmation shows changed fields and explains why the workflow restarts or resumes.

Any generated preview or approval snapshot affected by revision is marked superseded or voided according to document policy; it is not deleted.

## 13. SLA and Work Calendars

Each step configures:

```text
slaMode: calendar_hours | business_hours
slaDuration
reminderBeforeDue
repeatReminderInterval
maximumReminderCount
```

Business-hour calendars resolve in this order:

1. Branch calendar
2. Company calendar
3. Global default calendar

Calendars define timezone, working days, daily working intervals, company holidays, branch holidays, and exceptional working dates.

The task due date and calendar version are snapshotted when the task becomes pending. Later calendar edits affect newly activated tasks only; they do not silently recalculate an existing task's due date.

SLA behavior is reminder-only:

- Notify the current approver before due time as configured.
- Mark the task overdue at the due time.
- Repeat reminders up to the configured maximum.
- Do not notify the approver's manager.
- Do not escalate or reassign because of SLA.
- Stop reminders immediately when the task completes, is returned, is rejected, or is reassigned.

## 14. Notification Architecture

Channels in version 1:

- In-app notification
- Email

Events include:

- Approval task assigned
- Approval task reassigned
- SLA reminder
- Document returned
- Document rejected
- Document approved
- Document cancelled
- PDF generated or failed
- Workflow resolution error

Notification creation uses a transactional outbox. A worker processes delivery asynchronously. Notification failure cannot roll back the business action that produced the event.

Required reliability controls:

- Stable event ID and deduplication key
- Delivery attempt history
- Bounded retry with exponential backoff
- Dead-letter state for permanent failure
- Cancellation of stale scheduled reminders
- Admin view for failed notifications

## 15. React Email Design

Email templates are code-owned TypeScript/React components. Administrators cannot edit JSX or arbitrary HTML in version 1.

Suggested structure:

```text
emails/components
  EmailLayout
  EmailHeader
  DocumentSummary
  ActionButton
  EmailFooter

emails/templates
  ApprovalTaskAssignedEmail
  ApprovalTaskReassignedEmail
  SlaReminderEmail
  DocumentReturnedEmail
  DocumentRejectedEmail
  DocumentApprovedEmail
  PdfGenerationFailedEmail
  WorkflowResolutionErrorEmail
```

An Email Template Registry maps notification event type to template key, version, subject builder, component, sample props, and supported locale.

React Email produces HTML and plain text. The Microsoft Graph adapter composes MIME with multipart alternatives so recipients have an accessible text fallback.

Company branding is passed through a safe typed context:

```text
companyName
logoUrl
primaryColor
footerText
supportContact
replyTo
locale
```

Database branding values cannot inject arbitrary HTML, JSX, or CSS.

## 16. Microsoft 365 Delivery

Email is sent from one central Microsoft 365 mailbox configured at deployment. All companies share the same From identity; company-specific branding and an optional company Reply-To appear in the email content and headers.

Authentication and authorization:

- Single-tenant Microsoft Entra application
- App-only client credentials flow
- Microsoft Graph `Mail.Send` application permission
- Administrator consent
- Exchange Online App RBAC scope restricted to the ProcureHub sender mailbox
- Certificate or federated identity credential preferred over a client secret
- Credentials stored only in a server-side secret store

The provider calls Microsoft Graph `POST /users/{configured-sender}/sendMail`.

Delivery statuses:

```text
queued
rendering
sending
accepted
retrying
failed
```

`accepted` means Microsoft Graph accepted the request; it does not claim final delivery. Message-trace integration is a future enhancement.

Retry policy:

- 429: honor Retry-After
- Transient 5xx: exponential backoff with jitter
- 401/403: configuration failure; do not retry indefinitely
- Network timeout: record an ambiguous attempt and use bounded retry
- Permanent failure: move to dead-letter state and alert System Admin

Security rules:

- Frontend cannot choose From or arbitrary recipients.
- Recipient addresses come from authorized backend user/organization data.
- Email actions link to ProcureHub and require authentication and permission checks.
- Version 1 has no direct approve/reject links.
- PDFs are not attached automatically.
- Notification logs avoid storing credentials and minimize sensitive content.

## 17. Data Model Outline

Core records:

```text
WorkflowDefinition
WorkflowStepDefinition
WorkflowDefinitionActivation
WorkflowSimulationCase
WorkflowInstance
ApprovalRound
ApprovalTask
ApprovalHistory
ApproverAssignment
ApprovalDelegation
OrganizationAssignmentChange
DocumentLifecycle
DocumentStatusHistory
DocumentRevisionHistory
ApprovalImpactResult
WorkCalendar
WorkCalendarException
ApprovalTaskSlaState
NotificationOutbox
InAppNotification
EmailDelivery
EmailDeliveryAttempt
```

Published definitions, completed tasks, approval history, document revisions, and notification attempts are append-only from normal application APIs.

## 18. API Surface Concept

Administration:

```text
GET    /api/workflow-definitions
POST   /api/workflow-definitions
GET    /api/workflow-definitions/:id
POST   /api/workflow-definitions/:id/clone
POST   /api/workflow-definitions/:id/validate
POST   /api/workflow-definitions/:id/simulate
POST   /api/workflow-definitions/:id/activate
POST   /api/workflow-definitions/:id/archive
GET    /api/workflow-definitions/:id/diff
```

Execution:

```text
POST   /api/documents/:documentType/:refId/workflow/start
GET    /api/documents/:documentType/:refId/workflow
GET    /api/documents/:documentType/:refId/tasks
GET    /api/documents/:documentType/:refId/timeline
POST   /api/approval-tasks/:taskId/action
POST   /api/approval-tasks/:taskId/retry-resolve
POST   /api/documents/:documentType/:refId/resubmit
```

Notifications:

```text
GET    /api/notifications
POST   /api/notifications/:id/read
POST   /api/notifications/read-all
GET    /api/admin/notification-deliveries
POST   /api/admin/notification-deliveries/:id/retry
```

## 19. Error Handling and Concurrency

- All task actions use idempotency keys or equivalent request deduplication.
- Approval, reassignment, and next-task activation are transactionally consistent.
- Optimistic locking prevents an old assignee from acting after reassignment.
- Invalid transitions return a readable domain error and the current task state.
- A failed notification or email never reverses a completed workflow action.
- Resolution errors pause the workflow and provide a repair action; they never skip approval.
- Structured logs include correlation ID, document type, reference ID, workflow instance ID, task ID, event ID, and delivery ID where applicable.

## 20. User Experience Requirements

### 20.1 System Administrator

- Tree/list view grouped by company, branch, and department
- Clear inherited/override source labels
- Create override by cloning the effective parent
- Visual editor with automatic sequential layout
- Typed properties and condition forms
- Validation panel linked to affected steps
- Workflow simulator with resolution explanation
- Version diff and impact summary before activation
- Workflow health indicators for missing approvers and organization changes

### 20.2 Approver

- My Approval Tasks inbox
- Clear document number, owner department, amount, requester, due time, and current action
- Read-only stepper showing complete path and skipped steps
- Separate Approve, Acknowledge, Return, and Reject actions
- Mandatory reason dialog for Return and Reject
- Visible reassignment and overdue indicators
- Mobile-friendly task review and action UI

### 20.3 Requester

- High-level document status separate from current approval task
- Current approver and due time
- Return reason prominently displayed
- Revision comparison before resubmission
- Explanation of restart versus resume behavior
- Approval rounds and unified timeline

## 21. Testing Strategy

Minimum automated coverage:

- Workflow fallback resolution for every hierarchy level
- Uniqueness of active workflow per scope
- Definition immutability and activation/version behavior
- Conditional step match and skip behavior
- Preflight blocks missing approvers without partial instance creation
- Approver re-resolution at step activation
- Automatic pending-task reassignment after organization change
- Delegation specificity, effective dates, cycle prevention, and pending-task reconciliation
- Race between reassignment and approval
- Resolution-error pause and retry
- Approve, acknowledge, reject, and return transitions
- Return requires a reason
- Approval rounds remain immutable
- Material change forces restart
- Scope change selects a new workflow and restarts
- Calendar-hour and business-hour SLA calculations
- Holiday and timezone handling
- Reminder cancellation after completion or reassignment
- Notification outbox deduplication and retry
- React Email rendering in Thai and English sample cases
- Microsoft Graph provider handling of accepted, throttled, unauthorized, timeout, and server-error responses

Manual compatibility checks:

- Workflow editor desktop usability
- Read-only progress view on mobile
- Gmail and Microsoft Outlook rendering
- Thai fonts and long document numbers
- Company branding variants
- Plain-text email output

## 22. Delivery Sequence

This design should be implemented in independently verifiable slices:

1. Organization assignments, workflow definition model, hierarchy, and versioning
2. Visual sequential editor, validation, and simulator
3. Workflow instance, conditional tasks, actions, and lifecycle integration
4. Dynamic resolution, organization-change reconciliation, and resolution repair
5. Return, revisions, approval rounds, and impact evaluation
6. Work calendars, SLA state, and reminder scheduling
7. Notification outbox and in-app notification
8. React Email templates and Microsoft Graph provider
9. Runtime stepper, timelines, administration health views, and end-to-end hardening

## 23. Acceptance Criteria

The workflow foundation is acceptable when:

- A System Admin can create, validate, simulate, and activate a sequential workflow diagram.
- Department-specific definitions resolve using the approved hierarchy.
- Only one active definition exists per exact scope.
- Existing workflow instances remain on their original definition version.
- Conditional steps are applied or skipped with an auditable reason.
- Submission fails safely if an applicable approver is missing.
- Approvers resolve again when their step starts.
- Pending tasks move automatically after an effective organization assignment change.
- Active System Admin-managed delegation is applied and audited without rewriting the original approver.
- Return, revision, resubmit strategies, approval rounds, and impact rules work as designed.
- Material changes invalidate prior approvals and restart from the first step.
- SLA can use calendar or business hours and sends reminders only.
- In-app and email notifications are asynchronous and auditable.
- React Email produces branded HTML and plain-text content.
- Microsoft Graph sends from the central mailbox using scoped app-only permission.
- No PR/PO-specific behavior is hard-coded into the generic workflow core.

## 24. Future Notes

The following capabilities are explicitly deferred, not forgotten:

- Decision branch node with Yes/No paths
- Parallel approval, approve-any, approve-all, and quorum
- Workflow subflows
- Scheduled workflow activation
- SLA escalation and SLA-driven reassignment
- Self-service approval delegation
- LINE OA and Microsoft Teams channels
- React Email visual editor
- Microsoft 365 message-trace delivery reconciliation
- Direct email actions with one-time signed tokens

## 25. Primary Technical References

- React Email introduction: https://react.email/docs/introduction
- React Email render utility: https://react.email/docs/utilities/render
- React Email CLI and preview server: https://react.email/docs/cli
- Microsoft Graph sendMail: https://learn.microsoft.com/en-us/graph/api/user-sendmail?view=graph-rest-1.0
- Microsoft Graph app-only authentication: https://learn.microsoft.com/en-us/graph/auth-v2-service
- Microsoft Graph Mail.Send permission: https://learn.microsoft.com/en-us/graph/permissions-reference
- Exchange Online App RBAC: https://learn.microsoft.com/en-us/exchange/permissions-exo/application-rbac
- Microsoft Graph throttling guidance: https://learn.microsoft.com/en-us/graph/throttling
