---
ticket: 25
phase: O2C
module: Billing / AR
screens: ["R01 Billing Workbench", "R02 AR Invoice"]
refs: ["P02", "P03", "T02"]
blocked_by: [24, 36]
tags:
  - erp/ticket
  - erp/o2c
---

# 25: ออกบิลและใบแจ้งหนี้ลูกหนี้ (AR)

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [24 จัดสรร หยิบ แพ็ก ส่ง และ POD](24-allocation-shipping-pod.md), [36 บริการนำเข้าไฟล์กลาง (แบบ Oracle FBDI)](36-file-based-data-import.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

ออกบิลจาก shipment ที่ถึงเงื่อนไข (หลัง Ship Confirm หรือ POD ตามไซต์) รวมหลาย shipment ได้เมื่อเงื่อนไขตรงกัน ล็อกจำนวนที่ออกบิล และเก็บ snapshot ราคา/ภาษี

หน้าจอ: R01 Billing Workbench, R02 AR Invoice
อ้างอิง: P02, P03, T02

## ขอบเขต

- ใบกำกับภาษีขายสินค้าลงวันที่ส่งมอบ (P19) ไม่ใช่วันที่สร้าง invoice · ใบกำกับบริการตามตัวเลือกของ Customer Site: ออกตอนออกบิล หรือออกตอนรับเงิน
- นำเข้าผ่านบริการกลาง (ticket 36): import object `AR_INTERFACE_LINES` เทียบ Fusion AutoInvoice Import · หน่วยความสำเร็จ: ต่อกลุ่มตาม grouping rule ของ customer site (P02)
- ARInvoice/Line, InvoiceShipmentAllocation, AROpenItem
- `/billing-eligible-lines`, `/ar-invoices/{id}/post`
- SLA: Dr AR / Cr Revenue (+Tax), ย้าย credit commitment ไป AR

## Acceptance criteria

- [ ] ออกบิลซ้ำจำนวนเดิมไม่ได้แม้เปิดสองหน้าจอ
- [ ] รวมบิลได้เฉพาะเมื่อ company/OU, bill-to, currency, terms ตรงกัน
- [ ] ยอดคำนวณฝั่ง server
- [ ] credit exposure ต่อเนื่องไม่หายหรือซ้ำ

## เทสต์ที่ต้องมี

- [ ] e2e: ส่วน invoice ของ T02, รวมบิล P02

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ไม่เชื่อยอดรวมจาก browser
- [ ] Idempotency-Key ที่ post
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
