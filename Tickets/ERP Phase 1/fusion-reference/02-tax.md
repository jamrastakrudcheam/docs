---
tags:
  - erp/fusion-reference
  - erp/tax
module: Tax
fusion_release: 26D
---

# Fusion อ้างอิง: Tax (VAT และภาษีหัก ณ ที่จ่าย)

อ่านจาก Oracle Help Center รุ่น 26D และเอกสาร localization ไทยรุ่น 24C–25D (สรุปด้วยคำของเรา) · ใช้กับ ticket [06](../issues/06-chart-of-accounts-tax.md), [38](../issues/38-tax-registration-tax-register.md), [39](../issues/39-vat-reports-pp30.md), [40](../issues/40-wht-certificates-pnd3-pnd53.md), [41](../issues/41-pnd1-payroll-wht.md) · แนวทางของเรา: [gl-process.md §8–9](../gl-process.md)

## ข้อค้นพบสำคัญ
- Fusion รวมข้อมูลภาษีไว้ที่ **Tax Reporting Ledger** แล้วรายงานภาษีทุกฉบับดึงจากที่นั่น แหล่งข้อมูลคือ Payables, Receivables และ tax repository (รายการภาษีที่นำเข้า) [1] → ตรงกับแนว **tax register กลาง** ของเรา (ticket 38)
- Fusion มี localization ไทยสำหรับ **หนังสือรับรองการหักภาษี ณ ที่จ่าย (50 ทวิ)** โดยต้องเปิดฟีเจอร์ Regional Localization = Thailand [5][6] มี **PND Type** (1, 2, 3, 53, 54) [7], **ประเภทเงินได้ตามมาตรา 40** [8] และ **เลขที่หนังสือรับรองต่อ Legal Reporting Unit + PND Type** [9]
- ในรายการรายงาน Asia/Pacific ของ Fusion ไม่พบ **รายงานภาษีซื้อ/ภาษีขายแบบไทย หรือ ภ.พ.30** [10] → ส่วนนี้ต้องออกแบบเอง (ticket 39) โดยอิงโครง Tax Register ของ Fusion

## ตาราง: Fusion ทำอย่างไร → เราทำอย่างไร

| หัวข้อ | Fusion | ระบบเรา | ต่าง / ต้องตัดสินใจ |
| --- | --- | --- | --- |
| แหล่งรวมภาษี | Tax Reporting Ledger: ภาษีซื้อจาก AP (invoice, credit memo, prepayment), ภาษีขายจาก AR (invoice, memo, adjustment, receipt ที่ภาษีตั้งพักไว้ ฯลฯ) + รายการนำเข้า [1] | tax register: ตอนนี้มาจาก Journal ที่ Post, ภายหลังจาก AP/AR ผ่าน SLA | Fusion ไม่ดึงจาก GL journal ตรง → ของเราเป็นทางชั่วคราวจนกว่ามี AP/AR และต้องกันไม่ให้นับซ้ำเมื่อ AP/AR เปิดใช้ (เพิ่มกฎใน ticket 38) |
| ระดับการรายงาน | Ledger / Legal Entity / **Tax Registration Number** (รวมหลายหน่วยที่ใช้เลขเดียวกันได้) [2] | Legal Entity + สถานประกอบการ (สาขา 5 หลัก) | ตรงแนวคิด: "Tax registration" ของ Fusion ≈ สถานประกอบการของเรา |
| Tax point (วันที่รับรู้ภาษี) | ตั้งต่อภาษีได้: Invoice (ค่าเริ่มต้น), Payment (พักใน **interim tax account** แล้วรับรู้ตามสัดส่วนเมื่อจ่าย), Accounting, Delivery [3] | เดือนภาษี/เดือนที่ขอเครดิตระบุต่อรายการ | **ภาษีขายประเภทบริการของไทยรับรู้เมื่อรับชำระ** → ต้องรองรับ tax point = Payment และบัญชีภาษีขายรอเรียกเก็บ (interim) ใน ticket 25/26 และ 38 → **เพิ่มประเด็นใน ticket 01** |
| Recoverable / Nonrecoverable | ภาษีซื้อแบ่งส่วนขอคืนได้ตาม recovery rate ส่วนที่ขอคืนไม่ได้เป็นต้นทุน [4] | ยังไม่มี | ภาษีซื้อต้องห้าม (ขอคืนไม่ได้) ของไทยต้องลงเป็นค่าใช้จ่าย และไม่อยู่ในรายงานภาษีซื้อ → เพิ่ม recovery rate ที่ tax code (ticket 06/38) |
| รายงานมาตรฐาน | Tax Register, Financial Tax Register, Tax Reconciliation, Audit Trail, Interim Tax Register, Tax Box Return Preparation ฯลฯ [11] | รายงานภาษีซื้อ/ขาย, ภ.พ.30 | เพิ่ม **รายงานกระทบยอดภาษีกับบัญชี** (Tax reconciliation) ใน ticket 39 ช่วยผู้สอบบัญชี |
| Tax Register | แยก register: ทั้งหมด / ภาษีที่ต้องชำระและภาษีซื้อขอคืนได้ / ภาษีขอคืนไม่ได้ · พารามิเตอร์: ระดับการรายงาน, ช่วง tax point date, สกุลเงิน, สถานะลงบัญชี [2] | รายงานภาษีซื้อ (เฉพาะส่วนขอคืนได้) / ภาษีขาย | ตรงแนวคิด |
| WHT: การตั้งค่า | withholding tax rate บน tax, calculation point (invoice/payment), rate schedule, เปิดให้คีย์/แก้บรรทัด WHT มือได้เป็นตัวเลือก, ยกเลิกบรรทัดไม่ได้หลังจ่ายแล้ว, สร้าง "tax authority invoice" (ใบแจ้งหนี้ถึงกรมสรรพากร) ได้ [12] | WHT code: ประเภทเงินได้, อัตรา, แบบ | เพิ่ม **calculation point = payment** (ไทยหักตอนจ่าย) ใน ticket 22 · แนวคิด tax authority invoice ใช้ตั้งหนี้ค้างนำส่งกรมสรรพากรได้ |
| WHT ไทย: PND Type | PND 1 (เงินเดือน), 2 (เงินปันผล ฯลฯ), 3 (บุคคล), 53 (นิติบุคคล), 54 (นิติบุคคลต่างประเทศ) ตั้งบน withholding tax rate [7] | ภ.ง.ด.1, 3, 53 | ภ.ง.ด.2 และ 54 อยู่นอกขอบเขตรอบแรก (เขียนไว้ใน spec แล้วสำหรับ 2) |
| WHT ไทย: ประเภทเงินได้ | ตั้งเป็น product fiscal classification ตามมาตรา 40(1)–(8) และอื่น ๆ [8] | wht_income_type ใน template และ WHT code | ตรงกัน |
| WHT ไทย: เลขที่ 50 ทวิ | กฎเลขที่ต่อ Legal Reporting Unit + PND Type มี prefix และเลขเริ่ม [9] | ticket 05 (เลขที่เอกสาร) | ใช้ scope = สถานประกอบการ + แบบ (ภ.ง.ด.3/53) ตาม Fusion |

