---
ticket: 32
phase: Close
module: GL
screens: ["G04 Recurring/Allocation", "G05 Reversal"]
refs: ["D13"]
blocked_by: [12]
tags:
  - erp/ticket
  - erp/close
---

# 32: Recurring, Allocation และ Reversal

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [12 GL: คีย์ Journal, อนุมัติ, Post, Reverse และ Trial Balance](12-gl-journal-entry-posting.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

รายการบัญชีประจำงวดและการปันส่วนจาก template version: preview → approve → post, ไม่ rerun ทับรายการที่ post แล้ว และการกลับรายการที่เชื่อมต้นทาง

หน้าจอ: G04 Recurring/Allocation, G05 Reversal
อ้างอิง: D13

## ขอบเขต

- Template version, run key unique ต่อ template-period
- STAT/driver evidence สำหรับ allocation

## Acceptance criteria

- [ ] ยอดต้นทาง = ปลายทาง
- [ ] run ซ้ำงวดเดิมไม่ได้
- [ ] reversal เชื่อมกับ journal เดิม

## เทสต์ที่ต้องมี

- [ ] e2e: allocation balance, rerun, reversal

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ผู้สร้าง template ไม่อนุมัติ run ของตัวเอง
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
