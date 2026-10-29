# GL: ช่องว่างระหว่างคู่มือ ICC Oracle R12 กับระบบปัจจุบัน

แหล่งที่มา: คู่มือผู้ใช้ GL ของ ICC (A-HOST, Oracle EBS R12) 43 ฉบับ ได้แก่ GL-101.1 ถึง GL-206.5 และเอกสารภาพรวม COA ที่เก็บไว้บนเครื่องผู้ใช้ เอกสารนี้สรุปด้วยคำของเราเอง ไม่ได้คัดลอกเนื้อหาคู่มือ

เทียบกับ: ticket 04 (ปฏิทิน/งวด), 05 (เลขที่เอกสาร), 06 (ผังบัญชี/ภาษี) และ 12 (Journal/Post/TB) ที่ commit แล้วบน branch `12-gl-journal-entry-posting`

เครื่องหมาย
- ✅ มีแล้ว
- 🟡 มีบางส่วน
- ❌ ยังไม่มี
- ⚠️ ขัดกับข้อตัดสินใจที่ยืนยันแล้ว ต้องให้ผู้ใช้ตัดสิน

## 1. สิ่งที่ขัดกับข้อตัดสินใจ

ผลตัดสิน (27 ต.ค. 2026): C1 → P26 ผังร่วม (ADR-0007) · C2 → P27 หลายสกุลเงิน (ADR-0008) · C3 → คงนอกขอบเขต · C4 → P28 ครบ 3 โหมด · C5 → P29 เลข Voucher แบบ ICC · C6 → ไม่เปิด Suspense, STAT ทำพร้อม Recurring/Allocation (ticket 32)

| # | คู่มือ ICC | ข้อตัดสินใจปัจจุบัน | ผลกระทบ |
|---|---|---|---|
| C1 | ICC/ITK ใช้ผังบัญชีชุดเดียว (8 segment, 57 หลัก) แยกกันด้วย segment Company และรหัสบัญชีรันร่วมกัน | ADR-0005 กำหนดผังบัญชี 1 ชุดต่อบริษัท (`chart_of_accounts.company_id unique`) | ถ้าใช้ผังร่วม: ย้าย COA ไปผูกกับ "กลุ่ม" แทนบริษัท, segment value ใช้ร่วม, combination ยังแยกตาม company segment |
| C2 | มีสกุลเงินต่างประเทศ (USD EUR JPY SGD MMK), อัตราแลกเปลี่ยนรายวัน (Excel / ดึงจากธนาคาร), Revaluation พร้อมกำไรขาดทุนยังไม่เกิดและกลับรายการต้นงวด | D06 ใช้เงินบาทเท่านั้น | ถ้ารองรับ: journal line ต้องมี entered/accounted amount, ตารางอัตรา และ revaluation |
| C3 | มี Consolidation Book (CSO): mapping บัญชีบริษัทย่อยไปบริษัทแม่, Elimination set (AR/AP, รายได้/ค่าใช้จ่ายระหว่าง ICC↔ITK) และปิดงวดของ book รวม | ADR-0002 และ D14 ไม่ทำ Consolidation/Elimination | คงเดิมได้ (ตัดออกจากขอบเขต) หรือเปิดเป็น ticket แยกในเฟสหลัง |
| C4 | Budget มี 3 ชุดต่อปี (Budget −5%, Current, BG Available), Funds check แบบ Absolute (tolerance 5% ไม่เกิน 5,000 บาท), Advisory และ None (เงินเดือน) | D08 รุ่นแรกใช้ Absolute อย่างเดียว | ticket 13 ต้องรองรับหลายโหมด หรือคง Absolute แล้วเก็บส่วนอื่นไว้ภายหลัง |
| C5 | เลข Voucher 9 หลัก `[Source][Category][YY][MM][Run 3]` เช่น 111801001 นับใหม่ทุกเดือน แยกตาม category; Interface ใช้ `[2][YY][MM][Run 4]` | ticket 05 นับต่อ "กฎ + ปีบัญชี" | ต้องเพิ่มรอบการนับรายเดือน และผูกกฎกับ category (ไม่ขัด decision โดยตรง แต่เปลี่ยนแบบ ticket 05) |
| C6 | กลุ่มบัญชี 9 = Suspense และกลุ่ม 8 = STAT (บัญชีสถิติ) | แบบเดิมไม่มี suspense; journal ต้อง balance | Suspense: ไม่เปิด (แนะนำ) หรือเปิดเฉพาะ journal import; STAT: รองรับ journal สถิติแยกจากเงิน |

