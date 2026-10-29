---
ticket: 06
phase: Foundation
module: Master / GL
screens: ["S05 Tax/Accounts"]
refs: ["P08", "D15"]
blocked_by: []
tags:
  - erp/ticket
  - erp/foundation
---

# 06: ผังบัญชี รหัสภาษี และการปัดเศษ

**Status:** resolved
**Type:** task
**Blocked by:** None (can start immediately)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

ข้อมูลหลักบัญชีต่อบริษัท: segment บัญชีและ account combination, รหัสภาษีแบบมี version ตามวันที่มีผล และนโยบายปัดเศษ เพื่อให้ SLA และทุกโมดูลใช้ร่วมกัน

หน้าจอ: S05 Tax/Accounts
อ้างอิง: P08, D15

## ขอบเขต

- รหัสภาษีซื้อต้องห้ามตั้งต้น 5 ประเภท (ค่ารับรอง, รถยนต์นั่ง, เอกสารไม่สมบูรณ์, ไม่เกี่ยวกับกิจการ, อื่น ๆ) recovery 0% และบัญชี "ภาษีซื้อขอคืนไม่ได้" แยก (P20)
- tax code มี recovery rate (P20) และ tax point basis: invoice / payment (P19)
- โครงสร้างผังบัญชีตั้งค่าได้ตาม P14 / ADR-0005: segment, value set, delimiter, label บังคับ Company (balancing) และ Natural account · cross-validation rule (condition + validation) · ค่าจริงนำเข้าจาก Oracle EBS ใน ticket 43
- AccountCombination (company + segments) พร้อมสถานะใช้งาน
- TaxCode/Version (อัตรา, effective dates, บัญชีภาษี)
- Rounding policy ต่อบริษัท
- นำเข้า COA จากไฟล์ตัวอย่าง (CSV)

## Acceptance criteria

- [x] ลงบัญชีด้วย combination ที่ปิดใช้งานหรือไม่มีอยู่ถูกปฏิเสธ
- [x] เลือก tax version ตามวันที่เอกสารได้ถูกต้อง
- [x] ICC/ITK มีผังบัญชีของตัวเอง

## เทสต์ที่ต้องมี

- [x] e2e: tax version ข้ามวันมีผล, นำเข้า COA ซ้ำ, combination ข้ามบริษัท

## มาตรฐานโค้ดและความปลอดภัย

- [x] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [x] ไม่ใส่เลขบัญชีหรืออัตราภาษีจริงที่เดาเอง ใช้ข้อมูลทดสอบ
- [ ] ไฟล์นำเข้าตรวจชนิด/ขนาด
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments

- 2026-10-26: ทำเสร็จบน branch `06-chart-of-accounts-tax`
  - ผังบัญชีต่อบริษัท (หนึ่งบริษัทหนึ่งผัง): segment, ป้าย Company/Natural account บังคับ, ตัวคั่น · ค่า segment โหลดเป็นชุดแบบ all-or-nothing และโหลดซ้ำไม่สร้างซ้ำ (`PUT /charts-of-accounts/:id/segment-values`) · กฎ cross-validation (condition + validation) ใช้กับบัญชีที่สร้างใหม่
  - Account combination + `AccountsService.assertUsable()` ให้ทุกโมดูลตรวจบัญชีตอนลงบัญชี (ไม่มี / ปิดใช้งาน / ข้ามบริษัท → 422)
  - รหัสภาษีพร้อม tax point (delivery/invoice/payment) และ version ตามวันที่มีผล (อัตรา, recovery rate, บัญชีภาษี, บัญชีภาษีซื้อขอคืนไม่ได้) ห้ามช่วงวันทับกัน · `TaxService.versionAt()` · ปุ่มสร้างรหัสภาษีซื้อต้องห้ามตั้งต้น 5 ตัว (ไม่ใส่อัตราให้เอง)
  - นโยบายปัดเศษต่อบริษัท · หน้า S05 `/settings/accounts`
  - **เลื่อนไป ticket 36/43:** การอัปโหลดไฟล์ CSV เพราะ CODING_STANDARDS กำหนดให้ทุกไฟล์ผ่านบริการนำเข้ากลาง (ตรวจชนิด/ขนาดที่นั่น) ticket นี้เตรียม service โหลดชุดค่าไว้ให้ import object เรียกแล้ว
- 2026-10-27: ปรับตามคู่มือ ICC (GL-R1, P26, ADR-0007) บน branch `gl-icc-r12-revision`
  - ผังบัญชีชุดเดียวใช้ร่วม ICC/ITK: บริษัทผูกกับผังผ่าน `gl.chart_company` พร้อมค่า segment Company ของตัวเอง (`POST /charts-of-accounts/:id/companies`) · combination สร้างได้เฉพาะค่า Company ของบริษัทนั้น (`company_segment_mismatch`)
  - ค่า segment มีคำอธิบายอังกฤษ, วันที่มีผล, Parent, Allow posting/budgeting, Third-party control, Reconcile และ DFF (`attributes`) · `assertUsable` ตรวจค่าที่ปิด/หมดผล ณ วันที่บัญชี (`value_inactive`)
  - ช่วงลูกของค่า Parent (`PUT/GET /charts-of-accounts/:id/hierarchy`) กันวนซ้ำ
  - คำขอเปิด/แก้บัญชี (`/account-requests`, doc type `GL_ACCOUNT_REQUEST`) ตั้งค่าเมื่ออนุมัติครบ · หน้า `/gl/account-requests`
- 2026-10-28: โครงสร้าง ICC 8 segment (57 หลัก) ตามเอกสารโครงร่าง COA ของ ICC
  - segment มีค่าเริ่มต้น (`default_value`, migration 0016): สร้าง combination โดยเว้น segment ว่างจะใช้ค่าเริ่มต้น เช่น Intercompany 0000, Reserve 000 · segment ที่ไม่มีค่าเริ่มต้นต้องระบุ (`segment_required`) · ค่าเริ่มต้นยาวเกินความยาว segment ไม่ได้
  - ค่า segment มีเว้นวรรคด้านในได้ (ชื่อ Product เช่น "BSC SHOES") ยังห้ามตัวคั่น
  - หน้า S05 มีปุ่ม "ใช้โครงสร้าง ICC 8 segment" และแสดงค่าเริ่มต้น
  - ฐานข้อมูล dev มีผัง `ICC_COA` 8 segment แล้ว (เฉพาะโครงสร้าง ยังไม่มีค่าและยังไม่ผูกบริษัท เพราะ dev มีแต่บริษัท TST)
  - เอกสารต้นทางมีตัวอย่างยาวไม่ตรงความยาว (Department "GAA1112" 7 หลัก, Account "5100000012" 10 หลัก) ต้องยืนยันกับฝ่ายบัญชี
