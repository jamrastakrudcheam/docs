---
ticket: 31
phase: CM
module: Cash Management
screens: ["CM01 Bank Account", "CM02 Statement Import", "CM03 Reconcile"]
refs: ["D03", "T10"]
blocked_by: [22, 26, 36]
tags:
  - erp/ticket
  - erp/cm
---

# 31: บัญชีธนาคารและกระทบยอด (ไฟล์ตัวอย่าง)

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [22 เสนอจ่ายและจ่ายเงิน (จำลอง)](22-payments.md), [26 รับชำระ ตัดหนี้ ลดหนี้/เพิ่มหนี้ และรับคืนสินค้า](26-receipts-memos-returns.md), [36 บริการนำเข้าไฟล์กลาง (แบบ Oracle FBDI)](36-file-based-data-import.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

บัญชีธนาคารของนิติบุคคล, นำเข้า statement ตัวอย่างพร้อม fingerprint กันนำเข้าซ้ำ และกระทบยอดกับ payment/receipt ทั้งเต็มและบางส่วน

หน้าจอ: CM01 Bank Account, CM02 Statement Import, CM03 Reconcile
อ้างอิง: D03, T10

## ขอบเขต

- นำเข้าผ่านบริการกลาง (ticket 36): import object `CE_STATEMENT_HEADERS_INT + CE_STATEMENT_LINES_INT + CE_STMT_BALANCES_INT` เทียบ Fusion Bank Statement Data Import · หน่วยความสำเร็จ: ทั้ง statement (Fusion นำเข้าแบบทั้งหมดหรือไม่มีเลย)
- BankAccount, BankStatement/Line, ReconciliationGroup/Match/Adjustment
- match/finalize/reverse

## Acceptance criteria

- [ ] ไฟล์เดิมนำเข้าซ้ำไม่ได้
- [ ] กระทบยอดแล้ว clearing account ล้าง
- [ ] ยอดรวมแต่ละกลุ่ม balance

## เทสต์ที่ต้องมี

- [ ] e2e: import ซ้ำ, partial match, reverse

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ไฟล์ตัวอย่างไม่มีข้อมูลธนาคารจริง
- [ ] ตรวจชนิด/ขนาดไฟล์
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
