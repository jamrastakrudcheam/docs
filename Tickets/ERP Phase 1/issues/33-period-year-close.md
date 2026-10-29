---
ticket: 33
phase: Close
module: Close
screens: ["CL01 Close Cockpit", "CL02 Adjustment", "CL03 Year Close/Opening"]
refs: ["D13", "P12", "T10"]
blocked_by: [17, 22, 26, 29, 31, 32]
tags:
  - erp/ticket
  - erp/close
---

# 33: ปิดงวด งวดปรับปรุง และปิดปี

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [17 ปิดงวดสต็อกต่อ Org](17-inventory-period-close.md), [22 เสนอจ่ายและจ่ายเงิน (จำลอง)](22-payments.md), [26 รับชำระ ตัดหนี้ ลดหนี้/เพิ่มหนี้ และรับคืนสินค้า](26-receipts-memos-returns.md), [29 ค่าเสื่อมราคา Corporate/Tax Book](29-fa-depreciation-books.md), [31 บัญชีธนาคารและกระทบยอด (ไฟล์ตัวอย่าง)](31-cash-management-reconcile.md), [32 Recurring, Allocation และ Reversal](32-recurring-allocation-reversal.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

Close cockpit แสดงสิ่งที่บล็อกการปิดของทุกโมดูล, ปิด subledger ก่อน GL โดยล็อกกับการ post, งวดปรับปรุงสำหรับรายการผู้สอบบัญชี และปิดปีที่สร้าง retained earnings/opening batch ครั้งเดียว

หน้าจอ: CL01 Close Cockpit, CL02 Adjustment, CL03 Year Close/Opening
อ้างอิง: D13, P12, T10

## ขอบเขต

- precheck ภาษี: สินค้าส่งมอบแล้วแต่ยังไม่ออกใบกำกับภายในเดือน ต้องเตือนและรวมภาษีขายของเดือนนั้น (P19)
- precheck: ยอด Shipped-not-billed ต้องเท่ามูลค่า shipment ที่ยังไม่ออกบิล (ADR-0006) และยอดภาษียังไม่ถึงกำหนดต้องเท่ารายการค้าง (P19)
- CloseRun/Check, YearCloseRun, OpeningBalance
- reopen แบบควบคุม (สิทธิ์, เหตุผล, audit, report revision)

## Acceptance criteria

- [ ] post พร้อมกับการปิดไม่เล็ดลอดเข้างวดที่ปิด
- [ ] ปิดปีซ้ำไม่ได้ ได้ batch เดียวต่อบริษัท/ปี
- [ ] ปิดถาวรเปิดกลับไม่ได้

## เทสต์ที่ต้องมี

- [ ] e2e: T10 ครบ

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] reopen ต้องมีเหตุผลและ audit
- [ ] แยกสิทธิ์ preview กับ finalize
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
