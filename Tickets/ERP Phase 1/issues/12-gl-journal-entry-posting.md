---
ticket: 12
phase: Foundation
module: GL
screens: ["G03 Journal", "G05 Reversal", "Account inquiry (ใหม่)", "CL04 TB"]
refs: ["D13", "D07", "P08", "P12", "T08", "T11"]
blocked_by: [03, 04, 05, 06, 07]
tags:
  - erp/ticket
  - erp/foundation
---

# 12: GL: คีย์ Journal, อนุมัติ, Post, Reverse และ Trial Balance

**Status:** claimed
**Type:** task
**Blocked by:** [03 Workflow อนุมัติและ Approval Inbox](03-approval-workflow-inbox.md), [04 ปฏิทินบัญชีและงวดของแต่ละโมดูล](04-fiscal-calendar-periods.md), [05 เลขที่เอกสาร](05-document-numbering.md), [06 ผังบัญชี รหัสภาษี และการปัดเศษ](06-chart-of-accounts-tax.md), [07 Idempotency และหน้าดู Audit](07-idempotency-audit-viewer.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

โมดูล GL แบบ Oracle Fusion: คีย์ Journal Batch → Journal → Lines ผ่านหน้าจอ, ตรวจ balance/งวด/บัญชี, ส่งอนุมัติตามกฎ (ticket 03), Post ลงตารางยอด, กลับรายการ (รวม auto-reverse) และดูยอดผ่าน Account inquiry/Trial Balance ลำดับขั้นทั้งหมดอยู่ใน [gl-process.md](../gl-process.md)

หน้าจอ: G03 Journal, G05 Reversal, Account inquiry (ใหม่), CL04 TB
อ้างอิง: D13, D07, P08, P12, T08, T11

## ขอบเขต

- กฎอนุมัติตาม P17: Journal source Manual/Spreadsheet อนุมัติ ≥1 ขั้นโดย GL manager ที่ไม่ใช่ผู้เตรียม, ขั้น 2 ตามวงเงินต่อ Ledger, Journal จาก SLA ไม่อนุมัติใน GL
- JournalBatch, Journal, JournalLine, JournalSource, JournalCategory (auto-reverse, ต้องอนุมัติ), GLBalance (ledger + account + period)
- สถานะ: Incomplete → Unposted → Pending approval → Ready to post → Posted / Error (gl-process §3)
- หน้าจอคีย์ Journal: header (Ledger จากบริษัทที่เลือก, Category, Accounting date, คำอธิบาย, Reference), ตารางบรรทัดคีย์ต่อเนื่องด้วยคีย์บอร์ด, ยอดรวม Dr/Cr และผลต่างตลอดเวลา, แผงข้อมูลภาษีต่อบรรทัด (เก็บข้อมูลไว้ให้ ticket 38)
- คำสั่ง: save, complete, submit, approve/return (ผ่าน ticket 03), post, reverse, delete (เฉพาะที่ยังไม่ submit)
- Account inquiry: ยอดต่อบัญชี/งวด เจาะลงถึง Journal และ Trial Balance (opening, period Dr/Cr, closing) ส่งออก Excel
- เลขที่ใบสำคัญออกตอน Complete ผ่าน ticket 05

## Acceptance criteria

- [ ] Journal ไม่ balance หรือมีบรรทัดที่มีทั้ง Dr และ Cr Complete ไม่ได้ และบอกบรรทัดที่ผิด
- [ ] บัญชีหัวรวม ปิดใช้งาน หรือต่างบริษัท ใช้ไม่ได้
- [ ] accounting date ในงวด Future Entry Complete ได้แต่ Post ไม่ได้, งวด Closed ทำไม่ได้ทั้งคู่
- [ ] แก้ Journal หลัง Complete แล้วกลับเป็น Incomplete และล้างการอนุมัติ
- [ ] ผู้เตรียมอนุมัติ Journal ของตัวเองไม่ได้ ถ้าเข้ากฎต้องอนุมัติจะ Post ไม่ได้จนกว่าจะอนุมัติ
- [ ] Posted แก้/ลบไม่ได้ กลับรายการได้ครั้งเดียว ด้วยการสลับ Dr/Cr ในวันที่ที่เลือก
- [ ] Category auto-reverse สร้าง reversal ในงวดถัดไป
- [ ] Post ซ้ำ (double-click, สองแท็บ) ยอดไม่เพิ่มซ้ำ
- [ ] TB ทั้งบริษัท Dr = Cr และไม่ปน ICC/ITK, ยอดใน TB ตรงกับผลรวม Journal ที่ Post แล้ว

## เทสต์ที่ต้องมี

- [ ] e2e: กฎตรวจทุกข้อใน gl-process §4, สถานะครบวงจร, self-approval, post ในงวดปิด, double post, reverse ซ้ำ, auto-reverse, TB balance และ company isolation (บางส่วนของ T08, T11)
- [ ] ตรวจในเบราว์เซอร์: คีย์ Journal 10 บรรทัดด้วยคีย์บอร์ดล้วน, desktop และ tablet

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] Post/Reverse เฉพาะ GL manager, คีย์เฉพาะ GL accountant ตามบริษัท/OU ที่มีสิทธิ์
- [ ] ยอดรวมคำนวณฝั่ง server ไม่เชื่อยอดจาก browser, จำนวนเงินเป็น string ทศนิยม 2 ตำแหน่ง
- [ ] Idempotency-Key ที่ complete/post/reverse
- [ ] ล็อกสถานะงวดตอน Post กัน race กับการปิดงวด
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments

