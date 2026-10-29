---
ticket: 04
phase: Foundation
module: GL / Period
screens: ["S04 Numbering/Period (ส่วนงวด)"]
refs: ["D13", "P08", "P12"]
blocked_by: [01]
tags:
  - erp/ticket
  - erp/foundation
---

# 04: ปฏิทินบัญชีและงวดของแต่ละโมดูล

**Status:** resolved
**Type:** task
**Blocked by:** [01 ตัดสินใจประเด็นออกแบบที่ค้าง](01-resolve-open-design-decisions.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

กำหนดปีบัญชีและงวดต่อ Ledger และสถานะงวดแยกต่อโมดูล (AP/AR/CM/INV-COST/FA/GL) เพื่อให้ทุกการลงบัญชีตรวจได้ว่างวดเปิดอยู่

หน้าจอ: S04 Numbering/Period (ส่วนงวด)
อ้างอิง: D13, P08, P12

## ขอบเขต

- งวดปรับปรุงปลายปีตั้งได้ 0–2 งวดต่อปฏิทิน ค่าเริ่มต้นงวด 13 (ปรับปรุงปิดบัญชี) และ 14 (ผู้สอบบัญชี) ตาม P15 · จำนวนงวด Future entry ต่อ Ledger
- FiscalCalendar ต่อ Ledger, FiscalPeriod (ปกติ + งวดปรับปรุงตามผล ticket 01)
- ModulePeriod: สถานะ never-opened/open/closed/permanently-closed ต่อโมดูลและ Org
- service กลาง `assertOpen(module, ledger/org, gl_date)` ให้ทุกโมดูลเรียก
- หน้าตั้งค่าปฏิทินและเปิด/ปิดงวด

## Acceptance criteria

- [x] ลงบัญชีในงวดที่ปิดถูกปฏิเสธด้วย 422 ที่บอกงวด
- [x] ปิดถาวรแล้วเปิดกลับไม่ได้
- [x] reopen ต้องมีสิทธิ์ เหตุผล และ audit (P12)
- [x] ICC และ ITK มีปฏิทินแยกกันได้

## เทสต์ที่ต้องมี

- [x] e2e: เปิด/ปิด/reopen, ลงวันที่ขอบงวด, งวดปรับปรุง

## มาตรฐานโค้ดและความปลอดภัย

- [x] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [x] เปิด/ปิดงวดเฉพาะบทบาท GL manager
- [x] ล็อกสถานะงวดกับการ post เพื่อกัน race (T10)
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments

- 2026-10-27: ทำเสร็จบน branch `04-fiscal-calendar-periods`
  - ปฏิทินบัญชีต่อ Ledger: เดือนเริ่มปี, งวดปรับปรุงปลายปี 0–2 งวด (13/14, P15), จำนวนงวดคีย์ล่วงหน้า · เพิ่มปีบัญชีแล้วได้ 12 งวด + งวดปรับปรุง (ชื่อ `FY2027-01` … `FY2027-14`) · หลังมีปีบัญชีแล้วแก้เดือนเริ่มต้น/จำนวนงวดปรับปรุงไม่ได้
  - สถานะต่อโมดูล GL/AP/AR/CM/INV/FA แบบ Fusion: ยังไม่เปิด → คีย์ล่วงหน้า → เปิด → ปิด → ปิดถาวร · เปิดงวดตามลำดับ (งวดปรับปรุงยกเว้น) · reopen ต้องมีเหตุผลและลง audit · ปิดถาวรแล้วกลับไม่ได้ · เปลี่ยนสถานะได้เฉพาะ `gl_accountant` ของบริษัท (System Admin ตั้งปฏิทินแต่ไม่ได้ปิดงวด)
  - `PeriodsService.assertOpen()` ล็อก module period แบบ share ใน transaction ของการลงบัญชี การปิดงวดจึงรอจน posting commit (T10, มีเทสต์)
  - หน้า `/gl/periods` (S04 ส่วนงวด)
  - **เลื่อนไป ticket 17:** สถานะงวด INV ต่อ Inventory Org (ยังไม่มี Inventory Org จนกว่า ticket 08) ตอนนี้ทุกโมดูลเป็นระดับ Ledger
