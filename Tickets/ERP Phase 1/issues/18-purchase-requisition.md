---
ticket: 18
phase: P2P
module: Purchasing
screens: ["PO01 PR List/Form"]
refs: ["D08", "T01", "T04"]
blocked_by: [13, 10, 08, 05]
tags:
  - erp/ticket
  - erp/p2p
---

# 18: ใบขอซื้อ (PR) และการจองงบ

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [13 งบประมาณและการจองงบ](13-budget-reservations.md), [10 ผู้ขาย ไซต์ และบัญชีธนาคาร](10-suppliers-bank.md), [08 สินค้า หน่วยนับ คลัง และ Inventory Org](08-items-uom-warehouses.md), [05 เลขที่เอกสาร](05-document-numbering.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

สร้าง PR, ส่งอนุมัติ และเมื่ออนุมัติให้จองงบใน transaction เดียว ถ้าเกินงบให้ rollback การอนุมัติ

หน้าจอ: PO01 PR List/Form
อ้างอิง: D08, T01, T04

## ขอบเขต

- PR/PRLine, สถานะ DRAFT → SUBMITTED → APPROVED/RETURNED → PO ALLOCATED → CLOSED/CANCELLED
- ยกเลิก/ปิดยอดค้างคืนงบ

## Acceptance criteria

- [ ] อนุมัติแล้วจองงบพร้อมกัน เกินงบไม่อนุมัติ
- [ ] ยกเลิก PR คืนงบที่จองไว้ทั้งหมด
- [ ] ผู้ขออนุมัติ PR ตัวเองไม่ได้

## เทสต์ที่ต้องมี

- [ ] e2e: T04 (ยอดพอดี/เกิน/ยกเลิก/แก้ไข)

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] scope PR ตาม OU ของผู้ขอ
- [ ] Idempotency-Key ที่ approve
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
