# ProcureHub Product Terminology

**Status:** Approved<br>
**Date:** 2026-07-15<br>
**Approved date:** 2026-07-15<br>
**Approved by:** Product Owner<br>
**Product:** ProcureHub<br>
Language direction: Thai-first UI with canonical English codes

## 1. Purpose

ProcureHub stores stable canonical codes and translates them for display. Database state and API contracts never depend on Thai or English labels.

Rules:

- Canonical codes use lowercase `snake_case`.
- UI text uses translation keys.
- The same action uses the same label across modules.
- Reject and Return for Revision are never interchangeable.
- Historical snapshots may store labels for audit, but logic uses canonical codes.

## 2. Product and Organization Terms

| Canonical term | English | Thai |
| --- | --- | --- |
| `company` | Company | บริษัท |
| `branch` | Branch | สาขา |
| `head_office` | Head Office | สำนักงานใหญ่ |
| `division` | Division | ฝ่าย |
| `department` | Department | แผนก |
| `position` | Position | ตำแหน่ง |
| `requester` | Requester | ผู้ขอซื้อ |
| `owner_department` | Owner Department | แผนกเจ้าของเรื่อง |
| `approver` | Approver | ผู้อนุมัติ |
| `delegate` | Delegate | ผู้รับมอบหมาย |
| `system_administrator` | System Administrator | ผู้ดูแลระบบส่วนกลาง |

## 3. Procurement Terms

| Canonical term | English | Thai |
| --- | --- | --- |
| `purchase_requisition` | Purchase Requisition (PR) | ใบขอซื้อ |
| `purchase_order` | Purchase Order (PO) | ใบสั่งซื้อ |
| `request_for_quotation` | Request for Quotation (RFQ) | ใบขอเสนอราคา |
| `goods_receipt` | Goods Receipt | ใบรับสินค้า |
| `service_acceptance` | Service Acceptance | ใบรับรองงานบริการ |
| `vendor` | Vendor / Supplier | ผู้ขาย / ผู้ให้บริการ |
| `attachment` | Attachment | เอกสารแนบ |
| `document_number` | Document Number | เลขที่เอกสาร |

## 4. Document Lifecycle Status

| Code | English | Thai | Meaning |
| --- | --- | --- | --- |
| `draft` | Draft | ฉบับร่าง | Editable and not submitted |
| `pending_approval` | Pending Approval | รออนุมัติ | Submitted and normally locked |
| `returned_for_revision` | Returned for Revision | ส่งกลับแก้ไข | Returned to owner and editable |
| `approved` | Approved | อนุมัติแล้ว | Required approval completed |
| `rejected` | Rejected | ไม่อนุมัติ | Final rejection; workflow ended |
| `cancelled` | Cancelled | ยกเลิก | Business document cancelled |
| `closed` | Closed | ปิดงาน | Business process completed |

## 5. Approval Task Status

| Code | English | Thai | Meaning |
| --- | --- | --- | --- |
| `waiting` | Waiting | รอตามลำดับ | Future step; no final assignee snapshot yet |
| `pending` | Pending Action | รอดำเนินการ | Current actionable task |
| `approved` | Approved | อนุมัติแล้ว | Approval action completed |
| `acknowledged` | Acknowledged | รับทราบแล้ว | Acknowledgement completed |
| `returned` | Returned | ส่งกลับแล้ว | Returned for revision |
| `rejected` | Rejected | ไม่อนุมัติ | Rejected and workflow stopped |
| `skipped` | Skipped | ข้ามตามเงื่อนไข | Condition did not apply |
| `cancelled` | Cancelled | ยกเลิก | Task cancelled by lifecycle change |
| `resolution_error` | Resolution Error | ไม่พบผู้รับผิดชอบ | Approver could not be resolved |

### SLA Indicator

`overdue` is a derived SLA indicator on a `pending` task. It is not an Approval Task status and never replaces the primary task status.

| Code | English | Thai | Meaning |
| --- | --- | --- | --- |
| `overdue` | Overdue | เกินกำหนด | Pending task exceeded its calculated SLA due time |

## 6. Workflow Actions

| Code | English | Thai | Consequence |
| --- | --- | --- | --- |
| `save_draft` | Save Draft | บันทึกฉบับร่าง | Saves editable document |
| `submit` | Submit for Approval | ส่งขออนุมัติ | Starts preflight and workflow |
| `approve` | Approve | อนุมัติ | Completes an approval step |
| `acknowledge` | Acknowledge | รับทราบ | Completes an acknowledgement step |
| `return` | Return for Revision | ส่งกลับแก้ไข | Unlocks document for correction |
| `reject` | Reject | ไม่อนุมัติ | Ends workflow as rejected |
| `resubmit` | Resubmit | ส่งขออนุมัติอีกครั้ง | Starts next approval round |
| `cancel` | Cancel | ยกเลิก | Cancels business document |
| `generate_pdf` | Generate PDF | สร้าง PDF | Queues document rendering |
| `retry_resolution` | Retry Resolve Approver | ค้นหาผู้อนุมัติอีกครั้ง | Repairs resolution error |

