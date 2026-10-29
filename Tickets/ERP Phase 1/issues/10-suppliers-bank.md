---
ticket: 10
phase: Foundation
module: Master / Supplier
screens: ["M02 Supplier/Site/Bank"]
refs: ["03 Authorization matrix"]
blocked_by: [03, 06]
tags:
  - erp/ticket
  - erp/foundation
---

# 10: ผู้ขาย ไซต์ และบัญชีธนาคาร

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [03 Workflow อนุมัติและ Approval Inbox](03-approval-workflow-inbox.md), [06 ผังบัญชี รหัสภาษี และการปัดเศษ](06-chart-of-accounts-tax.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

ข้อมูลผู้ขายและไซต์ พร้อมบัญชีธนาคารที่แก้ได้เฉพาะผู้ดูแลที่ได้รับมอบหมาย แยกจากผู้สร้าง Supplier

หน้าจอ: M02 Supplier/Site/Bank
อ้างอิง: 03 Authorization matrix

## ขอบเขต

- Supplier, SupplierSite, SupplierBankAccount
- บทบาท bank-detail custodian
- หน้า M02 แสดงเลขบัญชีแบบ mask

## Acceptance criteria

- [ ] ผู้สร้าง Supplier แก้บัญชีธนาคารไม่ได้
- [ ] API ไม่คืนเลขบัญชีเต็มให้บทบาทที่ไม่ได้รับสิทธิ์
- [ ] แก้บัญชีธนาคารต้องอนุมัติและมี audit

## เทสต์ที่ต้องมี

- [ ] e2e: mask ตามบทบาท, SoD ผู้สร้าง vs custodian

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] mask เลขบัญชีที่ API ไม่ใช่แค่หน้าเว็บ
- [ ] ไฟล์ตัวอย่างไม่มีข้อมูลธนาคารจริง
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
