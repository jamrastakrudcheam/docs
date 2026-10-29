---
ticket: 16
phase: Inventory Core
module: Inventory
screens: ["I05 Count Session", "I06 Count Entry/Recount", "I07 Variance Approval"]
refs: ["P01", "T03"]
blocked_by: [14, 03]
tags:
  - erp/ticket
  - erp/inventory-core
---

# 16: ตรวจนับสต็อก (Physical / Cycle Count)

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [14 ความเคลื่อนไหวสต็อกและต้นทุนถัวเฉลี่ย](14-stock-movements-costing.md), [03 Workflow อนุมัติและ Approval Inbox](03-approval-workflow-inbox.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

รอบตรวจนับที่ freeze ขอบเขต, นับแบบ blind, นับซ้ำตามเกณฑ์ และอนุมัติผลต่างโดยคนละคนกับผู้บันทึก ก่อนปรับสต็อกและบัญชีครั้งเดียว

หน้าจอ: I05 Count Session, I06 Count Entry/Recount, I07 Variance Approval
อ้างอิง: P01, T03

## ขอบเขต

- CountSession/Line/Attempt/Variance
- สถานะ DRAFT → SCOPE_FROZEN → COUNTING → REVIEW → RECOUNT/APPROVED → ADJUSTED → CLOSED
- เกณฑ์นับซ้ำ (จำนวน/มูลค่า) ตั้งค่าได้

## Acceptance criteria

- [ ] freeze บล็อก receiving/shipping/transfer ใน scope ทุกเส้นทาง
- [ ] blind count ไม่ส่ง expected qty ให้ผู้นับ รวมใน export
- [ ] ผู้บันทึกอนุมัติผลต่างตัวเองไม่ได้
- [ ] adjust ส่งซ้ำไม่ปรับซ้ำ
- [ ] นับได้น้อยกว่ายอดจองต้องระบุวิธีจัดการก่อน final

## เทสต์ที่ต้องมี

- [ ] e2e: T03 ครบ (short/over/recount/deny/concurrent movement)

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ซ่อน expected qty ที่ API ไม่ใช่แค่หน้าเว็บ
- [ ] attempt แก้ไม่ได้ (immutable)
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
