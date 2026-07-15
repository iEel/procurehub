# ProcureHub Developer Handoff

**Status:** Approved foundation documentation<br>
**Last reviewed:** 2026-07-15<br>
**Approved by:** Product Owner<br>
Audience: Developers, reviewers, architects, designers, and AI coding agents

## 1. Purpose

This file is the starting point for anyone working on ProcureHub. It summarizes the current direction, identifies the authoritative documents, and defines how future decisions must be recorded.

This is an index and guardrail document. Detailed requirements belong under `docs/`; do not duplicate entire specifications here.

## 2. Current Project State

ProcureHub is a new project. The repository contains approved design direction and approved foundation documents. It has no application framework, database layer, or production implementation.

Approved technology directions are Node.js, Carbone On-Premise, React Email, and Microsoft Graph with Microsoft 365. Frontend framework, backend framework, database, ORM, queue, object storage, authentication, deployment, and testing choices still require an approved implementation plan or architecture decision record before they are introduced.

Current approved foundation document set. Documents marked `Approved` and ADRs marked `Accepted` govern immediately:

1. [Workflow, SLA, and Notification Design](docs/superpowers/specs/2026-07-13-procurehub-workflow-design.md)
2. [ProcureHub Design System](docs/design/DESIGN_SYSTEM.md)
3. [Document Engine Design](docs/superpowers/specs/2026-07-15-procurehub-document-engine-design.md)
4. [Document Numbering Design](docs/superpowers/specs/2026-07-15-procurehub-document-numbering-design.md)
5. [Organization Model](docs/product/ORGANIZATION_MODEL.md)
6. [Product Terminology](docs/product/TERMINOLOGY.md)
7. [System Architecture](docs/architecture/SYSTEM_ARCHITECTURE.md)
8. [Core Data Model](docs/data/CORE_DATA_MODEL.md)
9. [File and Document Security](docs/security/FILE_AND_DOCUMENT_SECURITY.md)

Accepted technology decisions:

1. [ADR-0001: Node.js Runtime](docs/decisions/ADR-0001-nodejs-runtime.md)
2. [ADR-0002: Carbone Document Rendering](docs/decisions/ADR-0002-carbone-document-rendering.md)
3. [ADR-0003: React Email](docs/decisions/ADR-0003-react-email.md)
4. [ADR-0004: Microsoft Graph Email Delivery](docs/decisions/ADR-0004-microsoft-graph-email-delivery.md)

Read this handoff and every relevant authoritative document before proposing architecture or writing implementation code.

## 3. Product Summary

ProcureHub is a modern, multi-company and multi-branch procurement and document-workflow platform. The product will initially support procurement processes such as PR and PO, while its document engine and approval workflow remain generic enough to support future document types.

Core product capabilities include:

- Multi-company, multi-branch, and department-aware access and configuration
- PR and PO business modules added on top of generic platform services
- Configurable, versioned approval workflows
- Department-specific workflow selection and approver resolution
- Contract-first Word template management
- DOCX preparation and rendering through Carbone On-Premise
- Generated PDF storage and history owned by Node.js
- Document lifecycle, revision, approval, and audit tracking
- In-app and Microsoft 365 email notifications
- Modern Enterprise UI using the ProcureHub Design System

## 4. Source-of-Truth Rules

Use the following precedence when requirements appear to conflict:

1. A newer, explicitly approved product or architecture decision
2. The relevant feature specification under `docs/`
3. The workflow design specification
4. The ProcureHub Design System for visual and interaction behavior
5. The approved implementation plan
6. Existing implementation and tests

Code is not allowed to silently redefine approved product behavior. When behavior must change, update or add the relevant document first, then update the implementation plan and code.

If two documents conflict and neither is clearly newer or more specific, stop and request a decision rather than guessing.

## 5. Non-Negotiable Architecture Principles

### 5.1 Generic platform core

- Do not hard-code PR, PO, quotation, receipt, invoice, or other business-document rules into the generic document or workflow engine.
- A future document type is added through a registered contract, schema, sample data, data mapper, workflow definition, and optional requirement rules.
- Business modules integrate with platform services through explicit interfaces.

### 5.2 Contract-first document engine

- Node.js is the source of truth for document contracts, allowed tags, template versions, storage, validation, generated files, and render history.
- Word template authors must copy tags from the registered Tag Sheet.
- Uploaded DOCX templates must be scanned, validated, previewed, and explicitly activated. Lifecycle, validation, and preview use separate status fields.
- Unknown tags prevent activation.
- Template and contract versions are immutable once used; older versions must never be overwritten.
- Document template activation requires maker-checker separation in version 1: the uploader cannot activate the same template version.
- The frontend sends `documentType` and `refId` for final generation, not a complete document payload.
- The backend loads and maps authoritative data from the database.
- Node.js prepares the final DOCX template buffer, including optional header/footer image patching, and validates the normalized document data.
- Carbone On-Premise only renders the prepared DOCX template with the provided JSON payload and converts the result to PDF.
- Node.js stores the final PDF; temporary Carbone output is not the system of record.
- Header and footer replacement is an optional DOCX preparation hook before rendering, not a required Carbone feature.
- Branded document assets resolve from Company + Branch to Company Default only. Missing required assets block generation; final documents never use a Global Default header or footer.

