---
ticket: 27
phase: O2C
module: Collection
screens: ["C01 Worklist", "C02 Customer Case", "C03 Billing Batch/Statement", "C04 Promise/Dispute"]
refs: ["D12", "T06"]
blocked_by: [25]
tags:
  - erp/ticket
  - erp/o2c
---

# 27: ติดตามหนี้ (Collection)

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [25 ออกบิลและใบแจ้งหนี้ลูกหนี้ (AR)](25-billing-ar-invoice.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

งานติดตามหนี้: รายการค้างชำระ, case ต่อลูกค้า, บันทึกการติดต่อ, นัดชำระ, ข้อโต้แย้ง และวางบิล/statement โดยไม่เปลี่ยนยอดลูกหนี้

หน้าจอ: C01 Worklist, C02 Customer Case, C03 Billing Batch/Statement, C04 Promise/Dispute
อ้างอิง: D12, T06

## ขอบเขต

- CollectionCase, CaseInvoice, ContactActivity, PromiseToPay, Dispute, BillingBatch

## Acceptance criteria

- [ ] นัดชำระไม่ลด AR และไม่ลง GL
- [ ] case แสดงเงินรับจริงที่ match แล้ว
- [ ] collector เปลี่ยนยอดหนี้หรือรับเงินผ่าน case ไม่ได้

## เทสต์ที่ต้องมี

- [ ] e2e: T06 ครบ

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ข้อมูลติดต่อลูกค้าเป็นข้อมูลส่วนบุคคล
- [ ] scope ตามบริษัท/OU
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
