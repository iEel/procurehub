# ProcureHub Document Engine Design

**Status:** Approved<br>
**Date:** 2026-07-15<br>
**Approved date:** 2026-07-15<br>
**Approved by:** Product Owner<br>
**Product:** ProcureHub<br>
Scope: Generic contract-first DOCX template management, preparation, Carbone rendering, and generated-document storage

## 1. Purpose

ProcureHub requires a reusable document engine that supports any future `documentType` without embedding PR, PO, quotation, receipt, invoice, or other business logic in the core.

Node.js owns contracts, source data mapping, validation, templates, versions, storage, preparation, generated PDFs, and audit history. Carbone On-Premise only merges a prepared DOCX with validated JSON and converts it to PDF.

## 2. Confirmed Decisions

- Contract-first design is mandatory.
- Word authors copy tags from a system-generated Tag Sheet.
- Unknown tags prevent template activation.
- Preview must succeed before activation.
- Template lifecycle, validation status, and preview status are separate.
- The frontend sends `documentType` and `refId`, not final render data.
- The backend loads authoritative data through a registered mapper.
- Node.js stores every final PDF and render snapshot.
- Historical template and generated-document versions are immutable.
- Carbone Community/Free dynamic-image features are not required.
- Header and footer images are patched into the DOCX package by Node.js before Carbone rendering.
- One master DOCX form is maintained per document type in version 1; company/branch identity is provided by versioned document assets.
- Template activation uses maker-checker separation in version 1: the uploader cannot activate the same template version.
- Rendering is queue-capable and idempotent.

## 3. Non-Goals

- Business-specific PR or PO contracts
- A browser-based Word template designer
- Arbitrary user-authored expressions or code
- Dynamic images handled by Carbone Enterprise features
- Direct public access to stored templates or generated files
- Overwriting old templates or PDFs
- Trusting a complete payload supplied by the frontend

## 4. Module Boundaries

```text
Contract Registry
  Owns document type, contract version, schema, allowed tags, and sample cases.

Data Mapper Registry
  Maps an authorized refId to contract-shaped data.

Template Management
  Owns upload, versioning, validation, preview, activation, and archive.

DOCX Scanner and Validator
  Detects tags and checks them against the contract.

DOCX Preparation
  Applies optional versioned transformations such as branch header/footer patching.

Carbone Client
  Sends prepared DOCX and validated JSON to Carbone and returns a PDF buffer.

Storage
  Owns safe relative paths, reads, writes, and authorized URL generation.

Render Orchestration
  Owns jobs, retries, idempotency, snapshots, supersede rules, and final records.
```

Each module exposes typed interfaces and can be tested independently.

## 5. Document Contract

```ts
type DocumentType = string;

type TemplateTagDefinition = {
  label: string;
  tag: string;
  path: string;
  required: boolean;
  group?: string;
  helper?: boolean;
  description?: string;
};

type DocumentSampleCase = {
  code: string;
  label: string;
  data: Record<string, unknown>;
  context?: Record<string, unknown>;
};

type DocumentContract = {
  documentType: DocumentType;
  contractVersion: string;
  tags: TemplateTagDefinition[];
  schema: unknown;
  sampleCases: DocumentSampleCase[];
};
```

Registry operations:

```text
register(contract)
get(documentType, contractVersion?)
getCurrent(documentType)
list()
getAllowedTags(documentType, contractVersion?)
```

Registration fails for duplicate document type/version, duplicate tag definitions, invalid tag syntax, inconsistent paths, or sample data that fails the schema.

## 6. Data Mapper

```ts
type DocumentDataMapper = {
  documentType: DocumentType;
  mapData(input: {
    refId: string;
    actorId: string;
    companyId: string;
    branchId?: string;
  }): Promise<Record<string, unknown>>;
};
```

The mapper:

- authorizes access before loading data
- loads authoritative records from backend repositories
- returns data shaped to the current contract
- never accepts calculated totals, approval identity, or final display values from the browser
- provides deterministic data suitable for snapshotting

## 7. Tag Sheet

The Tag Sheet API exposes only registered definitions:

```text
GET /api/document-contracts
GET /api/document-contracts/:documentType
GET /api/document-contracts/:documentType/tags
```

Each tag includes label, exact copyable tag, path, required flag, group, helper flag, description, and contract version.

Carbone syntax is treated as its own language. The system must not invent Mustache, Handlebars, JSONPath, or Jinja syntax.

Word author rules:

- Use one pair of braces, for example `{d.vendor.name}`.
- Use lowercase `i` for iterators.
- A repeated block uses valid Carbone start and end markers, such as `{d.items[i].name}` and `{d.items[i+1]}`.
- Do not apply formatting to only part of a tag.
- Use straight single quotes in formatter arguments; do not use smart quotes.
- Do not rename or manually construct tags.
- Request a contract change when a field is missing.