`approve` remains the canonical version 1 backend command for `approve`, `review`, and `verify` steps. The visible label is derived from the step action type:

| Step action type | UI label (English) | UI label (Thai) | Task result |
| --- | --- | --- | --- |
| `approve` | Approve | อนุมัติ | `approved` |
| `review` | Complete Review | ตรวจสอบเสร็จสิ้น | `approved` |
| `verify` | Verify | ยืนยันการตรวจสอบ | `approved` |

Approval history records both the canonical command and the configured step action type so a review or verification is never displayed as a business approval.

## 7. Template Status

Template status is separated into three dimensions.

### Lifecycle

| Code | English | Thai |
| --- | --- | --- |
| `draft` | Draft | ฉบับร่าง |
| `active` | Active | ใช้งานอยู่ |
| `archived` | Archived | เก็บถาวร |

### Validation

| Code | English | Thai |
| --- | --- | --- |
| `pending` | Pending Validation | รอตรวจสอบ |
| `valid` | Valid | ตรวจสอบผ่าน |
| `invalid` | Invalid | ตรวจสอบไม่ผ่าน |

### Preview

| Code | English | Thai |
| --- | --- | --- |
| `not_generated` | Not Generated | ยังไม่สร้างตัวอย่าง |
| `generating` | Generating | กำลังสร้างตัวอย่าง |
| `succeeded` | Preview Succeeded | สร้างตัวอย่างสำเร็จ |
| `failed` | Preview Failed | สร้างตัวอย่างไม่สำเร็จ |

## 8. Render Job Status

| Code | English | Thai |
| --- | --- | --- |
| `queued` | Queued | รอสร้าง |
| `rendering` | Rendering | กำลังสร้างเอกสาร |
| `retrying` | Retrying | กำลังลองใหม่ |
| `succeeded` | Succeeded | ประมวลผลสำเร็จ |
| `failed` | Failed | สร้างไม่สำเร็จ |
| `cancelled` | Cancelled | ยกเลิกงานสร้าง |

## 9. Generated Document Status

| Code | English | Thai |
| --- | --- | --- |
| `generating` | Generating | กำลังสร้างเอกสาร |
| `generated` | Generated | สร้างสำเร็จ |
| `failed` | Failed | สร้างไม่สำเร็จ |
| `superseded` | Superseded | ถูกแทนที่ด้วยฉบับใหม่ |
| `voided` | Voided | ยกเลิกการใช้งานเอกสาร |

## 10. Workflow Administration Terms

| Canonical term | English | Thai |
| --- | --- | --- |
| `workflow_definition` | Workflow Definition | รูปแบบขั้นตอนอนุมัติ |
| `workflow_version` | Workflow Version | เวอร์ชันขั้นตอนอนุมัติ |
| `workflow_instance` | Workflow Instance | กระบวนการอนุมัติของเอกสาร |
| `approval_round` | Approval Round | รอบการอนุมัติ |
| `approval_step` | Approval Step | ขั้นตอนอนุมัติ |
| `approval_task` | Approval Task | งานอนุมัติ |
| `approver_resolver` | Approver Resolver | กติกาค้นหาผู้อนุมัติ |
| `delegation` | Delegation | การมอบหมายอนุมัติแทน |
| `sla_due_at` | SLA Due Time | เวลาครบกำหนด SLA |
| `timeline` | Timeline | ลำดับเหตุการณ์ |
| `revision` | Revision | ฉบับแก้ไข |

## 11. Display Rules

- Thai UI may show the Thai label first and English technical term in help text.
- Document numbers are never translated or reformatted.
- Currency code is always displayed, for example `125,000.00 THB`.
- Dates follow user/company locale and include an unambiguous year.
- Error messages identify the corrective action without exposing internal stack traces.
- Email and in-app notifications use the same canonical terminology as the application.

## 12. Related Documents

- [ProcureHub Design System](../design/DESIGN_SYSTEM.md)
- [Organization Model](ORGANIZATION_MODEL.md)
- [Workflow, SLA, and Notification Design](../superpowers/specs/2026-07-13-procurehub-workflow-design.md)
- [Document Numbering Design](../superpowers/specs/2026-07-15-procurehub-document-numbering-design.md)
