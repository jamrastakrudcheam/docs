---
ticket: 26
phase: O2C
module: AR
screens: ["R03 Credit/Debit Memo", "R04 Receipt/Application", "R05 Customer Aging"]
refs: ["T02", "T06", "T08"]
blocked_by: [25, 36]
tags:
  - erp/ticket
  - erp/o2c
---

# 26: รับชำระ ตัดหนี้ ลดหนี้/เพิ่มหนี้ และรับคืนสินค้า

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [25 ออกบิลและใบแจ้งหนี้ลูกหนี้ (AR)](25-billing-ar-invoice.md), [36 บริการนำเข้าไฟล์กลาง (แบบ Oracle FBDI)](36-file-based-data-import.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

บันทึกเงินรับและตัดหนี้ตามยอดที่จัดสรร, ลด/เพิ่มหนี้อ้าง invoice พร้อมเหตุผลและอนุมัติ, รับคืนสินค้าเป็นอีกคำสั่งที่เชื่อมกันได้

หน้าจอ: R03 Credit/Debit Memo, R04 Receipt/Application, R05 Customer Aging
อ้างอิง: T02, T06, T08

## ขอบเขต

- ภาษีขายค่าบริการ tax point = payment (P19): ตอนออก invoice พักใน ภาษีขายยังไม่ถึงกำหนด แล้วโอนเป็นภาษีขายตามสัดส่วนเมื่อรับชำระ
- นำเข้าผ่านบริการกลาง (ticket 36): import object `AR_RECEIPTS_INTERFACE` เทียบ Fusion Receivables Standard Receipt Import · หน่วยความสำเร็จ: ต่อเงินรับ
- Receipt, ReceiptApplication (reverse ได้), CreditMemo/DebitMemo, Return/ReturnDisposition
- Customer aging

## Acceptance criteria

- [ ] apply เกิน residual ได้ 422
- [ ] เงินที่ยังไม่ apply ไม่ลดยอด invoice
- [ ] financial-only memo ไม่เพิ่มสต็อก
- [ ] คืน 1 ชิ้นแล้วลดหนี้ ยอด AR คงเหลือถูก

## เทสต์ที่ต้องมี

- [ ] e2e: T02 ช่วงรับเงิน/คืน/ลดหนี้, T06 ส่วน receipt

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ล็อก unapplied money และ open items ตามลำดับ
- [ ] reversal จำกัดสิทธิ์
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
