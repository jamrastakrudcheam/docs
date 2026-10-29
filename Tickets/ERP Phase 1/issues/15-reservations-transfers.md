---
ticket: 15
phase: Inventory Core
module: Inventory
screens: ["I03 Reservations", "I04 Transfer"]
refs: ["D10", "P07", "T09"]
blocked_by: [14]
tags:
  - erp/ticket
  - erp/inventory-core
---

# 15: การจองสต็อกและโอนข้าม Org

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [14 ความเคลื่อนไหวสต็อกและต้นทุนถัวเฉลี่ย](14-stock-movements-costing.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

จองสต็อกก่อนหยิบ (available = eligible on-hand − reservations − holds) และโอนข้าม Org ที่มี shipment/receipt คู่กัน สินค้าระหว่างทางไม่นับเป็นพร้อมขายที่ปลายทาง

หน้าจอ: I03 Reservations, I04 Transfer
อ้างอิง: D10, P07, T09

## ขอบเขต

- Reservation (source document), Transfer/TransferLine, in-transit subinventory
- dispatch/receive ตาม valuation policy (P07)
- หน้า I03, I04

## Acceptance criteria

- [ ] จองเกิน available ไม่ได้
- [ ] in-transit ไม่ถูกนับซ้ำที่ต้นทางหรือปลายทาง
- [ ] โอนข้าม Org ลง SLA ทั้งสองฝั่ง

## เทสต์ที่ต้องมี

- [ ] e2e: จองพร้อมกัน, โอนบางส่วน, รับปลายทาง (T09)

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] โอนข้าม Org ตรวจสิทธิ์ทั้งต้นทางและปลายทาง
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