## 8. DOCX Tag Scanner

DOCX is processed as an untrusted ZIP package under the file-security specification.

Scanned parts include, when present:

```text
word/document.xml
word/header*.xml
word/footer*.xml
word/footnotes.xml
word/endnotes.xml
```

The scanner:

1. Opens the validated ZIP in memory or controlled temporary storage.
2. Reads only approved WordprocessingML parts.
3. Reconstructs text across adjacent XML text nodes so Word run splitting does not automatically hide a tag.
4. Examines drawing alternative-text attributes for configured image placeholders.
5. Extracts unique tags beginning with `{d.`.
6. Decodes XML entities safely without resolving external entities.
7. Returns sorted detected tags and scanner warnings.

The scanner is best effort. A successful Carbone preview is still required.

## 9. Tag Normalization and Validation

`normalizeCarboneTag(tag)` removes formatter chains while preserving the base data reference and iterator form.

Examples:

```text
{d.total:formatN(2)}             -> {d.total}
{d.date:formatD('YYYY-MM-DD')}   -> {d.date}
{d.items[i].name}                -> {d.items[i].name}
{d.items[i+1]}                   -> {d.items[i+1]}
```

Normalization must parse formatter separators outside quoted arguments and parentheses. It must not remove a colon by using an unsafe first-colon regular expression.

Validation returns:

```text
detectedTags
normalizedDetectedTags
allowedTags
unknownTags
missingRequiredTags
warnings
suggestions
validationErrors
```

Simple edit-distance suggestions are allowed. Suggestions are never auto-applied.

Loop start/end markers and helper tags are registered explicitly in the Tag Sheet. The validator does not invent or repair loop structure. Preview catches remaining document-structure errors.

## 10. Template State Model

These fields are independent:

```text
lifecycleStatus: draft | active | archived
validationStatus: pending | valid | invalid
previewStatus: not_generated | generating | succeeded | failed
```

Activation requires:

```text
lifecycleStatus = draft
validationStatus = valid
previewStatus = succeeded
unknownTags is empty
contractVersion is compatible with the current contract
file-security validation passed
activation policy passed
```

Version 1 activation policy requires `activatedBy != createdBy`. The API enforces this rule server-side even if the client hides or disables the action.

An invalid upload remains available to its authorized administrator with readable errors, unless security validation requires immediate quarantine or rejection.

## 11. Template Versioning

- Version numbers are assigned server-side within each document type.
- A stored version is never overwritten.
- A file hash is recorded.
- Exactly one active master template exists per document type in version 1.
- Activating a draft archives the previously active version transactionally.
- The user who uploaded or created a template version cannot activate that same version.
- Existing generated documents keep the original template and contract versions.
- Regeneration with a new version creates a new GeneratedDocument record.

Future company-specific master layouts require a separate approved template-scope design. They are not silently introduced through filename conventions.

## 12. Template Upload

```text
POST /api/document-templates/upload
GET  /api/document-templates
GET  /api/document-templates/:id
```

Upload input:

```text
documentType
name
file (.docx)
```

Flow:

1. Authenticate and authorize administrator.
2. Validate document type and current contract.
3. Enforce file-security limits and DOCX structure.
4. Compute SHA-256.
5. Store the immutable original DOCX under a server-generated relative path.
6. Scan and validate tags.
7. Create a template version with separated statuses.
8. Return structured validation results.

Maximum compressed upload size is 20 MB. Additional ZIP safety limits are defined in the security specification.

## 13. Company and Branch Document Assets

Header and footer images are versioned assets selected using document company and branch context.

```text
DocumentAssetSet
- id
- companyId
- branchId nullable; null means Company Default
- assetVersion
- headerImagePath
- footerImagePath
- file hashes
- lifecycleStatus
- effectiveFrom
- createdBy
- createdAt
```

Asset rules:

- Store relative paths only.
- Validate image type using content signatures.
- Preserve old versions used by generated documents.
- Header and footer dimensions/aspect ratios follow the master-template placeholder contract.
- Exactly one active asset set exists per exact `(companyId, branchId)` scope.
- A document type that requires branded assets selects them in this order: Company + Branch, then Company Default.
- There is no Global Default for final branded documents. If neither allowed scope exists, generation is blocked with a readable configuration error.
- The selected scope and asset version are stored in the generated-document snapshot.
- Asset activation requires a successful preview against the current active master template.

Preview coverage rules:

- Before a master template can activate, all registered sample cases must pass at least once.
- Every active company/branch asset set must pass the normal-data and multi-page sample cases against the candidate template.
- Before a new asset set can activate, it must pass the normal-data and multi-page sample cases against the current active template.
- Preview results record template version, asset-set ID/version, sample-case code, preparer version, and output hash.
- A failed required combination blocks activation; administrators cannot replace it with a warning-only override in version 1.

## 14. DOCX Preparation and Header/Footer Patching

```ts
type PrepareTemplateInput = {
  templateBuffer: Buffer;
  documentType: string;
  refId?: string;
  data: Record<string, unknown>;
  context: Record<string, unknown>;
};

type TemplatePreparer = {
  prepare(input: PrepareTemplateInput): Promise<Buffer>;
};
```

The default preparer returns the original buffer. The document-asset preparer:

1. Opens a working copy of the validated DOCX.
2. Locates configured header/footer image placeholders by controlled alternative-text tags.
3. Resolves the target media relationship and package part.
4. Clones a shared media part when replacement would unintentionally alter another image.
5. Replaces the binary image while preserving Word layout, size, anchor, margins, and section references.
6. Updates content-type metadata only when the approved image format requires it.
7. Verifies that every required placeholder was replaced exactly as configured.
8. Returns a prepared buffer without modifying the immutable master template.

Carbone does not perform this image replacement.

Optional compiled-template caching uses this key:

```text
masterTemplateHash
assetSetVersion
preparerVersion
outputDocxCompatibilityVersion
```

Cache files are derived artifacts and can be rebuilt. Generated-document records retain the source template and asset versions, not only a cache path.

## 15. Preview

```text
POST /api/document-templates/:id/preview
```

Preview flow:

1. Load template and contract.
2. Re-run or verify current validation results.
3. Select a registered sample case and authorized sample organization context.
4. Validate sample data against the schema.
5. Resolve Company + Branch assets, fall back only to Company Default, and prepare the DOCX when the template requires branded assets.
6. Render through Carbone.
7. Validate the response as a PDF.
8. Store the preview PDF.
9. Update preview status, timestamp, sample case, asset version, and safe error summary.

Minimum sample cases:

- normal data
- one line item
- many line items spanning pages
- long Thai and English text
- optional values absent
- high-value and multi-currency values

Preview success proves renderability for the selected cases, not correctness of future business data.

## 16. Carbone Client

Environment direction:

```text
CARBONE_URL
CARBONE_CONVERTER=L
CARBONE_VERSION=5
```

Server-side method:

```ts
generatePdfWithCarbone({
  templateBuffer,
  data,
  reportName
}): Promise<Buffer>
```

Request:

```text
POST {CARBONE_URL}/render/template?download=true
Content-Type: application/json
carbone-version: 5
```

```json
{
  "data": {},
  "template": "<base64 prepared DOCX>",
  "convertTo": "pdf",
  "converter": "L",
  "reportName": "server-generated-name"
}
```

The endpoint, converter, version, timeout, and response-size limit are server configuration. Clients cannot override them.

Typed failures:

```text
missing_config
network_error
timeout
http_error
invalid_response
response_too_large
```

Safe logs include correlation ID, status, duration, template version, and bounded error text. They never include document payloads, secrets, or full template content.

## 17. Generic Generation Flow

```text
Frontend sends documentType + refId
  -> authorize document access
  -> load active template and current contract
  -> map authoritative data
  -> validate schema and lifecycle eligibility
  -> create idempotent GeneratedDocument intent and render job
  -> load immutable master template
  -> select versioned company/branch assets
  -> prepare DOCX
  -> call Carbone
  -> validate PDF response
  -> save final PDF
  -> update GeneratedDocument with PDF metadata and immutable snapshots
  -> supersede prior active PDF when policy requires
  -> emit timeline event
  -> return authorized PDF URL and metadata
```

The final stored snapshot includes:

```text
documentType and refId
documentNumber nullable
templateId and templateVersion
contractVersion
documentAssetSetId and assetVersion nullable
preparerVersion
dataSnapshotJson
approvalSnapshotJson nullable
file hash
PDF storage path
generatedBy and generatedAt
```

## 18. Render Queue and Idempotency

Render job statuses:

```text
queued
rendering
retrying
succeeded
failed
cancelled
```

Rules:

- The API can use a synchronous adapter during early development, but orchestration remains queue-shaped.
- An idempotency key prevents duplicate final records for the same explicit request.
- Network and retryable Carbone failures use bounded exponential backoff with jitter.
- Invalid template, schema, authorization, and security failures are not retried automatically.
- A worker lease or optimistic version prevents two workers from completing one job.
- A successful replacement marks the prior current PDF `superseded` transactionally.

## 19. Storage

Logical paths:

