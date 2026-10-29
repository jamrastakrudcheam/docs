---
ticket: 03
phase: Foundation
module: Workflow
screens: ["S03 Workflow", "H02 Approval Inbox"]
refs: ["D07", "T04", "T08"]
blocked_by: []
tags:
  - erp/ticket
  - erp/foundation
---

# 03: Workflow อนุมัติและ Approval Inbox

**Status:** resolved
**Type:** task
**Blocked by:** None (can start immediately)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

เครื่องมืออนุมัติกลางที่ทุกโมดูลใช้ร่วมกัน ผู้ดูแลกำหนดสายอนุมัติตามชนิดเอกสาร OU และวงเงิน ผู้อนุมัติเห็นงานรอใน Inbox และอนุมัติหรือส่งกลับได้ทั้งบน desktop และมือถือ

หน้าจอ: S03 Workflow, H02 Approval Inbox
อ้างอิง: D07, T04, T08

## ขอบเขต

- WorkflowDefinition/Version ต่อ document type, route ตาม OU + ช่วงวงเงิน, หลายขั้น
- ApprovalInstance/Step, การมอบอำนาจแบบมีช่วงเวลา
- API: `/approvals` (inbox), `/{id}/approve`, `/{id}/return` พร้อมเหตุผล
- interface ให้โมดูลอื่นเรียก: submit(document) → instance, onApproved/onReturned
- หน้า S03 ตั้งค่า และ H02 Inbox (มือถือแสดงแบบรายการ)

## Acceptance criteria

- [x] ผู้สร้างเอกสารอนุมัติเอกสารตัวเองไม่ได้ รวมถึงผ่านการมอบอำนาจ
- [x] แก้สาระสำคัญหลัง submit ทำให้การอนุมัติเดิมเป็นโมฆะและเริ่ม route ใหม่
- [x] route เลือกตาม OU และวงเงินของเอกสาร ณ เวลาที่ submit
- [x] Inbox แสดงเฉพาะงานในบริษัท/OU ที่ผู้ใช้มีสิทธิ์
- [x] อนุมัติซ้ำ/กดสองครั้งไม่ทำให้ข้ามขั้น

## เทสต์ที่ต้องมี

- [x] e2e ด้วยเอกสารทดสอบ (fake document type) ครอบคลุม self-approval, revision reset, routing ตามวงเงิน, delegation หมดอายุ, scope

## มาตรฐานโค้ดและความปลอดภัย

- [x] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [x] ตรวจสิทธิ์อนุมัติที่ API ทุกครั้ง ไม่ใช่แค่ซ่อนปุ่ม
- [x] audit ทุกการอนุมัติ/ส่งกลับพร้อมเหตุผล
- [x] System Admin ไม่เป็นผู้อนุมัติการเงินโดยปริยาย
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments

- 2026-10-27: ทำเสร็จบน branch `03-approval-workflow-inbox`
  - Workflow ต่อบริษัท + ประเภทเอกสาร มี version ที่แก้ไม่ได้: route ตาม OU (OU เฉพาะก่อน "ทุก OU") + ช่วงวงเงิน, หลายขั้น แต่ละขั้นอนุมัติโดยบทบาท · ห้ามใช้ System Admin/Auditor เป็นผู้อนุมัติ
  - `ApprovalsService.submit()` (เรียกใน transaction ของเอกสาร) เลือก route ณ เวลาส่ง · ส่งซ้ำเนื้อหาเดิมได้รายการเดิม · แก้สาระสำคัญ → รายการเดิมเป็น `superseded` แล้วเริ่ม route ใหม่ · `registerHandler(docType, {onApproved, onReturned})` ทำงานใน transaction เดียวกับการอนุมัติ
  - ผู้อนุมัติต้องถือบทบาทของขั้นในบริษัท/OU ของเอกสาร หรือได้รับมอบอำนาจที่มีผล (ช่วงวันที่, ยกเลิกได้) · ผู้สร้างอนุมัติไม่ได้ทั้งทางตรงและผ่านการมอบอำนาจ (`SELF_APPROVAL`) · ต้องส่ง `expected_step` กดซ้ำได้ 409 ไม่ข้ามขั้น · ส่งกลับต้องมีเหตุผล · ทุกการกระทำลง audit
  - หน้า H02 `/approvals` (รายการแบบการ์ด ใช้บนมือถือได้ + แท็บมอบอำนาจ) และ S03 `/settings/workflows`
  - เทสต์ใช้เอกสารปลอม `TEST_DOC` · โมดูลจริงตัวแรกที่ใช้คือ GL Journal (ticket 12)
