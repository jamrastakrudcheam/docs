---
spec: erp-phase1
tags:
  - erp/spec
---

# Spec: ICC ERP รุ่นแรก — โมดูลที่เหลือหลัง Foundation

**Status:** ready-for-agent

เอกสารต้นทาง: `01-scope-decisions.md` … `06-validation-delivery.md` (ยืนยันแล้ว D01–D15, P01–P12)
มาตรฐานโค้ดและความปลอดภัย: [docs/CODING_STANDARDS.md](../../docs/CODING_STANDARDS.md)
รายการ ticket: [index.md](index.md) · กระบวนการ GL และภาษี: [gl-process.md](gl-process.md) · บริการนำเข้าไฟล์กลาง: [import-framework.md](import-framework.md)

## Problem Statement

ตอนนี้ ICC ERP มีแค่โครงพื้นฐาน: login, ตัวเลือกบริษัท/OU, S01 โครงสร้างองค์กร, S02 ผู้ใช้และสิทธิ์ และ reset password
เมนูอีก 13 โมดูลบนหน้าเว็บยังเป็นป้าย "ยังไม่พัฒนา" ทั้งหมด ฝ่ายบัญชีและฝ่ายปฏิบัติการของ ICC และ ITK จึงยังบันทึกงานจริงในระบบไม่ได้ ได้แก่ งบประมาณ, จัดซื้อ, รับของ, เจ้าหนี้, จ่ายเงิน, สต็อกและต้นทุน, ขาย, จัดส่ง, ออกบิล, ลูกหนี้, ติดตามหนี้, สินทรัพย์ถาวร, กระทบยอดธนาคาร, บัญชีแยกประเภท, ปิดงวด และงบการเงิน
ผู้ตรวจงานก็ยังไม่มีรายการงานที่แบ่งเป็นชิ้นชัดเจนให้ไล่ตรวจว่าแต่ละระบบต้องทำอะไรและทำครบหรือยัง

## Solution

สร้างโมดูลที่เหลือตาม Screen inventory (02) และ Workflow (04) ทีละชิ้นแบบ vertical slice แต่ละชิ้นต้องใช้งานได้จริงตั้งแต่ฐานข้อมูล API หน้าจอ ไปจนถึงเทสต์ และผูกกับบัญชีผ่าน SLA/GL ใน transaction เดียว (ADR-0001)
ลำดับงานตาม Delivery plan ใน 06: Foundation ที่เหลือ → Inventory Core → P2P → O2C → FA → CM → Close & Statements
ทุก ticket มีกฎธุรกิจ, acceptance criteria และข้อมาตรฐาน/ความปลอดภัยเฉพาะงานนั้น เพื่อให้ตรวจเช็คใน Obsidian ได้ทีละข้อ

## User Stories

### ตั้งค่าและข้อมูลหลัก
1. As a System Admin, I want to define approval routes by document type, OU and amount limit, so that every document goes to the right approver (D07).
2. As an approver, I want an Approval Inbox listing everything waiting for me across modules I have rights to, so that I don't miss approvals.
3. As an approver on mobile, I want to review and approve or return a document, so that approvals don't wait until I'm at a desk (D06).
4. As a requester, I want the system to stop me approving my own document, so that segregation of duties is enforced (D07).
5. As a requester, I want a material edit after submission to create a new revision and cancel prior approvals, so that approvers always see what they approved (D07).
6. As a GL manager, I want a fiscal calendar per ledger with normal and adjustment periods, so that postings land in the right period.
7. As a GL manager, I want to open and close each module's period separately per company/Org, so that subledgers close before GL (04).
8. As a System Admin, I want document numbering per document type, scope and fiscal year, so that numbers are unique and gap rules are clear.
9. As a GL manager, I want to maintain account combinations, tax codes with effective versions and rounding rules per company, so that postings use the right accounts and tax (P08).
10. As an auditor, I want to search the audit trail by document, user and date, so that I can trace who changed what.
11. As a clerk, I want double-clicks and two browser tabs not to create duplicate postings, so that balances stay correct (idempotency, T08).
12. As a master-data clerk, I want to maintain customers, sites and credit limits with maker/checker, so that sales use approved data (M01).
13. As a master-data clerk, I want to maintain suppliers and sites, with bank details visible only masked, so that payment fraud risk is reduced (M02).
14. As a bank-detail custodian, I want to be the only role that edits supplier bank accounts, separate from the supplier creator, so that bank changes are controlled (03).
15. As an inventory admin, I want items, UOM conversions, inventory orgs per brand, subinventories and locators, so that stock is tracked per Org (D05, M03, M05).
16. As a pricing admin, I want price lists and payment terms with effective dates, so that quotations and POs use current prices (M04).

