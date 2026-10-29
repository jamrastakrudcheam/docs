---
ticket: 35
phase: Close
module: Home
screens: ["H01 Overview", "H03 Search/Document Trace"]
refs: ["D13", "T11"]
blocked_by: [21, 25]
tags:
  - erp/ticket
  - erp/close
---

# 35: ติดตามเส้นทางเอกสารและ KPI หน้าภาพรวม

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [21 ใบแจ้งหนี้เจ้าหนี้ (AP) และ Matching/Hold](21-ap-invoice-matching.md), [25 ออกบิลและใบแจ้งหนี้ลูกหนี้ (AR)](25-billing-ar-invoice.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

ค้นหาเอกสารทุกโมดูลและไล่เส้นทางต้นทาง → ปลายทาง → รายการบัญชี และแทนตัวเลข "—" ในหน้าภาพรวมด้วยยอดจริงตามบริษัท/OU ที่เลือก

หน้าจอ: H01 Overview, H03 Search/Document Trace
อ้างอิง: D13, T11

## ขอบเขต

- global search ที่เคารพ scope
- document chain view
- KPI: AR/AP คงค้าง, มูลค่าสต็อก, งานรออนุมัติ, close readiness

## Acceptance criteria

- [ ] ค้นหาไม่คืนเอกสารนอกสิทธิ์
- [ ] KPI ตรงกับรายงานของโมดูล
- [ ] เปลี่ยนบริษัทแล้วตัวเลขเปลี่ยนตาม

## เทสต์ที่ต้องมี

- [ ] e2e: search scope, ยอด KPI เทียบรายงาน

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] filter สิทธิ์ก่อน aggregate
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
