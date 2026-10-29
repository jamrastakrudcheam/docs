---
ticket: 23
phase: O2C
module: Order Management
screens: ["O01 Quotation", "O02 SO", "O03 Credit/Hold Review"]
refs: ["D09", "T05"]
blocked_by: [09, 11, 03, 05]
tags:
  - erp/ticket
  - erp/o2c
---

# 23: ใบเสนอราคา ใบสั่งขาย และการตรวจเครดิต

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [09 ลูกค้า ไซต์ และวงเงินเครดิต](09-customers-credit.md), [11 ราคาขาย/ซื้อ และเงื่อนไขชำระเงิน](11-price-lists-terms.md), [03 Workflow อนุมัติและ Approval Inbox](03-approval-workflow-inbox.md), [05 เลขที่เอกสาร](05-document-numbering.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

Quotation → SO พร้อมตรวจราคา/GP และเครดิต (หนี้คงค้าง + ยอดสั่งที่จองเครดิต) ถ้าเกินหรือค้างชำระให้ Hold และขอปลดผ่านผู้อนุมัติเครดิต

หน้าจอ: O01 Quotation, O02 SO, O03 Credit/Hold Review
อ้างอิง: D09, T05

## ขอบเขต

- Quotation/Line, SO/Line, CreditReservation
- request-release/approve-release
- หน้า O01–O03

## Acceptance criteria

- [ ] สอง SO พร้อมกันรวมเกินวงเงิน ผ่านได้แค่ตามวงเงิน
- [ ] ฝ่ายขายปลด hold เองไม่ได้
- [ ] การปลด hold มีเหตุผล/ยอด/audit

## เทสต์ที่ต้องมี

- [ ] e2e: T05 ครบ

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ล็อก credit bucket ต่อ customer
- [ ] scope ตาม OU ของ SO
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
