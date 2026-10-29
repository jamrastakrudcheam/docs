---
ticket: 28
phase: FA
module: Fixed Assets
screens: ["FAI01 Capitalization Queue", "FAI02 Asset Register", "FAI03 CIP Allocation"]
refs: ["D11", "T07"]
blocked_by: [21, 36]
tags:
  - erp/ticket
  - erp/fa
---

# 28: ตั้งสินทรัพย์จาก AP และ CIP

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [21 ใบแจ้งหนี้เจ้าหนี้ (AP) และ Matching/Hold](21-ap-invoice-matching.md), [36 บริการนำเข้าไฟล์กลาง (แบบ Oracle FBDI)](36-file-based-data-import.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

คิว AP distribution ที่เป็นทุน แบ่งไปหลายสินทรัพย์ได้โดยรวมไม่เกินต้นทุน, ทะเบียนสินทรัพย์ และ CIP ที่โอนต้นทุนไปหลายสินทรัพย์พร้อมหลักฐาน

หน้าจอ: FAI01 Capitalization Queue, FAI02 Asset Register, FAI03 CIP Allocation
อ้างอิง: D11, T07

## ขอบเขต

- นำเข้าผ่านบริการกลาง (ticket 36): import object `FA_MASS_ADDITIONS` เทียบ Fusion Fixed Asset Mass Additions Import · หน่วยความสำเร็จ: ต่อบรรทัด เข้า capitalization queue
- CapitalizationSource, AssetCostAllocation, Asset, CIPProject/Cost/Allocation
- `POST /capitalizations`, `/cip-projects/{id}/allocate`

## Acceptance criteria

- [ ] ตั้งต้นทุนเดิมซ้ำไม่ได้
- [ ] ผลรวมการแบ่งเท่ากับต้นทุน
- [ ] ทะเบียนที่มีบัญชีแล้วลบไม่ได้

## เทสต์ที่ต้องมี

- [ ] e2e: ส่วนตั้งสินทรัพย์ของ T07

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] แยก maker/checker การ post
- [ ] audit การแบ่งต้นทุน
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
