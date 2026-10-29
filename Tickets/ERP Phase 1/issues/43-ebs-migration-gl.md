---
ticket: 43
phase: GL & Tax
module: Migration
screens: ["งานนำเข้า", "Account inquiry / TB"]
refs: ["D01", "P14", "P24"]
blocked_by: [06, 12, 36]
tags:
  - erp/ticket
  - erp/gl-&-tax
---

# 43: ย้ายข้อมูลจาก Oracle EBS: ผังบัญชีและยอดยกมา

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [06 ผังบัญชี รหัสภาษี และการปัดเศษ](06-chart-of-accounts-tax.md), [12 GL: คีย์ Journal, อนุมัติ, Post, Reverse และ Trial Balance](12-gl-journal-entry-posting.md), [36 บริการนำเข้าไฟล์กลาง (แบบ Oracle FBDI)](36-file-based-data-import.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

นำค่า segment และ account combination จากรายงาน Chart of Accounts – Segment Values Listing ของ Oracle EBS R12 และยอดยกมาต่อบัญชี/งวด เข้า ICC ERP ผ่านบริการนำเข้ากลาง แล้วกระทบยอด TB กับ EBS ให้ตรงก่อนเริ่มใช้งาน (P24)

หน้าจอ: งานนำเข้า, Account inquiry / TB
อ้างอิง: D01, P14, P24

## ขอบเขต

- import object GL_SEGMENT_VALUES_INTERFACE และ GL_OPENING_BALANCE_INTERFACE (ผ่าน ticket 36)
- ยอดยกมาลงเป็น Journal category Opening ใน Ledger ของแต่ละบริษัท ต้อง balance ต่อบริษัท
- รายงานเทียบ TB ของ ICC ERP กับ TB ที่ส่งออกจาก EBS ต่อบัญชี
- งานย้ายรายการค้าง AP/AR และทะเบียนสินทรัพย์ทำเป็น import object ใน ticket 21, 25, 28 ภายหลัง

## Acceptance criteria

- [ ] ค่า segment ทุกค่าจาก EBS นำเข้าได้โดยไม่ต้องแก้รหัส
- [ ] TB หลังนำเข้ายอดยกมาตรงกับ EBS ทุกบัญชี
- [ ] นำเข้าซ้ำไฟล์เดิมไม่เกิดยอดซ้ำ
- [ ] ICC และ ITK แยก Ledger ไม่ปนกัน

## เทสต์ที่ต้องมี

- [ ] e2e ด้วยไฟล์สังเคราะห์รูปแบบเดียวกับรายงาน EBS: segment ซ้ำ, combination ผิด cross-validation, ยอดยกมาไม่ balance, กระทบยอด TB

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ไฟล์จาก EBS เป็นข้อมูลการเงินจริง: เก็บในที่เก็บไฟล์ที่จำกัดสิทธิ์และ purge หลังกระทบยอดเสร็จ
- [ ] ใช้ข้อมูลจริงเฉพาะในสภาพแวดล้อมที่ได้รับอนุญาต ไม่ใช้ในเทสต์อัตโนมัติ
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
