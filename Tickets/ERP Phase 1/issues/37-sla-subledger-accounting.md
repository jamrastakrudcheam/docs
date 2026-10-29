---
ticket: 37
phase: GL & Tax
module: SLA
screens: ["G01 SLA Events/Exceptions", "G02 Rule Versions"]
refs: ["D13", "P12", "T08"]
blocked_by: [12]
tags:
  - erp/ticket
  - erp/gl-&-tax
---

# 37: SLA: แปลงเหตุการณ์จากโมดูลย่อยเป็น Journal

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [12 GL: คีย์ Journal, อนุมัติ, Post, Reverse และ Trial Balance](12-gl-journal-entry-posting.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

บริการบัญชีกลางที่โมดูลอื่น (สต็อก, AP, AR, FA, CM) เรียกใน transaction ของตัวเอง: รับ accounting event แล้วสร้าง Journal ที่ balance ตาม rule version ที่มีผล ณ วันที่ และ Post ทันที ถ้า mapping ไม่พร้อมให้คำสั่งต้นทาง rollback

หน้าจอ: G01 SLA Events/Exceptions, G02 Rule Versions
อ้างอิง: D13, P12, T08

## ขอบเขต

- AccountingEvent (unique ต่อ company/source type/source id/revision/event type), AccountingEntry/Line, AccountingEntryReference
- SLA rule version: effective dates, account mapping, tax/amount source, rounding
- Preview → Validated/Exception → Posted, source = ชื่อโมดูล
- หน้า G01 ดู event/exception, G02 จัดการ rule version

## Acceptance criteria

- [ ] mapping หายหรือ entries ไม่ balance ทำให้คำสั่งต้นทาง rollback ทั้งหมด
- [ ] event เดียวกันสร้าง Journal ซ้ำไม่ได้
- [ ] ทุก Journal จาก SLA ไล่กลับถึงเอกสารต้นทางได้
- [ ] Journal จาก SLA แก้ในหน้า GL ไม่ได้

## เทสต์ที่ต้องมี

- [ ] e2e ด้วยโมดูลทดสอบ (fake source): สร้าง event → Journal posted, mapping หาย → rollback, event ซ้ำ, rule version ข้ามวันที่

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] แก้ rule version เฉพาะ GL manager และเก็บ version history
- [ ] ไม่มี endpoint ให้โมดูลภายนอกยิง event ตรง ใช้ service ภายในเท่านั้น
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
