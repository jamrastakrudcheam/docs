---
ticket: 05
phase: Foundation
module: Numbering
screens: ["S04 Numbering/Period (ส่วนเลขที่)"]
refs: ["P08", "03 Constraints"]
blocked_by: [04]
tags:
  - erp/ticket
  - erp/foundation
---

# 05: เลขที่เอกสาร

**Status:** resolved
**Type:** task
**Blocked by:** [04 ปฏิทินบัญชีและงวดของแต่ละโมดูล](04-fiscal-calendar-periods.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

สร้างเลขที่เอกสารไม่ซ้ำตาม scope (บริษัท/OU), รูปแบบ และปีบัญชี ใช้ร่วมกันทุกโมดูล

หน้าจอ: S04 Numbering/Period (ส่วนเลขที่)
อ้างอิง: P08, 03 Constraints

## ขอบเขต

- DocumentSequence: document type + scope + fiscal year + pattern
- การออกเลขแบบ atomic ใน transaction ของเอกสาร
- หน้าตั้งค่ารูปแบบเลขที่

## Acceptance criteria

- [x] ออกเลขพร้อมกันหลาย request ไม่ซ้ำ
- [x] เลขขึ้นปีใหม่ตามปีบัญชีของ Ledger
- [x] เลขคงเลขศูนย์นำหน้าตามรูปแบบ

## เทสต์ที่ต้องมี

- [x] e2e: ออกเลขพร้อมกัน 20 request ได้เลขไม่ซ้ำ
- [x] e2e: ข้ามปีบัญชี

## มาตรฐานโค้ดและความปลอดภัย

- [x] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [x] แก้รูปแบบได้เฉพาะผู้ดูแล และไม่กระทบเลขที่ออกไปแล้ว
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments

- 2026-10-27: ทำเสร็จบน branch `05-document-numbering`
  - กฎเลขที่ต่อบริษัท (หรือ OU ซึ่งใช้ก่อนกฎทั้งบริษัท) + ประเภทเอกสาร · รูปแบบมี token `{SEQ:n}` (บังคับ) `{FY}` `{FY2}` `{MM}` `{CO}` `{OU}` เช่น `JV{FY2}-{SEQ:6}` → `JV27-000001`
  - ตัวนับแยกต่อปีบัญชีของ Ledger (จากปฏิทิน ticket 04) · `NumberingService.next()` ออกเลขด้วย upsert ใน transaction ของเอกสาร: ไม่ซ้ำ (เทสต์ 20 request พร้อมกัน) และ rollback แล้วเลขคืน ไม่ขาดช่วง · เลขเต็มจำนวนหลักแล้วปฏิเสธ 422
  - แก้รูปแบบได้เฉพาะ System Admin และมีผลกับเลขถัดไปเท่านั้น ตัวนับไม่ย้อน
  - หน้า `/settings/numbering` (S04 ส่วนเลขที่)
- 2026-10-27: เพิ่มรอบนับรายเดือน (`reset`), token `{YY}` และ `{JC}` (เฉพาะ GL_JOURNAL) และกฎเฉพาะ subtype สำหรับเลข Voucher แบบ ICC (P29, GL-R2)
