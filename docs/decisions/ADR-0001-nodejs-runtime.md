# ADR-0001: Use Node.js as the Application Runtime

**Status:** Accepted<br>
**Decision date:** 2026-07-15<br>

## Context

ProcureHub requires a server-owned document engine, workflow orchestration, file processing, background jobs, React Email rendering, and integrations with Carbone and Microsoft Graph. The project was explicitly defined as a Node.js project.

## Decision

Use a supported Node.js LTS release with TypeScript for application, worker, and integration code.

This ADR selects the runtime and language direction only. It does not select the frontend framework, HTTP framework, ORM, database, queue, storage provider, authentication provider, or deployment platform.

## Consequences

- Contracts, domain interfaces, and API types can share TypeScript definitions where boundaries permit.
- React Email can run in the same language ecosystem.
- DOCX, ZIP, hashing, HTTP, queue, and storage work must use bounded, security-reviewed libraries.
- CPU/memory-heavy or untrusted document conversion remains outside the Node.js process in Carbone/LibreOffice.
- The implementation plan must pin Node.js and package-manager versions and define upgrade policy.

## Alternatives Considered

- .NET or Java backend: technically suitable but conflicts with the approved Node.js project direction.
- Multiple runtime languages from the start: increases deployment and operational complexity without a version 1 need.

## Related Documents

- [System Architecture](../architecture/SYSTEM_ARCHITECTURE.md)
- [Document Engine Design](../superpowers/specs/2026-07-15-procurehub-document-engine-design.md)
