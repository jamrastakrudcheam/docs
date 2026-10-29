---
ticket: 21
phase: P2P
module: AP
screens: ["A01 AP Invoice", "A02 Matching/Hold", "PO06 Match Workbench"]
refs: ["P05", "P06", "T01", "T08"]
blocked_by: [20, 36]
tags:
  - erp/ticket
  - erp/p2p
---

# 21: ใบแจ้งหนี้เจ้าหนี้ (AP) และ Matching/Hold

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [20 รับสินค้าและคืนผู้ขาย](20-receiving-returns.md), [36 บริการนำเข้าไฟล์กลาง (แบบ Oracle FBDI)](36-file-based-data-import.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

บันทึก AP invoice จับคู่กับ receipt/PO ตาม tolerance ต่อบริษัท ถ้าเกินให้ Hold เมื่อ post ตั้งหนี้, ลง SLA และเปลี่ยนงบจาก encumbrance เป็น actual ครั้งเดียว

หน้าจอ: A01 AP Invoice, A02 Matching/Hold, PO06 Match Workbench
อ้างอิง: P05, P06, T01, T08

## ขอบเขต

- นำเข้าผ่านบริการกลาง (ticket 36): import object `AP_INVOICES_INTERFACE + AP_INVOICE_LINES_INTERFACE` เทียบ Fusion Payables Standard Invoice Import · หน่วยความสำเร็จ: ต่อใบ (ใบที่ถูกเข้าได้ ใบที่ผิดค้างให้แก้)
- APInvoice/Line, APInvoiceLineMatch
- tolerance policy (ราคา/จำนวน, ก่อน/หลังภาษี) ต่อบริษัท
- SLA: Dr GRNI (+VAT/variance) / Cr AP
- budget convertToActual (P05)

## Acceptance criteria

- [ ] เกิน tolerance ถูก Hold และ post ไม่ได้จนปลด
- [ ] งบไม่ถูกหักซ้ำจาก receipt + AP + GL
- [ ] คืนของ/ยกเลิกลด matching capacity

## เทสต์ที่ต้องมี

- [ ] e2e: match บางส่วน, price variance, hold/release, budget actual (T01, T04)

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ปลด hold ต้องมีสิทธิ์และเหตุผล
- [ ] Idempotency-Key ที่ post
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
