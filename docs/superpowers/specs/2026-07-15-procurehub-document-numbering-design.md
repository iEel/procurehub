# ProcureHub Document Numbering Design

**Status:** Approved<br>
**Date:** 2026-07-15<br>
**Approved date:** 2026-07-15<br>
**Approved by:** Product Owner<br>
**Product:** ProcureHub<br>
Scope: Generic, configurable, concurrent-safe official document numbering

## 1. Purpose

ProcureHub supports multiple companies and branches whose official document-number formats differ. Number allocation must be configurable without hard-coded PR or PO rules, deterministic under concurrency, effective-dated, and permanently auditable.

This service assigns official numbers only. Draft display IDs, database IDs, and temporary preview labels are separate concepts.

## 2. Confirmed Decisions

- Numbering rules are generic and selected by `documentType` plus organization context.
- Each company owns and approves its numbering rules.
- Official numbers are allocated only at the issuance action registered by the business document adapter.
- Drafts do not consume an official number.
- A number committed as issued is never reused, including after rejection, cancellation, voiding, regeneration, or downstream failure.
- Pattern parsing, preview, rule activation, allocation, and audit are owned by Node.js.
- Numbering does not depend on Carbone or PDF generation.
- Rule versions are immutable after activation or first use.
- Allocation is transactional and protected against concurrent duplicate or skipped-current-value claims.
- Preview never consumes or reserves a sequence value.

## 3. Non-Goals

- Business-specific PR or PO issuance logic in the numbering core
- Reusing cancelled or voided numbers
- Renumbering historical documents when a rule changes
- User-authored code, expressions, database queries, or arbitrary date formatting in a pattern
- Sharing one sequence across unrelated document types in version 1
- Manually editing the current sequence from the normal administration UI

## 4. Core Records

### 4.1 DocumentNumberRule

```text
id
documentType
companyId
branchId nullable
departmentId nullable
ruleVersion
name
pattern
resetPeriod: never | yearly | monthly
issueTriggerAction
timezone
lifecycleStatus: draft | active | archived
effectiveFrom
effectiveTo nullable
createdBy/createdAt
activatedBy/activatedAt nullable
concurrencyVersion
```

The scope is exact. A null branch or department represents a broader company-owned scope, not missing data.

### 4.2 DocumentSequence

```text
id
ruleId
scopeKey
periodKey
currentValue
concurrencyVersion
updatedAt
```

Unique `(ruleId, scopeKey, periodKey)`.

### 4.3 IssuedDocumentNumber

```text
id
companyId
documentType
refId
documentNumber
ruleId
ruleVersion
scopeKey
periodKey
sequenceValue
issueTriggerAction
issuedAt
issuedBy
```

This record is immutable. `documentNumber` is unique within the company. `(documentType, refId)` can have at most one official number unless a future document-type specification explicitly introduces a legally valid replacement-number model.

### 4.4 NumberAllocationAudit

Append-only allocation, validation failure, activation, archive, reconciliation, and administrative correction events. Safe metadata includes rule/scope IDs, attempted period, actor, correlation ID, and outcome; it never includes secrets.

## 5. Rule Selection

At the registered issuance action, select one complete active rule in this order:

1. Company + Branch + Department + Document Type
2. Company + Branch + Document Type
3. Company Default + Document Type

Rules are never merged. There is no Global Default for official document numbers in version 1. If no rule applies, issuance is blocked with `document_number_rule_not_found`.

The selected rule must be active at the issuance timestamp in its configured timezone. Overlapping active effective periods for the same exact scope are rejected.

## 6. Allowed Pattern Tokens

Version 1 supports these tokens:

| Token | Meaning | Example |
| --- | --- | --- |
| `{DOC_TYPE}` | Registered document-type code | `PR` |
| `{COMPANY_CODE}` | Company code | `SONIC` |
| `{BRANCH_CODE}` | Branch code | `HQ` |
| `{DEPARTMENT_CODE}` | Department code | `IT` |
| `{YYYY}` | Gregorian four-digit year | `2026` |
| `{YY}` | Gregorian two-digit year | `26` |
| `{YYYY_BE}` | Buddhist Era four-digit year | `2569` |
| `{YY_BE}` | Buddhist Era two-digit year | `69` |
| `{MM}` | Two-digit month | `07` |
| `{DD}` | Two-digit day | `15` |
| `{SEQ:0000}` | Zero-padded sequence; number of zeroes defines width | `0001` |

