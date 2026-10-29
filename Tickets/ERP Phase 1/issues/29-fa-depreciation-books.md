---
ticket: 29
phase: FA
module: Fixed Assets
screens: ["FAI04 Depreciation Preview/Run"]
refs: ["D11", "P09", "T07"]
blocked_by: [28, 37]
tags:
  - erp/ticket
  - erp/fa
---

# 29: ค่าเสื่อมราคา Corporate/Tax Book

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [28 ตั้งสินทรัพย์จาก AP และ CIP](28-fa-capitalization-cip.md), [37 SLA: แปลงเหตุการณ์จากโมดูลย่อยเป็น Journal](37-sla-subledger-accounting.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

คำนวณค่าเสื่อมเส้นตรงรายวันลงบัญชีรายงวด แยก Corporate/Tax book โดย Tax book ไม่ post GL, preview ไม่เปลี่ยนยอด, finalize ล็อก asset-book-period

หน้าจอ: FAI04 Depreciation Preview/Run
อ้างอิง: D11, P09, T07

## ขอบเขต

- AssetBook, DepreciationRun/Line
- day-count basis (365/fiscal-day), leap year, start/stop, mid-period ตาม Asset Book (P09)

## Acceptance criteria

- [ ] ค่าเสื่อมไม่เกิน depreciable amount, NBV ไม่ติดลบ
- [ ] run ซ้ำงวดเดิมไม่ได้
- [ ] Tax book ไม่สร้าง GL

## เทสต์ที่ต้องมี

- [ ] e2e: ชุด boundary (leap year, กลางงวด, residual), run ซ้ำ (T07)

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] finalize เฉพาะ asset manager
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
