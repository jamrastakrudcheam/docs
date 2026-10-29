---
ticket: 30
phase: FA
module: Fixed Assets
screens: ["FAI05 Transfer", "FAI06 Physical Count", "FAI07 Retirement"]
refs: ["D11", "T07"]
blocked_by: [29, 36]
tags:
  - erp/ticket
  - erp/fa
---

# 30: โอน ตรวจนับ และจำหน่ายสินทรัพย์

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [29 ค่าเสื่อมราคา Corporate/Tax Book](29-fa-depreciation-books.md), [36 บริการนำเข้าไฟล์กลาง (แบบ Oracle FBDI)](36-file-based-data-import.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

โอนสินทรัพย์ระหว่าง location/custodian, ตรวจนับสินทรัพย์ (แยกจากการนับสต็อก) และจำหน่ายพร้อมคำนวณ cost/accumulated/NBV/proceeds/gain-loss

หน้าจอ: FAI05 Transfer, FAI06 Physical Count, FAI07 Retirement
อ้างอิง: D11, T07

## ขอบเขต

- นำเข้าผ่านบริการกลาง (ticket 36): import object `FA_PHYSICAL_INVENTORY_INT` เทียบ Fusion Physical inventory import · หน่วยความสำเร็จ: ต่อรอบตรวจนับ
- AssetTransfer, AssetCount, Retirement

## Acceptance criteria

- [ ] จำหน่ายแล้วคิดค่าเสื่อมต่อไม่ได้
- [ ] gain/loss ลงบัญชีถูก
- [ ] โอนต้องอนุมัติ

## เทสต์ที่ต้องมี

- [ ] e2e: ส่วน transfer/count/retire ของ T07

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] อนุมัติแยกคนกับผู้ทำรายการ
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
