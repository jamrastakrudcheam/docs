---
ticket: 07
phase: Foundation
module: Control
screens: ["Audit log (ใหม่)"]
refs: ["T08", "03 Constraints"]
blocked_by: []
tags:
  - erp/ticket
  - erp/foundation
---

# 07: Idempotency และหน้าดู Audit

**Status:** resolved
**Type:** task
**Blocked by:** None (can start immediately)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

เครื่องมือกลางกันการบันทึกซ้ำจาก double-click/หลายแท็บ และหน้าให้ผู้ตรวจสอบค้นประวัติการเปลี่ยนแปลง

หน้าจอ: Audit log (ใหม่)
อ้างอิง: T08, 03 Constraints

## ขอบเขต

- IdempotencyRecord: user + scope + command + key + request hash → ผลลัพธ์ ใน transaction เดียว
- decorator/helper ให้ command final ใช้ `Idempotency-Key`
- หน้า audit: ค้นตามเอกสาร ผู้ใช้ ช่วงวันที่ บริษัท

## Acceptance criteria

- [x] key เดิม payload เดิมได้ผลเดิมโดยไม่ทำซ้ำ
- [x] key เดิม payload ต่างได้ 409
- [x] ผู้ตรวจสอบ (Auditor) อ่าน audit ได้แต่แก้ไม่ได้ และเห็นเฉพาะบริษัทที่มีสิทธิ์

## เทสต์ที่ต้องมี

- [x] e2e ด้วย command ทดสอบ: ยิงซ้ำพร้อมกัน, payload ต่าง
- [x] e2e: scope ของ audit viewer

## มาตรฐานโค้ดและความปลอดภัย

- [x] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [x] audit ไม่มี endpoint แก้/ลบ
- [x] ไม่แสดงข้อมูลลับใน before/after
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments

- 2026-10-26: ทำเสร็จบน branch `07-idempotency-audit-viewer`
  - `IdempotencyService.run()` + `@IdempotencyKey()` (header `Idempotency-Key`, ต้องมี, 8–100 ตัว) เก็บใน `audit.idempotency_record` ใน transaction เดียวกับงาน; ยิงซ้ำได้ผลเดิมพร้อม header `Idempotent-Replayed: true`; ทำงานพลาด record ก็ rollback ด้วย จึงลองใหม่ได้
  - `GET /api/v1/audit-entries` อ่านอย่างเดียว: System Admin เห็นทั้งหมด, Auditor เห็นเฉพาะบริษัทที่มีสิทธิ์; `audit_entry.company_id` เพิ่มใน migration 0005
  - หน้า `/audit` (ค้นตามบริษัท ประเภท ID ผู้ใช้ การกระทำ ช่วงวันที่ไทย, ดู before/after, แบ่งหน้า) และ `api.post(..., { idempotencyKey })` ฝั่งเว็บ
  - ยังไม่ได้รัน `/code-review` แบบเต็ม (ตรวจเองแล้ว)
