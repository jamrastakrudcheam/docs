---
ticket: 11
phase: Foundation
module: Master / Pricing
screens: ["M04 Price/Terms"]
refs: ["D06"]
blocked_by: [08]
tags:
  - erp/ticket
  - erp/foundation
---

# 11: ราคาขาย/ซื้อ และเงื่อนไขชำระเงิน

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [08 สินค้า หน่วยนับ คลัง และ Inventory Org](08-items-uom-warehouses.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

รายการราคาและเงื่อนไขการชำระเงินที่มีวันที่มีผล ใช้ใน Quotation/SO/PO

หน้าจอ: M04 Price/Terms
อ้างอิง: D06

## ขอบเขต

- PriceList/Line (THB, effective dates)
- PaymentTerm (due days)
- หน้า M04

## Acceptance criteria

- [ ] เลือกราคาตามวันที่เอกสารได้ถูก
- [ ] ช่วงวันที่ราคาซ้อนกันถูกปฏิเสธ

## เทสต์ที่ต้องมี

- [ ] e2e: ราคาข้ามวันมีผล, ซ้อนช่วง

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] สิทธิ์แก้ราคาเฉพาะบทบาทที่กำหนด
- [ ] audit การเปลี่ยนราคา
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