- 2026-10-27: รอบแรกบน branch `12-gl-journal-entry-posting` (ยังไม่ปิด ticket)
  - Journal: ร่าง → Complete (ออกเลขที่ผ่าน 05, ตรวจงวดผ่าน 04 รับงวด Future Entry) → ส่งอนุมัติ (03, P17) → Post (ล็อกงวด, ตารางยอด `gl.balance`) · แก้หลัง Complete กลับเป็นร่างและยกเลิกการอนุมัติ · ลบได้เฉพาะที่ไม่เคยส่งอนุมัติ · กลับรายการครั้งเดียว (สลับ Dr/Cr) · Category Accrual กลับรายการอัตโนมัติงวดถัดไป · Idempotency-Key ที่ complete/post/reverse
  - บัญชีหัวรวม (`allow_posting = false`) ใช้ลงรายการไม่ได้
  - Trial Balance (ยอดยกมา/งวด/คงเหลือ, ส่งออก Excel) และ Account inquiry เจาะถึง Journal · หน้า `/gl/journals`, `/gl/journals/:id`, `/gl/trial-balance`, `/gl/account-inquiry`
  - เทสต์ e2e 8 กรณี ผ่าน · ทดสอบคีย์ 10 บรรทัดในเบราว์เซอร์ผ่าน JavaScript (pane ไม่วาดจึงกดคีย์จริงไม่ได้) ยังต้องลองด้วยคีย์บอร์ดจริงและบน tablet
  - **ต่อไป:** ผู้ใช้สั่งปรับโมดูล GL ทั้งหมดตามคู่มือ ICC Oracle EBS R12 (ICC-UM-GL-101 … 206) ดู gap analysis ใน `.scratch/erp-phase1/gl-icc-r12-gap.md`
- 2026-10-27: GL-R2 ตามคู่มือ ICC (GL-106.2/106.3/106.6, P29) บน branch `gl-icc-r12-revision`
  - Journal Source เป็น master (`gl.journal_source`): ต้องอนุมัติ / แก้ใน GL ไม่ได้ (frozen) / เก็บอ้างอิงต้นทาง · MANUAL และ SPREADSHEET ปิดการอนุมัติไม่ได้ (P17) · source ที่ไม่ต้องอนุมัติ submit แล้วเป็น Ready to post ทันที
  - Category เพิ่ม/ปิดได้ พร้อมรหัส Voucher (ICC 11–15, 2, 31–35, 41–43) และเกณฑ์กลับรายการ: งวดถัดไป / งวดเดียวกัน / วันถัดไป / ไม่กำหนด, สลับ Dr/Cr หรือกลับเครื่องหมาย, Post รายการกลับอัตโนมัติ
  - เลข Voucher แบบ ICC: numbering token `{JC}` `{YY}` และรอบนับรายเดือน (`reset: monthly`) แยกตัวนับตามรหัส Voucher · กฎเฉพาะรหัส (`doc_subtype`) เช่น interface `2{YY}{MM}{SEQ:4}`
  - หน้า S06 `/settings/journal-setup`
