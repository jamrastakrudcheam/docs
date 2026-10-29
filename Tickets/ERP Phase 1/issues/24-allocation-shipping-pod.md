---
ticket: 24
phase: O2C
module: Shipping
screens: ["O04 Allocation", "F01 Delivery Workbench", "F02 Pick", "F03 Pack/Shipment", "F04 Ship Confirm", "F05 POD"]
refs: ["D10", "P03", "P04", "T02"]
blocked_by: [23, 15, 01]
tags:
  - erp/ticket
  - erp/o2c
---

# 24: จัดสรร หยิบ แพ็ก ส่ง และ POD

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [23 ใบเสนอราคา ใบสั่งขาย และการตรวจเครดิต](23-quotation-sales-order-credit.md), [15 การจองสต็อกและโอนข้าม Org](15-reservations-transfers.md), [01 ตัดสินใจประเด็นออกแบบที่ค้าง](01-resolve-open-design-decisions.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

จัดสรรสต็อกให้ SO, หยิบ/แพ็กต่อบรรทัด, Ship Confirm ลดสต็อกและการจองพร้อมลง COGS/SLA และกันยืนยันซ้ำ, บันทึก carrier/tracking/POD เอง

หน้าจอ: O04 Allocation, F01 Delivery Workbench, F02 Pick, F03 Pack/Shipment, F04 Ship Confirm, F05 POD
อ้างอิง: D10, P03, P04, T02

## ขอบเขต

- วันที่ภาษีขาย (P19): บันทึกวันที่ส่งมอบต่อ shipment line — ไซต์ออกบิลตอน Ship Confirm ใช้วัน Ship Confirm, ไซต์ POD ใช้วันที่ใน POD — ส่งต่อให้ ticket 25 ใช้เป็นวันที่ใบกำกับ
- Allocation, Pick/PackLine, Shipment/ShipmentLine/ShipmentSource, DeliveryAttempt, PODAttachment
- ส่งบางส่วน
- คู่บัญชีตาม P16 / ADR-0006: ไซต์ออกบิลหลัง POD ลง Dr Shipped-not-billed / Cr Inventory แล้วโอนเข้า COGS ตอนออก invoice · ไซต์ออกบิลตอน Ship Confirm ลง COGS ตรง

## Acceptance criteria

- [ ] allocation ไม่ลด on-hand
- [ ] ship confirm ซ้ำไม่ตัดสต็อกซ้ำ
- [ ] POD เปลี่ยนสถานะส่ง ไม่ตัดสต็อกซ้ำ
- [ ] ส่ง 6+4 ครบ 10 ไม่ติดลบ

## เทสต์ที่ต้องมี

- [ ] e2e: ส่วน shipping ของ T02, double confirm

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ฝ่ายจัดส่งแก้ invoice/AR ไม่ได้
- [ ] ไฟล์ POD ตรวจชนิด/ขนาด
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