### 5.3 Separate lifecycle concepts

Keep these states independent:

- Business document lifecycle
- Approval workflow and task lifecycle
- Word template lifecycle
- Generated PDF and render-job lifecycle

For example, a document may be approved while its latest PDF generation has failed. Do not collapse those conditions into one status.

### 5.4 Auditability and immutability

- Important actions create append-only history or timeline events.
- Issued document numbers are never reused.
- Old templates, generated PDFs, approval snapshots, and revision snapshots are retained.
- Re-rendering creates a new generated-document record.
- Organization or delegation changes must not rewrite completed approval history.

## 6. Confirmed Workflow Direction

The detailed behavior is defined in the workflow specification. The current direction includes:

- Sequential approval in version 1; no parallel approval or quorum
- A constrained visual workflow editor, not unrestricted BPMN
- Central System Administrator governance
- Versioned workflow drafts, simulation, validation, activation, archive, and rollback
- One active workflow per exact scope
- Hierarchical workflow selection in this order:
  1. Company + Branch + Department
  2. Company + Department
  3. Company + Branch
  4. Company Default
  5. Global Default
- Selection of one complete workflow; steps are never merged across scopes
- Department-specific workflows and different approvers for departments such as IT and HR
- Conditional steps in version 1; explicit decision branches are a future capability
- Preflight validation that blocks submission when a required approver cannot be resolved
- Dynamic resolution of waiting-step approvers when each step becomes active
- Pending tasks are reconciled and may be reassigned automatically when an effective Organization Assignment or Delegation changes. Every reassignment is audited and protected against concurrent approval.
- Completed task assignment and approval history are immutable and are never rewritten by later organization changes.
- Approve, acknowledge, reject, and return-for-revision as distinct actions
- Reject ends the workflow; Return for Revision unlocks the document for correction
- Configurable resubmission strategy, defaulting to restart from the first step
- Revision and approval-impact tracking for material changes
- Calendar-hour or business-hour SLA reminders without automatic escalation in version 1
- Administrator-managed approval delegation with effective dates

Do not encode department names or individual approvers directly in workflow engine code. Workflow definitions specify resolver intent, and organization data resolves the actual user.

## 7. UI and UX Direction

The [ProcureHub Design System](docs/design/DESIGN_SYSTEM.md) is the visual and interaction source of truth.

Key rules:

- Modern Enterprise Dashboard style
- Navy + Blue + Slate palette
- Desktop-first, responsive, data-dense but comfortable
- IBM Plex Sans Thai as the primary typeface
- Clear company, branch, and department context
- Document status, approval task, template status, and PDF status shown separately
- Return for Revision and Reject must use different language, colors, confirmations, and consequences
- Critical workflow actions remain visible and are not all hidden in overflow menus
- Every screen includes appropriate loading, empty, error, success, permission, and accessibility states
- Visual workflow editing uses constrained nodes and typed property panels
- Target WCAG 2.2 AA accessibility

Do not introduce a conflicting component library, palette, font family, or interaction pattern without documenting the decision.

## 8. Notification and Email Direction

- Version 1 uses in-app notifications and email.
- Transactional emails are code-owned React Email components.
- Email delivery uses Microsoft Graph with application permissions and a single central Microsoft 365 mailbox.
- Email delivery is asynchronous and must not roll back a completed approval or lifecycle transaction.
- Do not allow administrator-authored arbitrary React Email code.
- Secrets, tenant identifiers, mailbox configuration, and Graph credentials remain server-side.
- Notification attempts and safe failure details must be auditable.

## 9. Documentation Structure

All material changes belong under `docs/`. Use categorized subdirectories instead of placing every Markdown file directly in the `docs` root.

Recommended structure:

```text
docs/
  product/          Product scope, terminology, and business rules
  architecture/     System architecture and integration specifications
  decisions/        Architecture Decision Records (ADRs)
  design/           Design system and screen-level UX specifications
  api/              API contracts and integration behavior
  data/             Data model, ownership, migration, and retention rules
  security/         Threat model, authorization, secrets, and audit rules
  testing/          Test strategy and acceptance scenarios
  runbooks/         Deployment, operations, recovery, and support procedures
  superpowers/
    specs/           Approved feature and subsystem design specifications
    plans/           Detailed implementation plans
```

Do not create empty directories merely to match this structure. Add a directory when its first real document is created.

