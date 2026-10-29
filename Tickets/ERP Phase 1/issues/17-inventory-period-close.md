---
ticket: 17
phase: Inventory Core
module: Inventory
screens: ["I09 Org Period"]
refs: ["D10", "T09", "T10"]
blocked_by: [14, 04]
tags:
  - erp/ticket
  - erp/inventory-core
---

# 17: ปิดงวดสต็อกต่อ Org

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [14 ความเคลื่อนไหวสต็อกและต้นทุนถัวเฉลี่ย](14-stock-movements-costing.md), [04 ปฏิทินบัญชีและงวดของแต่ละโมดูล](04-fiscal-calendar-periods.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

ปิดงวด INV-COST ของแต่ละ Org โดยตรวจรายการค้างและความต่างระหว่างมูลค่าสต็อกกับ GL ก่อนปิด

หน้าจอ: I09 Org Period
อ้างอิง: D10, T09, T10

## ขอบเขต

- precheck: movement ค้าง, cost adjustment ค้าง, stock value ↔ GL
- ปิด/เปิดงวดผ่าน ticket 04

## Acceptance criteria

- [ ] ปิดไม่ได้ถ้ามีรายการค้าง พร้อมรายการที่บล็อก
- [ ] หลังปิด movement ลงวันที่ในงวดนั้นไม่ได้

## เทสต์ที่ต้องมี

- [ ] e2e: precheck blockers, post หลังปิด

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ปิดงวดเฉพาะบทบาทที่กำหนด
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
