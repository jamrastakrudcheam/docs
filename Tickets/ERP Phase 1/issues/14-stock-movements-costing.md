---
ticket: 14
phase: Inventory Core
module: Inventory
screens: ["I01 On-hand", "I02 Movements", "I08 Cost History/Valuation"]
refs: ["D10", "P07", "T09"]
blocked_by: [08, 37, 07]
tags:
  - erp/ticket
  - erp/inventory-core
---

# 14: ความเคลื่อนไหวสต็อกและต้นทุนถัวเฉลี่ย

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [08 สินค้า หน่วยนับ คลัง และ Inventory Org](08-items-uom-warehouses.md), [37 SLA: แปลงเหตุการณ์จากโมดูลย่อยเป็น Journal](37-sla-subledger-accounting.md), [07 Idempotency และหน้าดู Audit](07-idempotency-audit-viewer.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

แกนกลางสต็อก: ทุก movement บันทึกจำนวนและมูลค่าแบบมีเครื่องหมายพร้อมเอกสารต้นทาง คงเหลือต่อ Org/สินค้า/locator ต้นทุนถัวเฉลี่ยเคลื่อนที่ต่อ Org+สินค้า และลง SLA ในคำสั่งเดียวกัน

หน้าจอ: I01 On-hand, I02 Movements, I08 Cost History/Valuation
อ้างอิง: D10, P07, T09

## ขอบเขต

- StockMovement, StockBalance, CostBucket, CostAdjustment
- service: issue/receive/adjust ที่ล็อก stock bucket และเรียก SLA
- costing policy version (P07) สำหรับ return, price variance, backdating
- หน้า I01, I02, I08

## Acceptance criteria

- [ ] สต็อกติดลบไม่ได้
- [ ] ต้นทุนเฉลี่ยคำนวณแบบ precision สูง จัดการเศษตามนโยบาย qty=0 ไม่หารศูนย์
- [ ] ปรับราคาหลังขายบางส่วนแยก inventory/COGS/variance
- [ ] มูลค่าสต็อกตรงกับ GL

## เทสต์ที่ต้องมี

- [ ] e2e: รับ-จ่าย-ติดลบ, ต้นทุนเฉลี่ยหลายรอบ, price variance หลังขาย (T09)

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ห้าม UPDATE/DELETE movement ใช้ adjustment
- [ ] ล็อก bucket ตามลำดับ org/item
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