### บัญชี (SLA / GL)
17. As a GL accountant, I want each source transaction to create an accounting event and balanced Dr/Cr entries from a versioned SLA rule, so that every posting traces back to its source (D13, P12).
18. As a GL accountant, I want a preview of accounting entries before final posting, so that I can fix mapping exceptions first.
19. As a clerk, I want a final command to roll back entirely if an account mapping is missing, so that stock or settlement never changes without accounting (03).
20. As a GL accountant, I want manual journals with submit, approve, post and linked reversal, so that adjustments are controlled.
21. As a GL accountant, I want recurring and allocation journals from template versions with preview, so that monthly entries are consistent (G04).
22. As a GL manager, I want a trial balance per company/ledger with opening, period Dr/Cr and closing, so that I can review balances (CL04).
22a. As a GL accountant, I want a journal entry screen in the Oracle Fusion style (batch, journal, lines, running Dr/Cr difference, keyboard entry), so that I can key adjustments quickly (G03).
22b. As a GL accountant, I want to download an Excel template, fill many journals and upload them, getting row-level errors and no partial import when anything is wrong, so that bulk entries are fast and safe.
22m. As any module user, I want every file import (journals, invoices, receipts, bank statements, asset additions, counts, payroll tax) to work the same way as Oracle FBDI (template → load → import → correct errors → purge) with job status I can follow, so that I learn one process.
22n. As an integrator, I want to send a zip of CSV files through an API and poll the job status, so that other systems can feed ICC ERP without screen work.
22c. As a GL accountant, I want to record VAT and withholding tax details on journal lines, so that tax reports can be produced before AP/AR exist.
22d. As a GL manager, I want accrual journals to reverse automatically in the next period, so that I do not forget reversals.
22e. As an accountant, I want account inquiry that drills from balances to journals and source documents, so that I can explain any balance.

### ภาษี
22f. As a tax officer, I want tax registrations (head office and branches) per legal entity, so that reports are produced per establishment.
22g. As a tax officer, I want monthly purchase and sales VAT reports per establishment, so that I meet record-keeping requirements.
22h. As a tax officer, I want a monthly PP30 (ภ.พ.30) summary computed from those reports including excess tax brought forward, so that I can file VAT.
22i. As a tax officer, I want withholding tax certificates (50 ทวิ) per payee per payment, so that payees receive proof of withholding.
22j. As a tax officer, I want PND3 and PND53 monthly forms with attachments split by payee type, so that I can file withholding tax.
22k. As a payroll officer, I want to import monthly payroll tax data and produce PND1, reconciled with GL, so that salary withholding is filed without a payroll module.
22l. As a tax officer, I want confirmed filings to be frozen as numbered snapshots with additional filings for later changes, so that what was filed is preserved.

### งบประมาณและจัดซื้อ (P2P)
23. As a budget owner, I want budget versions and lines by company/OU/year/period/cost centre/account/category, so that spending is controlled (B01–B02).
24. As a budget owner, I want to see original, changes, encumbered, actual and available per budget line, so that I know what is left (B03).
25. As a budget owner, I want transfer/increase requests that go through approval, so that budget changes are authorised (B04).
26. As a requester, I want an approved PR to reserve budget atomically and be refused if it exceeds the absolute limit, so that overspending is impossible (D08, T04).
27. As a buyer, I want to convert PR lines to a PO without reserving budget twice, so that budget is counted once (D08).
28. As a buyer, I want PO revisions that re-trigger approval when material fields change, so that changes are controlled (P03 screen).
29. As a buyer, I want cancelling or closing a PO balance to release the unused reservation, so that budget returns to the pool (D08).
30. As a receiver, I want to record received quantity, inspection and destination, and on acceptance update stock, cost and accounting together, so that inventory and GL agree (04).
31. As a receiver, I want to return goods to a supplier against a receipt, so that stock, cost and supplier credit are adjusted without deleting the receipt.
32. As an AP accountant, I want to match invoice lines to receipts and PO lines with price/quantity tolerance per company, so that over-billing is held (P06).
33. As an AP accountant, I want posting an AP invoice to convert the budget reservation to actual exactly once, so that budget is not double counted across receipt, AP and GL (P05).
34. As a treasury officer, I want payment proposals and finalised payments with locked unpaid balances, so that two people cannot pay the same invoice (04).
35. As an AP accountant, I want supplier aging per company, so that I can plan payments (A05).

