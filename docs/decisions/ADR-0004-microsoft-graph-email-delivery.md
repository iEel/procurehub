# ADR-0004: Use Microsoft Graph for Central Microsoft 365 Email Delivery

**Status:** Accepted<br>
**Decision date:** 2026-07-15<br>

## Context

The organization uses Microsoft 365. ProcureHub needs server-side transactional email with centralized sender governance, auditable attempts, and no user mailbox dependency.

## Decision

Use Microsoft Graph with app-only authentication to send from one configured central Microsoft 365 mailbox.

- Recipient addresses are resolved from authorized backend user/organization data.
- Graph credentials and mailbox configuration remain server-side.
- Application permission is limited and scoped operationally as tightly as the tenant supports.
- Delivery is asynchronous through notification jobs.
- Graph request acceptance does not claim final mailbox delivery.
- PDF attachment is not enabled by default.

## Consequences

- Tenant administration must provision app identity, permission consent, mailbox policy, credential rotation, and message tracing/support procedures.
- Throttling, timeout, unauthorized, and server errors require bounded retry classification.
- Notification failure never rolls back the completed approval or lifecycle transaction.
- Provider request IDs and safe attempt metadata are retained.

## Alternatives Considered

- SMTP authenticated mailbox: simpler in some environments but weaker alignment with the selected Microsoft 365 application-permission model.
- Delegated Graph authentication: depends on an interactive user and is unsuitable for background jobs.
- Third-party transactional email: unnecessary while Microsoft 365 is the approved organizational provider.

## Related Documents

- [Workflow, SLA, and Notification Design](../superpowers/specs/2026-07-13-procurehub-workflow-design.md)
- [File and Document Security](../security/FILE_AND_DOCUMENT_SECURITY.md)
- [ADR-0003: React Email](ADR-0003-react-email.md)
