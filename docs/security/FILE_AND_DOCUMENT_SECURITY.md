# ProcureHub File and Document Security

**Status:** Ready for user review<br>
**Date:** 2026-07-15<br>
Scope: DOCX templates, images, attachments, generated PDFs, storage access, Carbone integration, and document audit

## 1. Security Objectives

- Prevent malicious or malformed files from affecting Node.js, storage, Carbone, LibreOffice, workers, or users.
- Prevent cross-company and unauthorized file access.
- Keep service credentials and private document data server-side.
- Preserve document integrity and an auditable chain from source data to final PDF.
- Fail closed without losing historical records.

## 2. Trust Boundaries

Treat all of the following as untrusted:

- Browser upload content and metadata
- DOCX ZIP entries and XML
- Image files and filenames
- Attachments supplied by users or vendors
- Responses returned by Carbone until validated
- Client-provided document, company, branch, and storage identifiers

Trusted configuration is loaded server-side from controlled environment or secret storage.

## 3. Upload Allowlist

Version 1 allowlist:

| Purpose | Allowed formats | Default compressed limit |
| --- | --- | --- |
| Word template | DOCX | 20 MB |
| Header/footer asset | PNG, JPEG | 10 MB |
| Business attachment | PDF, DOCX, XLSX, PNG, JPEG | 25 MB |

Limits are configuration with secure upper bounds. Increasing a limit requires operational review.

Validation uses filename extension, declared MIME, magic bytes, and internal structure. A matching extension alone is never sufficient.

## 4. DOCX ZIP Safety

Before scanning or patching a DOCX:

- Maximum compressed size: 20 MB
- Maximum total uncompressed size: 100 MB
- Maximum ZIP entries: 2,000
- Maximum per-entry uncompressed size: 25 MB
- Maximum compression ratio per entry: 100:1
- Reject encrypted ZIP entries
- Reject absolute paths, drive paths, null bytes, and `..` traversal
- Normalize separators before validation
- Reject duplicate normalized entry names
- Require `[Content_Types].xml`, `_rels/.rels`, and `word/document.xml`
- Read only allowlisted WordprocessingML parts required by the operation

Abort extraction immediately when a limit is exceeded. Do not partially continue scanning.

## 5. XML Safety

- Use parsers configured to disable DTDs and external entities.
- Do not load remote schemas, images, or relationships while scanning.
- Reject XML with forbidden external declarations when the parser cannot guarantee safe handling.
- Bound XML text size and parser depth.
- Decode entities locally without resolving external resources.
- Do not execute macros, embedded objects, scripts, or ActiveX content.

DOCM and other macro-enabled formats are not accepted as templates in version 1.

## 6. Filename and Path Safety

- Generate storage paths and internal filenames server-side.
- Store original filename only as sanitized display metadata.
- Remove control characters and normalize Unicode for display.
- Store relative paths in database records.
- Resolve and verify every local path remains under the configured storage root.
- Never concatenate user input into a filesystem path.
- Never expose internal storage paths in API responses.

## 7. Malware Scanning

Provide a `MalwareScanner` interface even when the first environment uses a no-op development adapter.

Production policy:

- New uploads enter `pending_scan` or quarantine state.
- Files are not activated, rendered, downloaded by ordinary users, or attached to email until scanning succeeds.
- A malicious or unscannable file is quarantined with a safe reason.
- Administrators can see metadata but cannot bypass quarantine without a separately audited security permission.

## 8. Image Safety

- Verify PNG/JPEG signatures and decode dimensions using a bounded parser.
- Reject decompression bombs and dimensions above configured pixel limits.
- Strip unneeded metadata when policy permits, while preserving required print quality.
- Reject SVG as a branch header/footer upload in version 1.
- Enforce the aspect-ratio and size contract required by the DOCX placeholder.

## 9. Storage and Download Authorization

- Templates, assets, attachments, previews, and generated PDFs are private objects.
- A download request authenticates the user and authorizes company, branch, document, and file purpose.
- Local-storage downloads stream through an authorized endpoint.
- Object-storage downloads use short-lived signed URLs, default five minutes.
- Signed URLs are issued only after authorization and are not stored in business records.
- Responses set safe `Content-Type`, `Content-Disposition`, `X-Content-Type-Options: nosniff`, and restrictive cache policy.
- Inline display is allowed only for explicitly safe PDF and image types.

Every final-document download records actor, document, generated-document ID, timestamp, and request correlation ID.

## 10. Multi-Company Isolation

