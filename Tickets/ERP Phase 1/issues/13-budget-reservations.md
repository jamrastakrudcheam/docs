---
ticket: 13
phase: Foundation
module: Budget
screens: ["B01 Budget Versions", "B02 Budget Detail", "B03 Reservation Ledger", "B04 Transfer/Increase"]
refs: ["D08", "P05", "T04"]
blocked_by: [03, 04, 06]
tags:
  - erp/ticket
  - erp/foundation
---

# 13: งบประมาณและการจองงบ

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [03 Workflow อนุมัติและ Approval Inbox](03-approval-workflow-inbox.md), [04 ปฏิทินบัญชีและงวดของแต่ละโมดูล](04-fiscal-calendar-periods.md), [06 ผังบัญชี รหัสภาษี และการปัดเศษ](06-chart-of-accounts-tax.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

งบประมาณแบบ Absolute: version ที่อนุมัติ, บรรทัดงบตามมิติ, ledger การจอง/ใช้จริง และคำขอโอน/เพิ่มงบ เพื่อให้ PR/PO/AP เรียกใช้

หน้าจอ: B01 Budget Versions, B02 Budget Detail, B03 Reservation Ledger, B04 Transfer/Increase
อ้างอิง: D08, P05, T04

## ขอบเขต

- BudgetVersion → BudgetLine (company/OU/year/period/cost centre/account/category: stock/capital/expense)
- BudgetReservation → ReservationMovement (reserve, transfer-to-PO, release, actual)
- service: reserve/transfer/release/convertToActual ที่ล็อก budget bucket
- B04 ผ่าน workflow

## Acceptance criteria

- [ ] available = approved + changes − encumbrance − actual
- [ ] จองเกินยอดคงเหลือถูกปฏิเสธ (Absolute)
- [ ] จองพร้อมกันสองรายการที่รวมกันเกินยอด ผ่านได้แค่รายการเดียว
- [ ] UI แก้ยอด reservation ตรงไม่ได้

## เทสต์ที่ต้องมี

- [ ] e2e: ยอดพอดีวงเงิน, เกินวงเงิน, concurrency, โอนงบผ่านอนุมัติ (ส่วนหนึ่งของ T04)

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] budget owner ไม่แก้ reservation ตรง
- [ ] ล็อก bucket ตามลำดับ key
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
