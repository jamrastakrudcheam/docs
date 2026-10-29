---
ticket: 22
phase: P2P
module: AP / Treasury
screens: ["A03 Payment Proposal", "A04 Payment", "A05 Supplier Aging"]
refs: ["D03", "T01", "T08"]
blocked_by: [21]
tags:
  - erp/ticket
  - erp/p2p
---

# 22: เสนอจ่ายและจ่ายเงิน (จำลอง)

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [21 ใบแจ้งหนี้เจ้าหนี้ (AP) และ Matching/Hold](21-ap-invoice-matching.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

สร้างข้อเสนอจ่าย (ยังไม่จ่ายจริง) แล้ว finalize payment จำลองที่ล็อกยอดค้าง สร้าง application/clearing/SLA พร้อมกัน และ aging เจ้าหนี้

หน้าจอ: A03 Payment Proposal, A04 Payment, A05 Supplier Aging
อ้างอิง: D03, T01, T08

## ขอบเขต

- WHT รวมเงินจ่ายล่วงหน้า, ฐานไม่รวม VAT, ภาษีออกแทนเงื่อนไข 2/3 = เงินได้ × อัตรา/(100 − อัตรา) ลงค่าใช้จ่ายแยกบัญชี (P21)
- WHT คำนวณตอนจ่ายเงิน (P21) ตั้งหนี้ภาษีค้างนำส่งกับผู้ขายประเภทหน่วยงานภาษี (กรมสรรพากร) และนำส่งผ่าน AP
- PaymentProposal, Payment, PaymentApplication
- finalize/reverse (สิทธิ์จำกัด)
- SLA ผ่าน clearing account
- Supplier aging ต่อบริษัท

## Acceptance criteria

- [ ] สองคนจ่าย invoice เดียวกันพร้อมกัน ผ่านได้คนเดียว
- [ ] จ่ายบางส่วนได้ ยอดค้างถูกต้อง
- [ ] ไม่มีการเรียกธนาคารจริง

## เทสต์ที่ต้องมี

- [ ] e2e: T01 ช่วงจ่าย/หักลดหนี้, concurrent finalize (T08)

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] แยกสิทธิ์ผู้เสนอจ่ายกับผู้ finalize
- [ ] แสดงบัญชีผู้ขายแบบ mask
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
