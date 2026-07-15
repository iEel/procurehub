# ADR-0003: Use Code-Owned React Email Templates

**Status:** Accepted<br>
**Decision date:** 2026-07-15<br>

## Context

ProcureHub needs consistent Thai/English transactional email for approval tasks, returns, reminders, resolution errors, and document events. Email must share terminology and visual identity with the application while remaining secure and testable.

## Decision

Use React Email components owned in source code for transactional email presentation.

- Templates are versioned and reviewed with application code.
- Rendering produces HTML and plain-text alternatives.
- Inputs use typed, server-created notification payloads.
- Administrators cannot upload or edit arbitrary React code.
- Business event creation is separated from presentation and provider delivery.

## Consequences

- Email components can be previewed and tested with representative Thai and English cases.
- Design changes follow the ProcureHub email visual standard.
- Content changes require the normal development/review lifecycle.
- Notification records store template code/version and payload snapshot, not executable code.

## Alternatives Considered

- Administrator-editable HTML: more flexible but creates security, consistency, and support risk.
- Provider-owned templates: couples content/version history to the provider and weakens local testing.
- Plain-text-only email: accessible but insufficient for the selected modern enterprise presentation.

## Related Documents

- [Workflow, SLA, and Notification Design](../superpowers/specs/2026-07-13-procurehub-workflow-design.md)
- [ProcureHub Design System](../design/DESIGN_SYSTEM.md)