## 2. การตั้งค่า (101–106)

| หัวข้อ | คู่มือ | ปัจจุบัน | สิ่งที่ต้องแก้ |
|---|---|---|---|
| Segment value | มี Value, ชื่อไทย/อังกฤษ, Enabled, วันที่มีผล from/to, Parent flag, Rollup group, Level, Allow Budgeting, Allow Posting, Account type (5 แบบ), Third-party control, Reconcile, DFF (Monthly/Comparative income, Management charge, Price per unit, Charge product) | 🟡 มี value, description, account_type, enabled, allow_posting | เพิ่ม `name_en`, `effective_from/to`, `allow_budgeting`, `third_party_control`, `reconcile`, `attributes jsonb` (สำหรับ DFF) และให้ assertUsable ตรวจวันที่มีผล |
| Hierarchy | Parent→child ranges (child values / parent values), รายงาน Rollup detail listing | ❌ | เพิ่ม `gl.segment_hierarchy` (parent, from, to, range_type) ใช้ใน FSG/TB สรุป, Allocation และ Budget organization |
| ขอเปิด/แก้บัญชี | ใช้แบบฟอร์มขอเปิด/แก้บัญชี → ผจก.บัญชีทั่วไปตรวจ → ผู้มีอำนาจอนุมัติ → ตั้งค่า → ผู้ตรวจสอบตรวจ → แจ้ง IT/ระบบภายนอก | 🟡 System Admin ตั้งค่าได้ทันที | ผ่าน ApprovalsService (doc type `GL_ACCOUNT_REQUEST`) ก่อนสร้าง/แก้ค่า, และรายงาน "บัญชีใหม่/แก้ไขตามช่วงวันที่" ให้ผู้ตรวจสอบ |
| Combination | 8 segment, Reserve ค่าเริ่มต้น 000, วันที่มีผล | 🟡 8 segment + ค่าเริ่มต้นต่อ segment ✅ (28 ต.ค.), วันที่มีผลยังไม่มี | segment ที่เว้นว่างใช้ `default_value` · ค่า segment มีเว้นวรรคได้ (Product) · ยังต้องเพิ่ม `effective_from/to` ของ combination |
| Journal Source | เป็น master: Manual, Spreadsheet, Recurring, MassAllocation, Payables, Receivables, Assets, Purchasing, Cost Management มี flag Require approval, Freeze journals, Import reference; ขอเพิ่มผ่านแบบฟอร์มอนุมัติ | ❌ ใช้ check constraint 3 ค่า | ตาราง `gl.journal_source` (code, ชื่อ, `require_approval`, `freeze`, `import_reference`, `voucher_digit`) ระบบสร้างค่าเริ่มต้น, เพิ่ม/ปิดได้ |
| Journal Category | เป็น master ที่ขอเพิ่มได้ ICC มี Adjustment, Budget, Accrual, Recurring, Allocation, Indirect, Interface (AR promotion, Accrued, VAT, Payroll), FA (Addition, Reclass, Transfer, Retirement, Depreciation), INV (Receiving, Inventory, WIP) | 🟡 seed 5 ค่า แก้ไม่ได้ | หน้า Settings สำหรับเพิ่ม/ปิด category, เพิ่ม `voucher_digit` และเกณฑ์กลับรายการ (ดู AutoReverse) |
| Calendar | prefix, ปี, ไตรมาส, งวด 1–14, งวดปรับปรุง | 🟡 ไม่มีไตรมาส | เพิ่ม `quarter` ใน fiscal_period สำหรับยอด QTD ในรายงาน |
| AutoPost | criteria set ตาม priority (ledger, source, category, balance type, period) ตั้งเวลาได้ | ❌ | ทำหลัง SLA (37) โดย journal จาก subledger ถูกสร้างเป็น posted อยู่แล้ว ส่วน AutoPost เฉพาะ source ที่ไม่ต้องอนุมัติ (Recurring/Allocation) อยู่ใน ticket 32 |
| AutoReverse | ต่อ category: งวดกลับรายการ (No default / Next non-adjusting / Next period / Same period / Next day), วิธี Switch Dr/Cr หรือ Change sign, AutoReverse, AutoPost reversal | 🟡 มี `auto_reverse` boolean กลับในงวดถัดไปแบบ Switch Dr/Cr | เพิ่ม `reversal_period_rule`, `reversal_method`, `auto_post_reversal` ใน category; Change sign ต้องให้บรรทัดติดลบได้เฉพาะ reversal |
| Currency | ขอเปิดสกุลเงินผ่านแบบฟอร์มอนุมัติ | ✅ GL-R4 | คำขอเปิดสกุลเงินผ่าน ApprovalsService (`GL_CURRENCY`) |