Pattern examples:

```text
PR-{COMPANY_CODE}-{BRANCH_CODE}-{YYYY}-{SEQ:0000}
PO/{BRANCH_CODE}/{YY_BE}/{MM}/{SEQ:00000}
{DOC_TYPE}-{DEPARTMENT_CODE}-{YYYY}{MM}-{SEQ:0000}
```

`{YYYYMM}` is not a version 1 token. Authors must write `{YYYY}{MM}` as shown above.

Pattern rules:

- Exactly one `{SEQ:...}` token is required.
- Sequence width is 1–12 zeroes.
- Unknown, repeated sequence, malformed, or context-incompatible tokens prevent activation.
- Static text is limited to uppercase ASCII letters, digits, hyphen, slash, underscore, and period.
- Company, branch, department, and document-type codes use the same safe character set.
- A pattern cannot exceed 120 characters before or after formatting.
- Formatting uses the issuance timestamp converted to the rule timezone.
- Buddhist Era year equals the Gregorian year plus 543.
- A rule using `{BRANCH_CODE}` or `{DEPARTMENT_CODE}` must select a context containing that value; the service never inserts an empty token.

## 7. Reset Period and Sequence Scope

`periodKey` is derived from `resetPeriod`:

| Reset period | Period key example |
| --- | --- |
| `never` | `ALL` |
| `yearly` | `2026` |
| `monthly` | `2026-07` |

The period uses the rule timezone and Gregorian values internally, even when the visible pattern uses Buddhist Era tokens.

`scopeKey` is deterministic from the selected exact rule scope and document type. Version 1 does not share a sequence between different rules or document types.

Changing reset behavior or scope requires a new rule version. It never mutates an existing `DocumentSequence` identity.

## 8. Issuance Trigger

The numbering core does not know PR or PO lifecycle semantics. Each registered business document adapter declares one canonical `issueTriggerAction`, such as `submit` or `issue`, and the rule must match that registration.

Rules:

- Save Draft never allocates an official number.
- The same issuance request is idempotent for `(documentType, refId, issueTriggerAction)`.
- Repeating the issuance action returns the existing issued number.
- Preview, Word-template preview, and PDF regeneration never allocate a number.
- Cancellation or voiding changes the business lifecycle but does not release the number.

## 9. Concurrent Allocation Transaction

The default allocation flow runs in the same database transaction as the business lifecycle transition that makes the number official:

1. Authorize the actor and lock or compare-and-swap the business record.
2. Return the existing number when the issuance idempotency key already completed.
3. Select and validate the effective rule.
4. Lock the `(ruleId, scopeKey, periodKey)` sequence row or create it safely.
5. Increment `currentValue` once.
6. Format the candidate number and verify uniqueness.
7. Insert the immutable `IssuedDocumentNumber`.
8. Attach the number and perform the business lifecycle transition.
9. Commit all changes atomically.

Supported implementations may use a row lock, serializable transaction, database sequence adapter with an immutable reservation record, or optimistic compare-and-swap. The chosen database ADR and implementation plan must prove equivalent uniqueness and idempotency.

## 10. Failure and Gap Policy

- If the shared transaction fails before commit, the sequence change, issued-number record, and document transition all roll back. No official number was issued.
- If the transaction commits and a later action fails, including PDF generation, notification, or external integration, the number remains issued and cannot be reused.
- The failed downstream action is retried independently; numbering is not retried with a new number.
- If the selected persistence technology cannot roll back its sequence primitive, it must write an immutable reservation before exposing the value. Abandoned reserved values become audited gaps and are never reused.
- Administrators cannot close a gap by decrementing `currentValue`.
- Any exceptional data repair requires a separate privileged command, reason, before/after snapshot, and audit event; it cannot change an issued number.

