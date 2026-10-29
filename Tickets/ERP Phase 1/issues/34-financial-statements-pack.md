---
ticket: 34
phase: Close
module: Statements
screens: ["FS01 Account Mapping", "FS02 Financial Pack", "FS03 Review/Finalize"]
refs: ["D14", "D15", "P10", "T11"]
blocked_by: [33]
tags:
  - erp/ticket
  - erp/close
---

# 34: งบการเงินและ Financial Pack

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [33 ปิดงวด งวดปรับปรุง และปิดปี](33-period-year-close.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

TB, งบฐานะการเงิน, กำไรขาดทุน, กระแสเงินสดวิธี indirect และการเปลี่ยนแปลงส่วนของเจ้าของ แยก ICC/ITK พร้อมยอดเปรียบเทียบ, mapping version, cutoff, ตารางประกอบ และ sign-off ส่งออก PDF/Excel

หน้าจอ: FS01 Account Mapping, FS02 Financial Pack, FS03 Review/Finalize
อ้างอิง: D14, D15, P10, T11

## ขอบเขต

- FinancialMappingVersion/Line, ReportRun, FinancialPack/Snapshot, SupplementalSchedule, ReviewSignoff

## Acceptance criteria

- [ ] ICC ไม่ปน ITK
- [ ] opening + movement = closing
- [ ] แก้ source/mapping หลัง final สร้าง revision ใหม่ ชุดเดิมยังอยู่
- [ ] ยอดในไฟล์ส่งออกตรงกับหน้าจอ

## เทสต์ที่ต้องมี

- [ ] e2e: T11 ครบ

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] export เป็น audit แยกจากการ post
- [ ] บังคับ scope company/ledger ทุกรายงาน
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