### สต็อกและต้นทุน
36. As a stock controller, I want on-hand, reserved and available quantities per Org/item/locator, so that I know what can be sold (I01).
37. As a stock controller, I want every movement to record signed quantity and value with its source line, so that stock is fully traceable (I02).
38. As a stock controller, I want the system to refuse negative stock, so that physical and system stock stay consistent (D10).
39. As a cost accountant, I want moving average cost per Org+item with high-precision rounding, so that valuation is correct (D10, 03).
40. As a cost accountant, I want AP price changes after receipt to split between inventory, COGS and variance per the costing policy version, so that history isn't silently rewritten (P07).
41. As a stock controller, I want transfers between Orgs with in-transit stock that is not available at the destination until received, so that stock isn't double counted (I04).
42. As a stock controller, I want physical and cycle counts with scope freeze, blind count, recount thresholds and variance approval by a different person, so that adjustments are trustworthy (P01, T03).
43. As a cost accountant, I want to close the inventory period per Org, so that GL close can proceed (I09).

### ขายและส่งของ (O2C)
44. As a salesperson, I want quotations and sales orders with price/GP checks, so that margins are protected (O01–O02).
45. As a credit approver, I want orders checked against outstanding AR plus committed orders and held when over limit or overdue, so that credit risk is controlled (D09, T05).
46. As a salesperson, I want to request a hold release but not approve it myself, so that credit decisions are separate (03).
47. As a warehouse clerk, I want allocation to reserve available stock without reducing on-hand, so that picking is planned (O04).
48. As a warehouse clerk, I want pick, pack, ship confirm and POD per line with partial shipments, so that deliveries are tracked (F01–F05).
49. As a shipping clerk, I want ship confirm to reduce stock and reservation and post COGS in one step that can't be confirmed twice, so that stock and GL agree.
50. As a shipping clerk, I want to record carrier, tracking and POD manually, so that delivery status is known without a carrier API (P04).
51. As a billing clerk, I want billing eligibility after ship confirm or after POD depending on customer site, so that invoices follow agreed terms (P03).
52. As a billing clerk, I want to combine several shipments into one invoice when company/OU, bill-to, currency, terms and billing conditions match, so that customers get fewer invoices (P02).
53. As an AR accountant, I want invoice lines linked to shipment allocations and locked while billing, so that nothing is billed twice.
54. As an AR accountant, I want receipts applied to open items only up to the allocated amount, so that unapplied money doesn't reduce invoices (04).
55. As an AR accountant, I want credit/debit memos linked to invoices with reason and approval, and physical returns as a separate linked operation, so that stock isn't increased by a financial-only memo.
56. As a collector, I want a worklist of overdue customers, cases, contacts, promises and disputes, so that I can follow up (C01–C04).
57. As a collector, I want promises to pay not to change AR balances, so that only real receipts reduce debt (01, T06).

### สินทรัพย์ถาวร (FA)
58. As an asset accountant, I want capital AP distributions queued for capitalisation and allocatable to several assets without exceeding cost, so that no cost is registered twice (FAI01, T07).
59. As an asset accountant, I want CIP projects whose cost transfers to multiple assets with evidence, so that capitalisation is traceable (FAI03).
60. As an asset accountant, I want Corporate and Tax books with separate methods, where only Corporate posts to GL, so that tax depreciation doesn't duplicate postings (D11).
61. As an asset accountant, I want daily straight-line depreciation posted per period with configurable day-count, leap-year and mid-period rules, so that depreciation is accurate (P09).
62. As an asset manager, I want transfers, physical counts and retirements with gain/loss, so that the register stays correct (FAI05–FAI07).

### ธนาคารและปิดงวด
63. As a treasury officer, I want to import sample bank statements with a file fingerprint and reconcile lines to payments and receipts, so that clearing accounts are cleared (CM01–CM03).
64. As a GL manager, I want a close cockpit showing blockers per module (drafts, holds, unaccounted events, control account mismatches), so that I know what stops the close (CL01).
65. As a GL manager, I want closing to lock module periods against concurrent final postings, so that nothing posts into a closed period (T10).
66. As an auditor-facing accountant, I want adjustment periods and auditor adjustments, so that year-end entries are separate (CL02).
67. As a GL manager, I want year close that creates one retained-earnings/opening batch per company/year, with controlled reopen, so that years roll forward safely (CL03, P12).
68. As a CFO, I want a financial pack per company with TB, balance sheet, P&L, indirect cash flow and equity changes, comparatives, mapping version and sign-off, so that statements are reproducible (D14, P10, FS01–FS03).
69. As a CFO, I want revisions of a finalised pack to keep the old final version, so that history is preserved.