- Every company-owned file record includes `companyId`.
- Branch and document references must belong to that company.
- Repository queries include authorized company scope server-side.
- Storage path obscurity is not authorization.
- Cross-company System Administrator access is explicit and audited.
- Tests must attempt horizontal access by changing IDs and URLs.

## 11. Carbone and Converter Isolation

- Carbone is accessed through a fixed, allowlisted server-side URL.
- The client cannot supply the Carbone host, converter, version, or arbitrary callback URL.
- Carbone and its converter run on a private network with no unnecessary inbound exposure.
- Network egress is restricted where deployment supports it.
- Requests have connection, total-duration, request-size, and response-size limits.
- Worker concurrency is bounded to protect LibreOffice and host resources.
- Temporary files use isolated job directories and are deleted after bounded retention.
- Carbone errors are treated as untrusted text and sanitized before logs or API responses.

## 12. Generated PDF Validation

Before storage as a successful result:

- Require HTTP success from the configured Carbone endpoint.
- Enforce maximum response size.
- Verify PDF signature and minimum structural validity.
- Reject HTML, JSON, or error text returned with an incorrect content type.
- Compute SHA-256 after receipt.
- Store the PDF before marking the render job successful.
- Create the generated-document record and supersede transition transactionally where possible.

## 13. Integrity and Provenance

Record hashes and immutable identifiers for:

- Original template
- Branch asset files
- Prepared-template cache artifact when retained
- Final generated PDF
- Business data snapshot
- Approval snapshot

A GeneratedDocument points to exact template, contract, asset, preparer, and workflow snapshot versions.

## 14. Secrets and Sensitive Data

- Carbone configuration, Microsoft Graph credentials, tenant ID, client ID, certificates/secrets, mailbox address, storage credentials, and signing keys remain server-side.
- Secrets are never returned to the browser, stored in document snapshots, or written to application logs.
- Use managed identity or certificate credentials when available; otherwise use a secret manager with rotation.
- Redact authorization headers, access tokens, payloads, and private file paths from logs.

## 15. Microsoft Graph Email Safety

- Use only the configured central mailbox.
- Resolve recipients from authorized backend user/organization data.
- Do not accept arbitrary recipient arrays from workflow clients.
- Do not attach final PDFs by default.
- Treat Graph acceptance as request acceptance, not proof of final mailbox delivery.
- Record provider request ID, result category, attempt count, and bounded error text.

## 16. Audit Events

Minimum events:

```text
file_uploaded
file_rejected
file_quarantined
malware_scan_completed
template_scanned
template_validation_failed
template_previewed
template_activated
document_render_requested
document_render_failed
document_generated
document_downloaded
generated_document_superseded
generated_document_voided
```

Audit events must not contain the full document payload or file content.

## 17. Error Handling

Public errors include a stable code, readable corrective message, and correlation ID.

Examples:

```text
unsupported_file_type
file_too_large
unsafe_archive
invalid_docx
malware_detected
template_validation_failed
file_access_denied
render_timeout
invalid_render_response
```

Internal diagnostic detail is access-controlled and redacted.

## 18. Retention and Deletion

- Files referenced by issued documents, workflow history, or generated-document snapshots are not physically deleted by normal UI actions.
- Archive, supersede, void, and cancel are metadata transitions.
- Temporary upload and render files use short operational retention.
- Business retention periods and legal holds require a separate approved retention policy before production rollout.

## 19. Security Test Baseline

- Extension/MIME/magic mismatch
- DOCX missing required package parts
- ZIP slip and absolute entry paths
- ZIP bomb, excessive ratio, excessive entry count, and oversized XML
- Encrypted ZIP
- XML external entity payload
- Malformed image and excessive dimensions
- Malware scanner success, detection, timeout, and unavailable states
- Cross-company IDOR for templates, assets, attachments, previews, and PDFs
- Unauthorized signed URL request
- Carbone timeout, oversized response, HTML error response, and invalid PDF
- Log-redaction verification
- Concurrent render completion and supersede race

## 20. Related Documents

- [Document Engine Design](../superpowers/specs/2026-07-15-procurehub-document-engine-design.md)
- [System Architecture](../architecture/SYSTEM_ARCHITECTURE.md)
- [ADR-0002: Carbone Document Rendering](../decisions/ADR-0002-carbone-document-rendering.md)
- [ADR-0004: Microsoft Graph Email Delivery](../decisions/ADR-0004-microsoft-graph-email-delivery.md)
