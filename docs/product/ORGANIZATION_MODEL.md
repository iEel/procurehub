# ProcureHub Organization Model

**Status:** Ready for user review<br>
**Date:** 2026-07-15<br>
**Product:** ProcureHub<br>
Scope: Multi-company, multi-branch organization structure, membership, position assignment, and approver resolution context

## 1. Purpose

ProcureHub uses organization data to isolate access, select document and workflow configuration, and resolve real approvers. This document defines the canonical hierarchy and effective-dated assignment rules that all modules must share.

The model must support a company whose Head Office IT department follows a different approval flow and uses different approvers from the Head Office HR department. Department names and people are never hard-coded in the workflow engine.

## 2. Canonical Hierarchy

```text
Company
├── Branch
├── Division
│   └── Department
└── Users through effective memberships and position assignments
```

- A `Company` is the top-level business and authorization boundary.
- A `Branch` belongs to exactly one company.
- A `Division` belongs to exactly one company.
- A `Department` belongs to exactly one division and therefore one company.
- A department can operate in one or more branches through `OrganizationUnitPresence`.
- Head Office is represented as a normal branch with an explicit branch code and `isHeadOffice = true`.

Do not model Head Office as a null branch. Explicit branch identity avoids ambiguous numbering, document assets, permissions, and workflow scope.

## 3. Company and Branch

### Company

Required concepts:

```text
id
code
legalNameTh
legalNameEn nullable
taxId
defaultCurrency
defaultTimezone
active
```

Company code is stable after first use in a document number or external integration.

### Branch

Required concepts:

```text
id
companyId
code
nameTh
nameEn nullable
taxBranchCode nullable
isHeadOffice
address and contact snapshot source
active
```

Branch code is unique within a company and stable after first use.

## 4. Division and Department

### Division

A division is a company-level management unit such as Information Technology, Finance, or Operations.

### Department

A department is the operational owner of a document and belongs to one division. Examples may include IT Infrastructure or Human Resources Operations, but production code uses identifiers and canonical position codes rather than display names.

### OrganizationUnitPresence

This association declares where a division or department operates:

```text
id
companyId
branchId
organizationUnitId
activeFrom
activeTo nullable
```

It enables both:

- company-wide departments that operate in several branches
- branch-local departments that operate only in one branch

## 5. User Membership

A user can work across more than one company, branch, or department. Access is granted through effective-dated memberships rather than a single company or department field on the user.

```text
OrganizationMembership
- id
- userId
- companyId
- branchId nullable
- divisionId nullable
- departmentId nullable
- membershipType
- effectiveFrom
- effectiveTo nullable
- active
```

Rules:

- A membership must be internally consistent with the company hierarchy.
- An inactive or out-of-period membership grants no access.
- Cross-company access is explicit; it is never inferred from an email domain or global role.
- The user selects an authorized company and branch context in the UI.
- Server authorization validates the selected context on every request.

## 6. Position and Assignment

`PositionDefinition` describes a stable organizational function. `PositionAssignment` assigns a user to that function for an effective period and scope.

Example canonical position codes:

```text
department_head
division_head
purchasing_head
accounting_head
```

```text
PositionAssignment
- id
- positionCode
- userId
- companyId
- branchId nullable
- divisionId nullable
- departmentId nullable
- effectiveFrom
- effectiveTo nullable
- assignmentType: primary | acting
- source
- version
```

Rules:

- The database prevents overlapping primary assignments for the same position and exact scope.
- Acting assignments are explicit and effective-dated.
- Approval delegation is separate from organization position assignment.
- Changes are append-only or versioned; historical effective periods remain auditable.

## 7. Workflow Context

Every workflow submission receives an immutable context snapshot containing at least:

```text
documentType
refId
companyId
branchId
ownerDivisionId
ownerDepartmentId
requesterUserId
amount
currency
category nullable
submittedAt
```

For a PO created from a PR, owner company, branch, division, and department come from the source PR or approved business source. They do not change to Purchasing merely because Purchasing creates the PO.

## 8. Workflow Scope Resolution

The active definition is selected in this order:

1. Company + Branch + Department
2. Company + Department
3. Company + Branch
4. Company Default
5. Global Default

The resolver selects one complete workflow. It never merges steps from different scopes.

This means Company A, Head Office, IT can have a different definition and different approvers from Company A, Head Office, HR.

## 9. Approver Resolution

Resolver examples:

```text
requester_department_head
requester_division_head
owner_department_head
owner_division_head
purchasing_head
accounting_head
fixed_role
fixed_user
```

Resolution order for an organization position:

1. Validate workflow context and effective date.
2. Find the effective primary position assignment for the required scope.
3. Apply an effective acting assignment when organization policy requires it.
4. Apply an effective approval delegation.
5. Enforce self-approval and distinct-approver policies.
6. Return the resolved user and resolution metadata.

If the required user cannot be resolved during preflight, submission is blocked and no partial workflow instance is created.

## 10. Duplicate and Self-Approval Rules

- The document creator cannot approve their own task when `preventSelfApproval` is enabled.
- Workflow steps that must use different people share a `distinctApproverGroup`.
- If two steps in that group resolve to the same user, preflight fails with an organization-configuration error.
- Automatic skipping or double approval by the same person is not the default.
- A future workflow may allow the same user only through an explicit, audited policy exception.

The standard department-head and division-head approval flow assumes different people.

## 11. Effective Changes and Running Workflows

The approved behavior is:

- `waiting` steps keep resolver intent and resolve when they become active.
- `pending` tasks are reconciled when an effective Organization Assignment or Delegation changes.
- A pending task may be reassigned automatically to the newly resolved user.
- Reassignment revokes the old user's task access, cancels obsolete reminders, notifies the new user, and writes an audit event.
- Approval and reassignment use transaction or optimistic-lock protection to prevent race conditions.
- Completed tasks, acted-by identity, comments, snapshots, and approval history never change.

## 12. Deactivation Rules

- A company, branch, division, department, user membership, or position assignment referenced by active documents cannot be physically deleted.
- Deactivation requires an effective date.
- Deactivation is blocked when it would leave a required current approver unresolved unless a replacement is effective at the same time.
- Historical documents retain identifiers and display-name snapshots.

## 13. Authorization Boundary

Every company-owned record carries `companyId`. Branch and department scope are added where relevant.

Repository and service queries must apply authorized scope server-side. A company or branch selector in the frontend is context, not proof of authorization.

System Administrator permissions for cross-company configuration are explicit and fully audited.

## 14. Validation and Test Scenarios

Minimum scenarios:

- User with one company and one branch
- User with several authorized company and branch memberships
- Department operating in one branch and in multiple branches
- IT and HR resolving different workflows and approvers in the same branch
- Missing department head blocks submission
- Duplicate department-head and division-head user violates distinct-approver policy
- Effective organization change reassigns a pending task with audit
- Waiting step resolves against the latest effective assignment
- Completed task remains unchanged after assignment changes
- Cross-company organization lookup is denied

## 15. Related Documents

- [Workflow, SLA, and Notification Design](../superpowers/specs/2026-07-13-procurehub-workflow-design.md)
- [Core Data Model](../data/CORE_DATA_MODEL.md)
- [Product Terminology](TERMINOLOGY.md)
