---
ticket: 08
phase: Foundation
module: Master / Inventory
screens: ["M03 Item/UOM", "M05 Warehouse/Locator", "S01 (Inventory Org)"]
refs: ["D05", "D10"]
blocked_by: []
tags:
  - erp/ticket
  - erp/foundation
---

# 08: สินค้า หน่วยนับ คลัง และ Inventory Org

**Status:** ready-for-agent
**Type:** task
**Blocked by:** None (can start immediately)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

ข้อมูลหลักสต็อก: Inventory Org แบบ logical แยกแบรนด์ใต้ OU, Subinventory, Locator, สินค้าและการแปลงหน่วย เพื่อให้โมดูลสต็อกและจัดซื้อ/ขายใช้ได้

หน้าจอ: M03 Item/UOM, M05 Warehouse/Locator, S01 (Inventory Org)
อ้างอิง: D05, D10

## ขอบเขต

- InventoryOrg (ใต้ OU), Subinventory (ประเภท valued/non-valued/inspection/in-transit), Locator
- Item, UOM, UOMConversion
- หน้าจอ M03, M05 และเพิ่ม Inventory Org ใน S01

## Acceptance criteria

- [ ] Inventory Org ผูก OU ของบริษัทเดียวเท่านั้น
- [ ] แปลงหน่วยไป-กลับได้ค่าเท่าเดิมตาม precision 6 ตำแหน่ง
- [ ] รหัสสินค้าไม่ซ้ำและคงเลขศูนย์นำหน้า

## เทสต์ที่ต้องมี

- [ ] e2e: org ข้ามบริษัท, UOM conversion, locator ซ้ำ

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] สิทธิ์แก้ข้อมูลหลักตามบทบาท
- [ ] audit ทุกการแก้
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
