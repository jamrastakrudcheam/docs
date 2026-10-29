---
ticket: 19
phase: P2P
module: Purchasing
screens: ["PO02 PO List/Form", "PO03 PO Revision"]
refs: ["D08", "D07", "T01"]
blocked_by: [18, 11]
tags:
  - erp/ticket
  - erp/p2p
---

# 19: ใบสั่งซื้อ (PO) และ PO Revision

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [18 ใบขอซื้อ (PR) และการจองงบ](18-purchase-requisition.md), [11 ราคาขาย/ซื้อ และเงื่อนไขชำระเงิน](11-price-lists-terms.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

แปลง PR เป็น PO โดยโอน commitment เฉพาะจำนวนที่ใช้ ไม่จองซ้ำ, แก้ PO แบบ revision ที่ต้องอนุมัติใหม่, ปิดยอดค้างคืนงบ

หน้าจอ: PO02 PO List/Form, PO03 PO Revision
อ้างอิง: D08, D07, T01

## ขอบเขต

- PO/POLine, POLineSource (PR line → PO line)
- `POST /purchase-orders/from-requisition-lines`
- revise/cancel/close

## Acceptance criteria

- [ ] PO ไม่จองงบซ้ำกับ PR
- [ ] revision ที่เปลี่ยนราคา/จำนวนต้องอนุมัติใหม่
- [ ] ปิด PO คืนงบส่วนที่ไม่ได้ใช้

## เทสต์ที่ต้องมี

- [ ] e2e: PR บางส่วน → PO, revision, close (ส่วนหนึ่งของ T01/T04)

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ล็อก PR line และ reservation ตอนแปลง
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
