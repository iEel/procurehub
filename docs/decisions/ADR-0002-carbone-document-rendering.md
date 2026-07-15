# ADR-0002: Use Carbone On-Premise for DOCX Rendering and PDF Conversion

**Status:** Accepted<br>
**Decision date:** 2026-07-15<br>

## Context

Company document forms must match approved Word layouts. ProcureHub uses contract-first tags and needs to generate PDF while preserving company/branch headers and footers. The selected Carbone edition does not provide the required dynamic-image feature.

## Decision

Use Carbone On-Premise as a render/convert adapter only.

Node.js owns:

- document contracts and allowed tags
- template upload, validation, versioning, preview, and activation
- authoritative JSON mapping and schema validation
- optional header/footer image patching inside a working DOCX package
- final PDF storage, hashes, snapshots, render history, and authorization

Carbone receives a prepared DOCX buffer plus validated JSON, renders the template, converts it to PDF, and returns the result.

Do not depend on Carbone dynamic-image features. Use one master form per document type in version 1 and patch versioned company/branch image assets before render.

## Consequences

- Word controls layout while ProcureHub controls data and assets.
- Carbone remains replaceable behind a renderer port.
- The Carbone endpoint must be private, fixed server-side, timeout-bounded, and response-validated.
- DOCX preparation and package security become first-class Node.js responsibilities.
- Old template, asset, and PDF versions must be retained for audit.

## Alternatives Considered

- Separate DOCX per company/branch: simpler rendering but duplicates templates and makes form maintenance expensive.
- Carbone Enterprise dynamic pictures: not selected because the approved edition and technique use Node.js patching.
- Build PDF layouts directly in code: would make exact Word-form matching and business-owned template maintenance harder.

## Related Documents

- [Document Engine Design](../superpowers/specs/2026-07-15-procurehub-document-engine-design.md)
- [File and Document Security](../security/FILE_AND_DOCUMENT_SECURITY.md)