Gaps are acceptable when required to preserve uniqueness and audit truth. Duplicate or reassigned official numbers are not acceptable.

## 11. Rule Lifecycle and Effective Dates

Lifecycle:

```text
draft -> active -> archived
```

Activation requires:

- valid exact scope and registered document type
- valid pattern and tokens
- valid timezone
- valid reset period and issue trigger
- no overlapping active effective period for the exact scope
- successful preview cases
- activation note and audit event

An active or used rule version is immutable. Editing clones a new draft version. Running and historical documents retain their original rule ID/version.

Activating a replacement rule may close the prior rule's effective period transactionally. The UI displays which rule becomes effective for each representative scope before confirmation.

## 12. Preview and Simulation

Preview input includes document type, company, branch, department, issuance date/time, and a non-persistent example sequence value.

Preview output includes:

- selected rule and fallback explanation
- rule version and exact scope
- resolved token values
- period key and scope key
- formatted examples for sequence values `1`, `12`, and the maximum width boundary
- validation errors and readable correction guidance

Preview and simulation never read or update `DocumentSequence.currentValue`.

Required saved cases cover:

- every active company rule scope
- branch override and company fallback
- Gregorian and Buddhist Era patterns
- yearly, monthly, and never reset
- month/year boundary in the configured timezone
- missing context required by a token
- sequence-width overflow

## 13. Overflow

When `currentValue` exceeds the configured sequence width, issuance is blocked with `document_number_sequence_overflow`. The service never expands the width silently because that changes the approved official format.

Administrators must activate a new compatible rule version or perform an approved period/scope correction. The failed issuance consumes no number when the transaction rolls back.

## 14. Error Codes

```text
document_number_rule_not_found
document_number_rule_not_effective
document_number_pattern_invalid
document_number_token_context_missing
document_number_issue_trigger_mismatch
document_number_sequence_conflict
document_number_sequence_overflow
document_number_already_issued
document_number_allocation_failed
```

Responses include a readable localized message and correlation ID. Internal lock, SQL, or provider details are not exposed.

## 15. Security and Authorization

- Only authorized numbering administrators can create, activate, archive, or simulate rules beyond ordinary examples.
- Issuance is invoked only through an authorized business lifecycle service; the frontend cannot request an arbitrary next number.
- Company scope is validated server-side for rule management, preview, and issuance.
- Sequence rows and issued-number records are never directly editable through public APIs.
- Every privileged action writes an audit event.

## 16. Testing

Minimum coverage:

- pattern parser accepts every supported token and rejects unknown/malformed tokens
- Buddhist Era and timezone boundary formatting
- each reset-period key
- exact rule selection and allowed fallback
- missing rule blocks issuance
- preview does not consume a sequence
- repeat issuance is idempotent
- concurrent allocation produces unique ordered values
- transaction rollback leaves no issued record
- committed number survives PDF or notification failure
- cancelled/voided number is never reused
- overflow blocks without silently widening
- effective-date replacement preserves historical rule versions
- cross-company rule and sequence isolation

Concurrency tests must use the selected real database adapter; an in-memory repository alone is insufficient evidence.

## 17. Acceptance Criteria

- A company can define different formats by document type, branch, and department.
- Only supported tokens can be activated.
- Drafts and previews do not consume official numbers.
- Issuance is idempotent and concurrent-safe.
- Issued numbers are unique and never reused.
- Effective-dated rule changes do not alter historical documents.
- Cancellation, voiding, rendering failure, and notification failure do not release a number.
- Missing or invalid configuration blocks safely with readable errors.
- Allocation and exceptional administration are auditable.
- The core contains no PR- or PO-specific numbering rule.

## 18. Related Documents

- [Developer Handoff](../../../DEVELOPER_HANDOFF.md)
- [System Architecture](../../architecture/SYSTEM_ARCHITECTURE.md)
- [Core Data Model](../../data/CORE_DATA_MODEL.md)
- [Organization Model](../../product/ORGANIZATION_MODEL.md)
- [Product Terminology](../../product/TERMINOLOGY.md)
- [File and Document Security](../../security/FILE_AND_DOCUMENT_SECURITY.md)
