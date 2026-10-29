---
ticket: 02
phase: Foundation
module: Cross-cutting
screens: ["Login"]
refs: ["D03", "CODING_STANDARDS §5"]
blocked_by: []
tags:
  - erp/ticket
  - erp/foundation
---

# 02: เสริมความปลอดภัยพื้นฐาน

**Status:** ready-for-agent
**Type:** task
**Blocked by:** None (can start immediately)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

ปิดช่องโหว่พื้นฐานที่ยังเหลือจากรอบ login ก่อนเพิ่มโมดูลธุรกรรม เพื่อให้ทุก ticket ต่อจากนี้ได้การป้องกันเหล่านี้โดยอัตโนมัติ

หน้าจอ: Login
อ้างอิง: D03, CODING_STANDARDS §5

## ขอบเขต

- จำกัดจำนวนครั้ง login ผิดต่อชื่อผู้ใช้และต่อ IP แล้วหน่วงเวลา
- security headers (CSP, X-Frame-Options, Referrer-Policy, no-sniff) ทั้ง API และเว็บ
- จำกัดขนาด body ของ request
- เพิ่ม `npm audit --audit-level=high` ใน CI
- ย้ายรหัสผ่าน PostgreSQL ใน docker-compose ไป `.env`

## Acceptance criteria

- [ ] login ผิดเกินเกณฑ์ได้ 429 พร้อมข้อความภาษาไทย และไม่บอกว่าชื่อผู้ใช้มีอยู่หรือไม่
- [ ] response มี security headers ครบ
- [ ] body ใหญ่เกินกำหนดได้ 413
- [ ] CI ล้มเมื่อมีช่องโหว่ระดับ high
- [ ] docker-compose ไม่มีรหัสผ่านตรง ๆ

## เทสต์ที่ต้องมี

- [ ] e2e: login ผิดซ้ำได้ 429 แล้วกลับมาใช้ได้หลังหมดเวลา
- [ ] e2e: ตรวจ header สำคัญใน response

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ค่าเกณฑ์ตั้งค่าได้ผ่าน env ไม่ hard-code
- [ ] ไม่ log รหัสผ่านที่กรอกผิด
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