## 3. การบันทึก Journal (201.x)

| หัวข้อ | คู่มือ | ปัจจุบัน | สิ่งที่ต้องแก้ |
|---|---|---|---|
| ค้นหา | batch, journal, source, category, period, currency, สถานะ post, สถานะ funds (Failed/In process/Passed/Required), control total | 🟡 มี status, ช่วงวันที่, คำค้น | เพิ่มตัวกรอง period, source, category, batch และ (หลัง ticket 13) funds status |
| Batch | batch ชื่อ "YYMM คำอธิบาย", 1 batch มีหลาย journal, control total | 🟡 มี batch_name | คง 1 journal = 1 เอกสาร แต่เพิ่ม `control_total` (ตรวจ total = control ตอน complete) และค้นหา/พิมพ์ตาม batch_name |
| Copy | AutoCopy จาก journal เดิมแล้วแก้ | ❌ | `POST /journals/:id/copy` สร้าง incomplete ใหม่ (วันที่/งวดใหม่) |
| ใบ Journal–Voucher | พิมพ์ก่อนขออนุมัติ และพิมพ์อีกครั้งหลัง post เพื่อตรวจและเก็บแฟ้ม; เขียนเลข voucher บนเอกสารต้นทาง | ❌ | หน้า/PDF Journal Voucher (เลขที่, ผู้เตรียม/อนุมัติ/post, บรรทัด, ลายมือชื่อ) |
| อนุมัติ | วงเงินตามพนักงาน/หัวหน้า, Approve/Reject ผ่าน notification; Reject แล้วกลับไปแก้ | ✅ P17 + ticket 03 | ตรวจว่ามี route ตามวงเงินต่อ ledger แล้ว (ไม่ต้องแก้) |
| ผู้ Post | ผู้ตรวจสอบฝ่ายบัญชีเป็นผู้ Post หลังอนุมัติ แล้วตรวจผล post | 🟡 ผู้มี role gl_accountant คนใดก็ได้ | เพิ่ม role `gl_poster` (ผู้ตรวจสอบ) และห้ามผู้เตรียม post เอง (SoD) |
| เลข Voucher | 9 หลักตาม source+category+เดือน (C5) ออกเมื่อ post หรือเมื่อบันทึก | 🟡 เลขต่อปีจาก NumberingService | เพิ่ม token `{YY}` และรอบนับ `monthly` ใน numbering rule, ให้ doc type ตาม category `GL_JOURNAL:<CAT>` |
| Spreadsheet | template layout ที่ผู้ใช้ตั้งค่าได้ (header/lines) แล้วอัปโหลด | 🟡 ticket 36 มีบริการกลาง | ticket 42 ใช้ import object `GL_JOURNAL` ตาม template ICC |
| Interface | หน้าแก้ข้อมูล import (ledger, source, category, actual flag, group id) แล้ว import ใหม่ | 🟡 บริการกลางมีไฟล์ error | ticket 42: ให้แก้แถวที่ error ในหน้า import แล้ว rerun |
| Indirect (Management charge) | journal เรียกเก็บค่าใช้จ่ายทางอ้อมรายเดือนภายในวันที่ 10 จากยอด MTD | ❌ | ใช้ Allocation (ticket 32) และ flag `Management Charge` ใน segment value |
| Balance type | Actual / Budget / Encumbrance | 🟡 Actual อย่างเดียว | Budget/Encumbrance มากับ ticket 13 (`balance_type` ใน journal และ balance) |
| STAT | journal สถิติในบัญชีกลุ่ม 8 | ❌ | ขึ้นกับ C6 |

