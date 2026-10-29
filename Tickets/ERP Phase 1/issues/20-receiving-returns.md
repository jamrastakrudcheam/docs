---
ticket: 20
phase: P2P
module: Purchasing / Inventory
screens: ["PO04 Receiving", "PO05 Return to Supplier"]
refs: ["D10", "P07", "T01"]
blocked_by: [19, 14]
tags:
  - erp/ticket
  - erp/p2p
---

# 20: รับสินค้าและคืนผู้ขาย

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [19 ใบสั่งซื้อ (PO) และ PO Revision](19-purchase-order-revision.md), [14 ความเคลื่อนไหวสต็อกและต้นทุนถัวเฉลี่ย](14-stock-movements-costing.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

บันทึกรับของ (จำนวนจริง, ตรวจรับ, ปลายทาง) เมื่อ accept/put-away ปรับ received qty, สต็อก, ต้นทุน และ SLA พร้อมกัน และคืนสินค้าอ้าง receipt

หน้าจอ: PO04 Receiving, PO05 Return to Supplier
อ้างอิง: D10, P07, T01

## ขอบเขต

- Receiving/ReceivingLine, ReceiptDistribution
- accept/put-away, return-to-supplier
- SLA: Dr Inventory / Cr GRNI

## Acceptance criteria

- [ ] รับเกินจำนวน PO ไม่ได้ (FK/unique source allocation)
- [ ] รับ 6+4 แล้วสต็อก 10, คืน 1 แล้วเหลือ 9
- [ ] คืนเกินที่รับไม่ได้ และไม่ลบ receipt เดิม

## เทสต์ที่ต้องมี

- [ ] e2e: ส่วน receiving/return ของ T01

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] Idempotency-Key ที่ accept
- [ ] ผู้รับของตรวจสิทธิ์ Org ปลายทาง
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