## 10. Where to Record a Change

| Change | Document location |
| --- | --- |
| Product scope or business rule | `docs/product/` or the relevant feature specification |
| Workflow behavior | Workflow specification or a newer workflow feature specification |
| Architecture or technology choice | `docs/decisions/ADR-0005-example-topic.md` naming pattern |
| System/component architecture | `docs/architecture/` |
| UI tokens or global interaction rules | `docs/design/DESIGN_SYSTEM.md` |
| One screen or user journey | `docs/design/screens/PR_SCREENS.md` naming pattern |
| API request, response, and error contract | `docs/api/` |
| Data ownership, schema, retention, or migration | `docs/data/` |
| Authentication, authorization, or threat decision | `docs/security/` |
| Test strategy or acceptance suite | `docs/testing/` |
| Deployment or operational procedure | `docs/runbooks/` |
| Approved implementation sequence | `docs/superpowers/plans/` |

Small clarifications may update an existing authoritative document. A material decision that changes trade-offs, technology, security, data ownership, or public contracts should receive its own ADR or specification.

## 11. Documentation Change Protocol

When a material change is accepted:

1. Identify the authoritative document for that topic.
2. Update it or create a focused Markdown document under `docs/`.
3. Record status, date, scope, and the accepted decision.
4. Explain impact on existing behavior, data, APIs, UI, security, and migration when applicable.
5. Link related specifications or ADRs.
6. Update this handoff only when the reading order, non-negotiable rules, project phase, or authoritative-document list changes.
7. Update the implementation plan before changing code if the accepted change affects planned work.
8. Keep old decision history; supersede documents explicitly rather than silently rewriting historical intent.

Avoid using chat history as the only source of a project decision. A decision is not durable until it is represented in the repository documentation.

## 12. Required Workflow Before Implementation

Before starting a feature:

1. Read this handoff.
2. Read the relevant specifications and ADRs.
3. Confirm that requirements are approved and internally consistent.
4. Create or update a detailed implementation plan.
5. Select technology using existing decisions or write an ADR when a new material choice is required.
6. Define acceptance tests and failure behavior.
7. Implement in small, independently testable modules.
8. Run proportional automated and manual verification.
9. Update documentation when implementation exposes a necessary design change.

Do not use implementation as a substitute for unresolved product or architecture decisions.

## 13. Initial Delivery Order

The next planning phase should decompose implementation into bounded deliveries. The expected high-level order is:

1. Repository and application foundation
2. Identity, organization, role, and permission foundation
3. Design tokens and application shell
4. Storage, attachment, and file-security foundation
5. Generic document contract, template, DOCX preparation, and Carbone foundation
6. Document numbering and sequence foundation
7. Document lifecycle, generated-document history, and timeline
8. Workflow definition, resolution, validation, and visual editor
9. Workflow runtime, tasks, return/revision, delegation, and SLA reminders
10. In-app notification, React Email, and Microsoft Graph delivery
11. PR business module
12. PO business module
13. Receiving, invoice handoff, reporting, and later procurement capabilities

An approved implementation plan may split or refine these deliveries, but it must preserve the dependency order and generic-core boundaries.

## 14. Review and Verification Expectations

Every implemented slice must include, as applicable:

- Unit tests for pure rules and validators
- Integration tests for repositories, storage, rendering, workflow transitions, and external adapters
- Authorization and cross-company data-isolation tests
- Concurrency and idempotency tests for numbering, approval actions, activation, and job processing
- Failure-path tests for Carbone, Microsoft Graph, storage, queue, and database errors
- UI loading, empty, error, permission, responsive, keyboard, and accessibility checks
- Audit and snapshot verification
- Documentation updates when behavior or interfaces changed

Do not claim completion based only on a successful happy-path demonstration.

## 15. Explicit Do-Not List

- Do not start implementation before an approved implementation plan exists.
- Do not invent a framework, database, or infrastructure choice without documenting it.
- Do not hard-code business-document logic into generic platform services.
- Do not let the frontend become the source of truth for final render data.
- Do not use Carbone temporary files as final storage.
- Do not overwrite active or historical template and PDF versions.
- Do not merge workflow steps from different scope levels.
- Do not treat Reject and Return for Revision as the same action.
- Do not allow unresolved approvers to enter a running workflow.
- Do not silently change completed approval history.
- Do not duplicate full specifications in this handoff.
- Do not rely on chat history instead of repository documentation.

## 16. Handoff Checklist

Before handing work to another developer or agent, state:

- What objective was completed
- Which files changed
- Which specification and plan governed the work
- Which tests and checks were run, with results
- Which decisions changed and where they were documented
- Known limitations, risks, migrations, or follow-up work
- The exact next safe step

If work is incomplete, describe the current state precisely. Do not label unverified or partially implemented work as complete.