## 4. ปิดบัญชี (202–204)

| หัวข้อ | คู่มือ | ปัจจุบัน | สิ่งที่ต้องแก้ |
|---|---|---|---|
| ปิดรายวัน | subledger ส่ง journal เป็น posted รายวัน (FA รายเดือนแบบ summary) PR/PO accrual ส่งทุกวันแล้ว auto reverse; GL ตรวจ unposted แล้ว post (subledger ไม่ต้องอนุมัติ) | ❌ | ticket 37 (SLA) + รายงาน "Journal ยังไม่ post" ต่องวด |
| Account Analysis | รายงานรายการต่อบัญชีพร้อม source/reference เพื่อกระทบยอด subledger กับ GL | 🟡 มี Account inquiry | เพิ่มคอลัมน์ source/category/เลขอ้างอิงต้นทาง, ยอดยกมา/ยกไป และ export Excel |
| ปิดงวด | ตรวจ BS/P&L → ผจก.ตรวจ (IC) → ปิด → เตือนเมื่อ AR/AP/FA/INV/CM ยังไม่ปิด → เปิดงวดถัดไป | 🟡 ปิด/เปิดได้ แต่ไม่ตรวจ module อื่น | ปิด GL ไม่ได้ถ้า module อื่นในงวดเดียวกันยังเปิด (หรือเตือน + ยืนยัน), มีปุ่ม "เปิดงวดถัดไป" และ checklist ปิดงวด (ticket 33) |
| งวดปรับปรุง 13/14 | การเปิดงวดปรับปรุงต้องมีแบบฟอร์มและผู้มีอำนาจอนุมัติทุกครั้ง | 🟡 ต้องใส่เหตุผลเมื่อ reopen | เปิดงวด adjusting (และ reopen) ผ่าน ApprovalsService doc type `GL_PERIOD_OPEN` |
| เลขเอกสารปีถัดไป | ตั้ง sequence ปีใหม่ต่อ category | ✅ counter แยกตามปี/เดือนอัตโนมัติ (เมื่อทำ C5) | ไม่ต้องตั้งรายปี |
| Revaluation | ปรับค่าเงินตราต่างประเทศ | ✅ GL-R4 | ยอดสินทรัพย์/หนี้สินสกุลต่างประเทศ ณ อัตราสิ้นงวด ลงกำไร/ขาดทุนยังไม่เกิด กลับรายการวันแรกงวดถัดไป |
| Carry forward PR/PO | ปลายปียกยอด encumbrance (rule 2: encumbrance + encumbered budget) | ❌ | ticket 13 / 33 |

## 5. งานที่ย้ายไป ticket อื่น (อัปเดตเนื้อหา ticket ตามคู่มือ)