## สิ่งที่ควรเพิ่มใน ticket จากการอ่านครั้งนี้
- ticket 01: ภาษีขายประเภทบริการรับรู้เมื่อรับชำระ (tax point = payment) ต้องรองรับรอบแรกหรือไม่
- ticket 06/38: recovery rate (ภาษีซื้อต้องห้าม), tax point basis ต่อ tax code
- ticket 38: กันนับซ้ำเมื่อมีทั้ง Journal และ AP/AR, เลข 50 ทวิ ต่อสถานประกอบการ + แบบ
- ticket 39: รายงานกระทบยอดภาษีกับบัญชี
- ticket 22: หัก WHT ตอนจ่ายเงิน

## แหล่งอ้างอิง
1. [Tax Reporting Ledger](https://docs.oracle.com/en/cloud/saas/financials/26d/fautx/tax-reporting-ledger.html)
2. [Tax Register and Financial Tax Register](https://docs.oracle.com/en/cloud/saas/financials/26d/fautx/tax-register-and-financial-tax-register.html)
3. [Tax Point Basis](https://docs.oracle.com/en/cloud/saas/financials/26d/fautx/tax-point-basis.html)
4. [Tax Recovery Distributions](https://docs.oracle.com/en/cloud/saas/financials/26d/fautx/tax-recovery-distributions.html)
5. [Opt-In for Withholding Tax Certificate for Thailand (25D)](https://docs.oracle.com/en/cloud/saas/financials/25d/faiap/opt-in-for-withholding-tax-certificate-for-thailand.html)
6. [Oracle Fusion Cloud Financials for Asia/Pacific Predefined Reports (26C)](https://docs.oracle.com/en/cloud/saas/financials/26c/ocuar/oracle-fusion-financials-for-asia-pacific-predefined-reports.html)
7. [Configure PND Type (25B)](https://docs.oracle.com/en/cloud/saas/financials/25b/faiap/configure-pnd-type.html)
8. [Configure Revenue Type (25B)](https://docs.oracle.com/en/cloud/saas/financials/25b/faiap/configure-revenue-type.html)
9. [Generate Withholding Tax Certificate for Thailand Sequence Number Rule (24C)](https://docs.oracle.com/en/cloud/saas/financials/24c/fauap/generate-withholding-tax-certificate-for-thailand-sequence.html)
10. รายการรายงาน Asia/Pacific ตาม [6] มีรายงานไทยรายการเดียวคือ Withholding Tax Certificate for Thailand
11. [Oracle Fusion Tax Predefined Reports](https://docs.oracle.com/en/cloud/saas/financials/26d/fautx/oracle-fusion-tax-predefined-reports.html)
12. [Manage Withholding Tax Settings](https://docs.oracle.com/en/cloud/saas/financials/26d/fautx/manage-withholding-tax-settings.html)