### ภาพรวมและการตรวจสอบ
70. As any user, I want the Overview KPIs and worklist to show real figures for my selected company/OU, so that the home page is useful (H01).
71. As an auditor, I want to trace any document to its source and accounting entries, so that I can verify the chain (H03).
72. As a reviewer, I want each module delivered as a ticket with acceptance criteria and a standards/security checklist, so that I can verify it in Obsidian.

## Implementation Decisions

- **ลำดับและการพึ่งพา** ตาม Delivery plan ใน 06 และกราฟใน [index.md](index.md) ticket ที่ไม่มีตัวบล็อกเริ่มได้ทันที
- **โมดูลและ schema:** ใช้ schema ที่มีอยู่แล้วตาม 03 (`budget`, `purchasing`, `inventory`, `sales`, `shipping`, `ar`, `collection`, `ap`, `assets`, `cash`, `sla`, `gl`, `audit`) โมดูลละ NestJS module เดียว โมดูลอื่นห้ามเขียนข้าม schema ตรง
- **Transaction:** คำสั่ง final ทุกตัวรับ `trx` ตัวเดียวแล้วส่งต่อให้ทุก service ที่เกี่ยวข้อง รวม SLA ถ้า mapping ไม่พร้อมให้ rollback ทั้งคำสั่ง ผู้ใช้บันทึก draft ไว้ได้ (03 Accounting design)
- **SLA ใช้ร่วมกัน:** ทำ accounting service กลางหนึ่งตัว รับ accounting event (source type/id/revision/event type) แล้วคืน entries ที่ balance ตาม rule version ที่มีผล ณ วันที่ เพื่อให้ทุกโมดูลเรียกผ่าน interface เดียว
- **คู่บัญชีหลัก** (P12, รหัสจริงมาจาก mapping): receipt Dr Inventory/Cr GRNI, AP Dr GRNI(+VAT/variance)/Cr AP, shipment Dr COGS/Cr Inventory, AR Dr AR/Cr Revenue(+Tax), payment/receipt ผ่าน clearing จน reconcile, depreciation Dr Expense/Cr Accumulated depreciation
- **GL แบบ Oracle Fusion** (gl-process.md): Journal Batch → Journal → Line, Source/Category, สถานะ Incomplete → Unposted → Pending approval → Ready to post → Posted, ยอดเก็บในตาราง balance ต่อ ledger + account + period, แก้ด้วย Reverse เท่านั้น
- **บริการนำเข้าไฟล์กลางแบบ Oracle FBDI** ([import-framework.md](import-framework.md), ticket 36, ADR-0004): import object registry, template .xlsx ต่อ object (แท็บชื่อ interface), รับ .xlsx หรือ .zip ของ CSV, Load Interface File → Import → Correct Import Errors / Purge เป็นงานเบื้องหลังใน PostgreSQL · ทุกโมดูลห้ามทำหน้าอัปโหลดของตัวเอง ให้ลงทะเบียน import object แทน
- **อัปโหลด Journal:** template Excel ([journal-upload-template.xlsx](../../docs/templates/journal-upload-template.xlsx)) แถว 1 เป็นรหัสคอลัมน์ที่ระบบอ่าน, staging ต่อแถว, ทั้งไฟล์ผ่านหรือไม่ผ่านพร้อมกัน
- **Tax register กลาง:** รายงานภาษีทุกฉบับอ่านจาก tax register ไม่อ่าน Journal ตรง ตอนนี้เขียนจาก Journal ที่ Post แล้ว ภายหลัง AP/AR เขียนผ่าน SLA ชุดเดียวกัน
- **แบบภาษี:** ยืนยันยื่นแล้วเก็บ snapshot มีเลขฉบับ การแก้ภายหลังเป็นฉบับยื่นเพิ่มเติม อัตรา/วันกำหนดยื่น/ระยะขอเครดิตเป็นค่าตั้งค่า
- **Approval engine ใช้ร่วมกัน:** workflow definition/version ต่อ document type, route ตาม OU และวงเงิน, step instances, maker ≠ checker, revision ทำให้การอนุมัติเดิมเป็นโมฆะ ทุกโมดูลเรียกผ่าน interface เดียว
- **Idempotency:** ตาราง IdempotencyRecord ผูก user/scope/command/key/request hash กับผลลัพธ์ใน transaction เดียว (ไม่ใช่ outbox)
- **Context:** หน้าเว็บและ list API ใช้ company/OU จาก session (`/me/context`) เป็นค่าเริ่มต้นของตัวกรอง และตรวจกับ scope grant ทุกครั้ง
- **สิทธิ์:** เพิ่ม permission ต่อ action ผูกกับ role ตาม authorization matrix ใน 03 และตรวจใน guard/service
- **ตัวเลข:** money `numeric(20,2)`, qty/cost `numeric(24,6)` ส่งเป็น string คำนวณใน PostgreSQL หรือด้วย decimal library ฝั่ง server
- **ระบบภายนอก:** ธนาคาร, POS, E-Tax, carrier จำลองหรือบันทึกมือเท่านั้น (D03, P04)
- **ประเด็นที่ยังต้องตัดสินใจ** อยู่ใน ticket 01 และบล็อก ticket ที่เกี่ยวข้อง