- **32 Recurring/Allocation/Reversal:**
  - Recurring 3 แบบ: Skeleton, Standard และ Formula (ยอด Actual/Budget/STAT แบบ PTD/YTD ของงวดปัจจุบัน/ก่อน/ปีก่อน) โดยบรรทัด 9999 เป็น offset
  - MassAllocation สูตร A×B÷C ไปยังบัญชีปลายทาง T และ offset O แบบ Looping/Summing/Constant ตาม parent
  - Generate แบบ Full หรือ Incremental และตั้งเวลารันได้
  - สูตรต้องได้รับอนุมัติก่อนตั้งค่า และมีรายงาน Formula listing
- **13 Budget:**
  - Budget organization 5 กลุ่ม
  - Budget หลายชุดพร้อมสถานะ Open/Current/Freeze
  - อัปโหลดงบเป็น budget journal
  - โอนงบ (% หรือจำนวน) ภายในเดือนเดียวกันเท่านั้น
  - ขอบเขตการคุม: PTD รายเดือน ยกเว้นบาง cost center ที่คุมรายปี
  - รายงาน Budget vs Actual
  - ทั้งหมดขึ้นกับ C4
- **33 ปิดงวด/ปิดปี:** checklist, ตรวจ module ก่อนปิด GL, งวด 13/14 และยกยอด encumbrance
- **34 งบการเงิน:** FSG = Row set + Column set (Current, YTD, 12 เดือน, YTD เทียบปีก่อน), Content set ตาม company และ export Excel; รูปแบบต้องได้รับอนุมัติก่อนใช้
- **42 Journal import:** template ICC, แก้แถว error แล้ว import ใหม่ และ group id
- **43 ย้ายข้อมูล EBS:** segment values พร้อม attribute ใหม่ในข้อ 2 และ hierarchy

## 6. ลำดับการแก้

1. ✅ **GL-R1 ผังบัญชีร่วม (P26):** (27 ต.ค. 2026)
   - COA ผูกกับกลุ่มบริษัท และกำหนดค่า Company segment ต่อบริษัท
   - attribute ของ segment value และวันที่มีผล
   - hierarchy
   - คำขอเปิด/แก้บัญชีผ่านการอนุมัติ
   - แก้ ticket 06 และหน้า /settings/accounts
2. ✅ **GL-R2 Source/Category และเลข Voucher (P29):** (27 ต.ค. 2026; AutoReverse ตามเกณฑ์ของ category ทำในรอบนี้ด้วย)
   - ตาราง journal_source
   - category เพิ่ม/ปิดได้ พร้อม voucher digit และเกณฑ์ AutoReverse
   - numbering รอบเดือนพร้อม token {YY}
3. ✅ **GL-R3 Journal:** (27 ต.ค. 2026)
   - ตัวกรอง
   - Copy
   - Control total
   - ใบ Journal Voucher
   - role ผู้ Post พร้อม SoD
   - AutoReverse ตามเกณฑ์
4. ✅ **GL-R4 หลายสกุลเงิน (P27):** (28 ต.ค. 2026; ธนาคารเป็นไฟล์จำลอง `BANK_RATE_SIM_FILE` ตาม D03)
   - สกุลเงิน, ประเภทอัตรา และอัตรารายวัน (นำเข้าผ่าน ImportRegistry)
   - journal line เก็บยอด entered/accounted
   - Revaluation
5. **GL-R5 ปิดงวด:**
   - ตรวจ module อื่นก่อนปิด GL
   - ปุ่ม "เปิดงวดถัดไป"
   - เปิดงวดปรับปรุงผ่านการอนุมัติ
   - ไตรมาสของงวด
6. **GL-R6 Account Analysis:** แสดง source/reference และ export Excel
7. **อัปเดตเนื้อหา ticket อื่น:** 13 (P28), 32, 33, 34, 42 และ 43 ตามข้อ 5
