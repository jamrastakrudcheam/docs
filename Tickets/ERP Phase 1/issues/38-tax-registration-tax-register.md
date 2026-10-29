---
ticket: 38
phase: GL & Tax
module: Tax
screens: ["S05 Tax (สถานประกอบการ, WHT code)", "Tax register (ใหม่)"]
refs: ["D14", "P08"]
blocked_by: [12, 06]
tags:
  - erp/ticket
  - erp/gl-&-tax
---

# 38: สถานประกอบการทางภาษีและทะเบียนภาษี (Tax register)

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [12 GL: คีย์ Journal, อนุมัติ, Post, Reverse และ Trial Balance](12-gl-journal-entry-posting.md), [06 ผังบัญชี รหัสภาษี และการปัดเศษ](06-chart-of-accounts-tax.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

ทะเบียนรายการภาษีกลาง (เทียบ Fusion Tax Reporting Ledger) ที่ทุกรายงานภาษีอ่าน: เก็บ VAT ซื้อ/ขาย และ WHT จากบรรทัด Journal ที่ Post แล้ว (ภายหลังจาก AP/AR ผ่าน SLA) พร้อมสถานประกอบการ/สาขาของแต่ละนิติบุคคล (gl-process §8)

หน้าจอ: S05 Tax (สถานประกอบการ, WHT code), Tax register (ใหม่)
อ้างอิง: D14, P08

## ขอบเขต

- ตาราง WHT code ตั้งต้นตาม tax review P21 ข้อ 4 · ภาษีซื้อเลยกำหนด 6 เดือนส่งเข้า approval ก่อนโอนเป็นค่าใช้จ่าย (P20)
- กฎภาษีตามคำตอบใน [tax-review-p18-p21.md](../tax-review-p18-p21.md) (ตอบโดย Claude แทนทีมภาษี — ให้ผู้เชี่ยวชาญภาษีทบทวนก่อนยื่นแบบจริง)
- tax point basis (P19): รายการ payment basis เข้าทะเบียนภาษีในเดือนที่รับ/จ่ายชำระ ไม่ใช่เดือนใบกำกับ · ส่วนที่ขอคืนไม่ได้ตาม recovery rate (P20) ไม่เข้าทะเบียนภาษีซื้อ
- TaxRegistration (Legal Entity + เลขสาขา 5 หลัก + ที่อยู่ + ยื่น ภ.พ.30 แยก/รวม)
- WHT code: ประเภทเงินได้ตามมาตรา 40, อัตรา, แบบ (ภ.ง.ด.3/53), effective dates
- TaxTransaction: ประเภท (VAT_INPUT, VAT_OUTPUT, WHT), ฐาน, ภาษี, ใบกำกับ, คู่ค้า (ชื่อ, เลขผู้เสียภาษี, สาขา, ที่อยู่), เดือนภาษี/เดือนที่ขอเครดิต, อ้าง Journal line
- เขียน tax register ใน transaction เดียวกับการ Post Journal, กลับรายการเมื่อ Journal ถูก reverse
- หน้าดู/ค้น tax register ต่อบริษัท สถานประกอบการ เดือน

## Acceptance criteria

- [ ] ข้อมูลภาษีไม่ครบหรือรูปแบบผิด (เลขผู้เสียภาษี 13 หลัก, สาขา 5 หลัก) Complete Journal ไม่ได้
- [ ] ยอดภาษีใน register = ยอดบรรทัดบัญชีภาษีของ Journal
- [ ] ภาษี ≠ ฐาน × อัตรา เกิน tolerance ถูกปฏิเสธ
- [ ] เดือนที่ขอเครดิตภาษีซื้อก่อนเดือนใบกำกับหรือเกินระยะที่ตั้ง (เริ่มต้น 6 เดือน) ถูกปฏิเสธ
- [ ] ใบกำกับเลขเดิมของผู้ขายรายเดิมขอเครดิตซ้ำไม่ได้
- [ ] Reverse Journal แล้วรายการภาษีถูกกลับในเดือนของ reversal

## เทสต์ที่ต้องมี

- [ ] e2e: Journal ที่มี VAT/WHT → register, ข้อมูลผิดรูปแบบ, ใบกำกับซ้ำ, ขอเครดิตเกินระยะ, reverse

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] เลขผู้เสียภาษีของบุคคลธรรมดาเป็นข้อมูลส่วนบุคคล: ไม่ใส่ใน log/URL
- [ ] อัตราภาษีและระยะเวลาเป็นค่าตั้งค่า มี effective date ไม่ hard-code
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