- 2026-10-27: GL-R3 ตามคู่มือ ICC GL-201 บน branch `gl-icc-r12-revision`
  - ค้นหา Journal ตามงวด / Source / Category / Batch · คัดลอก Journal (AutoCopy, `POST /journals/:id/copy`) เป็นร่างใหม่
  - Control Total: Complete ไม่ผ่านถ้ายอดเดบิตไม่ตรง
  - ใบ Journal Voucher สำหรับพิมพ์ (`GET /journals/:id/voucher`, หน้า `/print/journal-voucher/:id`) แสดงผู้จัดทำ / ผู้อนุมัติ / ผู้ Post
  - Post ได้เฉพาะ role ใหม่ `gl_reviewer` (ผู้ตรวจสอบฝ่ายบัญชี) และห้าม Post Journal ที่ตัวเองบันทึก (`SELF_POST`) ยกเว้นรายการกลับ
- 2026-10-28: GL-R4 หลายสกุลเงิน (P27, ADR-0008, คู่มือ ICC GL-101/203.3) บน branch `gl-icc-r12-revision`
  - สกุลเงิน `gl.currency` (THB เปิดอยู่; USD EUR JPY SGD MMK ต้องขอเปิด) · คำขอเปิดใช้ผ่าน ApprovalsService doc type `GL_CURRENCY` (`/currency-requests`)
  - ประเภทอัตรา CORPORATE / SPOT / USER และอัตรารายวันเป็นบาท (`/gl/daily-rates`) ใช้ร่วม ICC/ITK: คีย์เอง, นำเข้าผ่านบริการกลาง object `GL_DAILY_RATES` (ทั้งไฟล์หรือไม่เลย) และธนาคารจำลองอ่านไฟล์ `BANK_RATE_SIM_FILE` (D03) · import object ที่ไม่มีคอลัมน์บริษัทได้แล้ว (`companyColumn` ไม่บังคับ)
  - Journal มีสกุลเงิน ประเภทอัตรา วันที่อัตรา และอัตรา · บรรทัดเก็บ entered และ accounted (THB) · Complete ไม่ผ่านถ้าไม่มีอัตรา (`no_rate`) · ต้องสมดุลทั้งยอด entered และบาท ผลต่างปัดเศษลงบัญชีปัดเศษของ Ledger เป็นบรรทัดระบบ (`is_rounding`) · ทศนิยมตามสกุล (JPY 0) · Control Total เทียบยอด entered · รายการกลับใช้อัตราและยอดบาทเดิม
  - บัญชี FX ต่อ Ledger (`/gl/fx-setup`): ปัดเศษ, กำไร/ขาดทุนยังไม่เกิด · ยอด entered ต่อสกุล `gl.balance_entered`
  - Revaluation (`POST /gl/revaluations`, Idempotency-Key): ปรับยอดสินทรัพย์/หนี้สินสกุลต่างประเทศด้วยอัตราวันสิ้นงวด ลงกำไร/ขาดทุนยังไม่เกิด เป็น Journal source/category `REVALUATION` (ไม่ต้องอนุมัติ, ผู้ตรวจสอบ Post) กลับรายการและ Post เองวันแรกของงวดถัดไป · ครั้งเดียวต่องวดต่อประเภทอัตรา
  - ภาษีและรายงานภาษีอ่านยอดบาท (debit/credit) ตามเดิม
  - หน้า `/gl/currencies` และช่องสกุลเงิน/ตารางยอดบาทในหน้า Journal, ใบ Voucher แสดงสกุลและอัตรา
  - เทสต์ e2e 7 กรณีใหม่ รวมทั้งหมด 127 ผ่าน · ยังไม่ได้เปิดหน้าใน browser (ต้องมีผู้ใช้ทดสอบที่เข้าสู่ระบบได้)