```text
templates/{documentType}/{templateId}/v{version}.docx
assets/{companyId}/{branchId-or-company-default}/{assetSetId}/...
previews/{documentType}/{templateId}/{previewId}.pdf
generated/{documentType}/{year}/{generatedDocumentId}.pdf
```

`StorageService` operations:

```text
saveBuffer(relativePath, buffer)
readBuffer(relativePath)
exists(relativePath)
getAuthorizedUrl(relativePath, actorContext)
```

Paths are generated server-side and normalized. Storage can move from local disk to S3-compatible storage through the same interface.

## 20. Core Records

### DocumentTemplate

```text
id, documentType, name, templateVersion
lifecycleStatus, validationStatus, previewStatus
storagePath, fileHash
detectedTags, unknownTags, missingRequiredTags
contractVersion, validationErrors
previewPdfPath, lastPreviewedAt
createdBy, createdAt, activatedBy, activatedAt
concurrencyVersion
```

### GeneratedDocument

```text
id, documentType, refId, documentNumber nullable
templateId, templateVersion, contractVersion
documentAssetSetId/assetVersion nullable
dataSnapshotJson, approvalSnapshotJson nullable
pdfPath/fileHash nullable until successful generation
status: generating | generated | failed | superseded | voided
generatedBy, requestedAt, generatedAt nullable
```

A terminal render failure updates the intent record to `failed` and leaves PDF metadata empty. A retry may continue the same intent while the job is retryable. An explicit later generation request creates a new intent.

### RenderJob

```text
id, documentType, refId, templateId
idempotencyKey
status: queued | rendering | retrying | succeeded | failed | cancelled
attemptCount, maxAttempts
lastErrorCode, safeLastError
requestedBy, createdAt, startedAt, finishedAt
leaseOwner/leaseUntil or version token
```

## 21. API Surface

```text
GET  /api/document-contracts
GET  /api/document-contracts/:documentType
GET  /api/document-contracts/:documentType/tags

POST /api/document-templates/upload
GET  /api/document-templates
GET  /api/document-templates/:id
POST /api/document-templates/:id/preview
POST /api/document-templates/:id/activate

POST /api/documents/:documentType/:refId/generate
GET  /api/documents/:documentType/:refId/generated
GET  /api/generated-documents/:id
```

All endpoints require server-side authorization and return stable error codes plus readable messages and correlation IDs.

## 22. Security

The [File and Document Security](../../security/FILE_AND_DOCUMENT_SECURITY.md) specification is mandatory.

Key controls:

- verify extension, MIME, magic bytes, ZIP structure, and DOCX required parts
- limit compressed size, uncompressed size, entry count, and compression ratio
- prevent ZIP slip and path traversal
- do not resolve XML external entities
- store outside unauthenticated public roots
- enforce company/document authorization for every read and download
- keep Carbone on an allowlisted private endpoint
- add a malware-scanning hook before activation or final use
- never expose service credentials to the frontend

## 23. Testing

Minimum tests:

- contract registry duplicate and version rules
- sample data schema validation
- formatter-aware tag normalization
- array loop start and end tags
- known, unknown, and missing required tags
- split Word runs and XML entities
- header/footer and drawing alt-text scanning
- ZIP bomb, ZIP slip, invalid DOCX, and oversized upload rejection
- template activation gates and archival transaction
- shared media-part-safe image patching
- unchanged master template after preparation
- Carbone success, timeout, throttling/server error, and invalid PDF response
- render idempotency, retry, worker race, and supersede behavior
- cross-company authorization denial
- immutable snapshots and file hashes

## 24. Acceptance Criteria

- A generic contract can be registered without changing engine core.
- The Tag Sheet exposes exact allowed tags.
- A DOCX upload is safely scanned and validated.
- Unknown tags and missing required tags are readable and block activation.
- Lifecycle, validation, and preview statuses are independent.
- Preview success is required for activation.
- One immutable active master version is selected per document type.
- Node.js patches resolved, versioned company/branch header/footer assets without Carbone dynamic images.
- Required asset coverage and maker-checker policy block unsafe template or asset activation.
- Carbone receives only a prepared DOCX and validated JSON.
- Final PDFs and snapshots are stored by Node.js.
- Re-rendering creates a new record and preserves history.
- Queue and storage implementations are replaceable through interfaces.
- No real business document logic is hard-coded in the engine.

## 25. Related Documents

- [Developer Handoff](../../../DEVELOPER_HANDOFF.md)
- [File and Document Security](../../security/FILE_AND_DOCUMENT_SECURITY.md)
- [Core Data Model](../../data/CORE_DATA_MODEL.md)
- [ADR-0002: Carbone Document Rendering](../../decisions/ADR-0002-carbone-document-rendering.md)
- [ProcureHub Design System](../../design/DESIGN_SYSTEM.md)
