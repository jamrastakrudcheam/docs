---
ticket: 09
phase: Foundation
module: Master / Customer
screens: ["M01 Customer/Site/Credit"]
refs: ["D09", "P03"]
blocked_by: [03, 06]
tags:
  - erp/ticket
  - erp/foundation
---

# 09: ลูกค้า ไซต์ และวงเงินเครดิต

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [03 Workflow อนุมัติและ Approval Inbox](03-approval-workflow-inbox.md), [06 ผังบัญชี รหัสภาษี และการปัดเศษ](06-chart-of-accounts-tax.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

ข้อมูลลูกค้าพร้อม Bill-to/Ship-to site, วงเงินเครดิต, เงื่อนไขออกบิล (หลัง Ship Confirm หรือหลัง POD ต่อไซต์) ผ่าน maker/checker

หน้าจอ: M01 Customer/Site/Credit
อ้างอิง: D09, P03

## ขอบเขต

- Customer, CustomerSite (billing trigger ตาม P03), CreditProfile
- draft → submit → approve ผ่าน ticket 03
- หน้า M01 TH/EN

## Acceptance criteria

- [ ] ลูกค้าที่ยังไม่อนุมัติใช้ในเอกสารขายไม่ได้
- [ ] แก้วงเงินต้องอนุมัติใหม่
- [ ] ไซต์มี billing trigger ที่ตั้งค่าได้

## เทสต์ที่ต้องมี

- [ ] e2e: maker/checker, แก้วงเงิน, ไซต์ไม่ active

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ข้อมูลติดต่อเป็นข้อมูลส่วนบุคคล ไม่ใส่ใน log/URL
- [ ] แยกสิทธิ์แก้วงเงินกับฝ่ายขาย
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