## Testing Decisions

- **Seam เดียวเป็นหลัก:** REST API e2e (Nest app + PostgreSQL จริงในฐาน `icc_erp_test`) แบบเดียวกับเทสต์ที่มีอยู่ (`org`, `iam`, `auth`, `password-reset`) ทดสอบพฤติกรรมที่ผู้ใช้เห็นผ่าน API ไม่ทดสอบโครงสร้างภายใน
- **เทสต์ที่ดี** เขียนเป็นประโยคอธิบายกฎ ใช้ข้อมูลสังเคราะห์ที่สร้างใหม่ทุกเทสต์ (`freshStart`, `companyWithOus`) และตรวจผลทั้งยอดและรายการบัญชี (SLA balanced, GL control reconcile)
- **Acceptance scenarios T01–T11** ใน 06 เป็นเทสต์ปลายทางของ ticket ที่เกี่ยวข้อง แต่ละ ticket ระบุว่าครอบคลุม T ไหน
- **concurrency / double submit** ต้องมีในทุกคำสั่ง final (T08)
- **หน้าเว็บ:** ตรวจในเบราว์เซอร์ทั้ง desktop และมือถือในแต่ละ ticket ยังไม่มี UI test อัตโนมัติ ถ้าจะเพิ่มให้ทำเป็น ticket แยก
- **CI:** GitHub Actions ต้องเขียวทุก ticket (lint, typecheck, e2e, build)

## Out of Scope

ตาม 01 "นอกการส่งมอบรุ่นแรก": Manufacturing, Blanket/Back-to-Back PO, Landed cost/import, เงินตราต่างประเทศ/Revaluation, ช่องทางขาย Counter/Shop/Event/Van/Online แบบเต็ม, Consolidation/Elimination, การเชื่อม bank/POS/E-Tax/WMS/carrier จริง, รายงานที่ Blueprint ยังระบุ Confirm
OIDC/SSO จริง, ลืมรหัสผ่านทางอีเมล และ UI test อัตโนมัติ ไม่อยู่ใน spec นี้
โมดูลเงินเดือนเต็มรูปแบบ (รุ่นแรกนำเข้าข้อมูลเพื่อ ภ.ง.ด.1 เท่านั้น), แฟ้มยื่นออนไลน์ e-Filing/RD Prep (รอตัดสินใจใน ticket 01), ภ.ง.ด.50/51 และ ภ.ง.ด.2 ไม่อยู่ใน spec นี้

## Further Notes

- เอกสารต้นทางเป็น synthesis ของ Blueprint ไม่ใช่ requirement ครบ 300 ข้อ (06) การปิด ticket ครบทุกใบจึงไม่ได้หมายความว่าครอบคลุม requirement ต้นฉบับทั้งหมด
- ค่าที่เป็นนโยบาย (tolerance, วงเงิน, rounding, COA, เลขภาษี) เป็น configuration ต่อบริษัท ห้ามเดาค่าจริง ใช้ค่าทดสอบใน UAT
- ticket เป็นจุดเริ่มสำหรับตรวจและปรับ ถ้าเห็นว่าใหญ่หรือเล็กเกินไป ให้แก้ใน Obsidian ได้เลย
