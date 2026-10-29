---
tags:
  - erp/fusion-reference
  - erp/cash
module: Cash Management
fusion_release: 26D
---

# Fusion อ้างอิง: Cash Management

ใน 26D เนื้อหา Cash Management อยู่ในคู่มือ "Using Payables Invoice to Pay" (สรุปด้วยคำของเรา) · ใช้กับ ticket [31](../issues/31-cash-management-reconcile.md), [22](../issues/22-payments.md), [26](../issues/26-receipts-memos-returns.md)

## ตาราง: Fusion ทำอย่างไร → เราทำอย่างไร

| หัวข้อ | Fusion | ระบบเรา (ticket 31) | ต่าง / ต้องตัดสินใจ |
| --- | --- | --- | --- |
| นำเข้า statement | template Excel → CSV → interface → bank statement · แยก header / lines / balances (รหัสยอด: เปิด/ปิด แบบ booked/available) / line charges · ยอดเป็นบวกพร้อมตัวบอก Dr/Cr · transaction type และ transaction code ต้องตรงกับที่ตั้งไว้ · **นำเข้าแบบทั้งหมดหรือไม่มีเลย** แก้แล้วรันใหม่ [1] | นำเข้าไฟล์ตัวอย่าง + fingerprint | เพิ่ม **ยอดเปิด/ปิดของ statement** แล้วตรวจว่า เปิด + รายการ = ปิด · ใช้ template แนวเดียวกับ journal upload (ทั้งไฟล์ผ่านหรือไม่ผ่าน) |
| Auto-reconciliation | rule set (ลำดับ matching rules + tolerance rules) ผูกกับบัญชีธนาคาร · tolerance ตามวันที่/ยอด/% แบบห้ามหรือเตือน · บรรทัดที่หาคู่ไม่ได้เป็น exception · **Mark reviewed** กันการกลับรายการโดยไม่ตั้งใจ [2] | match/finalize/reverse | เพิ่ม rule set + tolerance และสถานะ reviewed (รอบแรกอาจทำ match มือก่อน แล้ว auto ภายหลัง) |
| รายการที่ไม่มีในระบบ | **External cash transactions**: ค่าธรรมเนียม ดอกเบี้ย ฯลฯ สร้างจากบรรทัด statement ที่ยังไม่ match ตามกฎ หรือเป็นรายการปรับผลต่างตอน reconcile · ผลต่าง 1–1 แยกไปบัญชีค่าธรรมเนียม/กำไรขาดทุนอัตราแลกเปลี่ยน ส่วนแบบ 1–M, M–1, M–M ไปบัญชีผลต่างการกระทบยอดของบัญชีธนาคาร [3] | ReconciliationAdjustment | ใช้แนวนี้: ตั้ง **บัญชีค่าธรรมเนียม** และ **บัญชีผลต่างการกระทบยอด** ต่อบัญชีธนาคาร · สร้างรายการลง SLA (ticket 37) |

## สิ่งที่ควรเพิ่มใน ticket 31
- ยอดเปิด/ปิดของ statement และการตรวจความต่อเนื่อง
- matching rule + tolerance, สถานะ reviewed
- external cash transaction (ค่าธรรมเนียม/ดอกเบี้ย) พร้อมบัญชีต่อบัญชีธนาคาร

## แหล่งอ้างอิง (Using Payables Invoice to Pay, 26D)
1. [How Bank Statements Import Data Is Processed](https://docs.oracle.com/en/cloud/saas/financials/26d/fappp/how-bank-statements-import-data-is-processed.html)
2. [Automatic Reconciliation](https://docs.oracle.com/en/cloud/saas/financials/26d/fappp/automatic-reconciliation.html)
3. [Overview of External Cash Transactions](https://docs.oracle.com/en/cloud/saas/financials/26d/fappp/overview-of-external-cash-transactions.html)
