---
ticket: 36
phase: Foundation
module: Import
screens: ["งานนำเข้า (ใหม่): อัปโหลด, Load, Import, Correct Import Errors, Purge"]
refs: ["D03", "T08", "ADR-0004"]
blocked_by: [07]
tags:
  - erp/ticket
  - erp/foundation
---

# 36: บริการนำเข้าไฟล์กลาง (แบบ Oracle FBDI)

**Status:** resolved
**Type:** task
**Blocked by:** [07 Idempotency และหน้าดู Audit](07-idempotency-audit-viewer.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

บริการกลางที่ทุกโมดูลใช้นำเข้าข้อมูลจากไฟล์ ตามลำดับขั้นของ Oracle FBDI: ดาวน์โหลด template ของ import object → อัปโหลด .xlsx หรือ .zip ของ CSV → **Load Interface File** (ตรวจโครงไฟล์ บรรจุลง interface rows ทั้งไฟล์หรือไม่โหลด) → **Import** (เรียก handler ของโมดูลเจ้าของ) → **Correct Import Errors** / Delete Import Data / **Purge** ทุกขั้นเป็นงานเบื้องหลังที่ติดตามสถานะได้ (รายละเอียด: [import-framework.md](../import-framework.md)) · ticket นี้ไม่มี import object ทางธุรกิจ ใช้ object ทดสอบในเทสต์ object แรกจริงคือ ticket 42

หน้าจอ: งานนำเข้า (ใหม่): อัปโหลด, Load, Import, Correct Import Errors, Purge
อ้างอิง: D03, T08, ADR-0004

## ขอบเขต

- Import object registry: code, โมดูลเจ้าของ, สิทธิ์ต่อขั้น, นิยาม interface และคอลัมน์ (ชนิด/ความยาว/บังคับ/lookup), เวอร์ชัน template, grouping rule, หน่วยของความสำเร็จ (ทั้ง load / ต่อกลุ่ม), handler validate/create, ข้อจำกัด
- สร้างไฟล์ template .xlsx จาก registry อัตโนมัติ (แท็บคำแนะนำ + แท็บชื่อ interface) — template ไม่ต้องทำมือทีละ object
- File store นอก web root แยกโฟลเดอร์ต่อ object, fingerprint กันไฟล์ซ้ำ
- ตาราง interface กลาง (load_request_id, interface, เลขแถว, group_id, ค่า typed JSON, สถานะ NEW/ERROR/PROCESSED, ข้อผิดพลาด, id เอกสารที่สร้าง)
- ตารางงาน (LOAD / IMPORT / PURGE) + worker ในโปรเซส API ตาม ADR-0004: Queued → Running → Succeeded / Warning / Error, จำนวนแถว, log, ไฟล์ผลลัพธ์, heartbeat
- Correct Import Errors: ดาวน์โหลดแถวผิดเป็น template เดิม + คอลัมน์ข้อผิดพลาด แก้แล้วอัปโหลดเป็นการแก้ load เดิม · Delete Import Data · Purge มือ/อัตโนมัติตามอายุ (ค่าเริ่มต้น 30 วัน)
- หน้า "งานนำเข้า" (เทียบ Scheduled Processes) และ REST สำหรับระบบภายนอก (อัปโหลด + สั่งรัน + ดูสถานะ)

## Acceptance criteria

- [x] ไฟล์ผิดโครง (sheet/คอลัมน์/เวอร์ชัน/ชนิด/ความยาว/ค่าบังคับ) ไม่โหลดแถวใดเลย และรายงานแถว/คอลัมน์/เหตุผลเป็นภาษาไทย
- [x] ไฟล์ .xlsx จาก template และ .zip ของ CSV ที่ข้อมูลเดียวกันได้ interface rows เหมือนกัน
- [x] Import ต่อกลุ่ม: กลุ่มที่ถูกสร้างเอกสาร กลุ่มที่ผิดเป็น ERROR · Import ทั้ง load: ผิดแถวเดียวไม่สร้างอะไรเลย
- [x] รัน LOAD/IMPORT ซ้ำบน load เดิมไม่ทำให้แถวหรือเอกสารซ้ำ
- [x] Correct Import Errors แล้ว import ใหม่ได้เฉพาะแถวที่แก้ แถวที่ PROCESSED ไม่ถูกทำซ้ำ
- [x] อัปโหลดไฟล์เดิมซ้ำถูกปฏิเสธ (fingerprint)
- [x] แถวที่บริษัท/OU อยู่นอกสิทธิ์ผู้อัปโหลดเป็น ERROR
- [x] Purge ลบ interface rows ของ load ได้ทั้งแบบมือและตามอายุ และไม่ลบเอกสารที่สร้างแล้ว
- [x] worker ที่ตายระหว่างงาน: งานกลับเป็น Queued หลัง heartbeat หมดเวลา และรันต่อได้ถูกต้อง

## เทสต์ที่ต้องมี

- [x] e2e ด้วย import object ทดสอบ (สร้างเอกสารปลอมในตารางทดสอบ): โครงไฟล์ผิด, xlsx เทียบ zip CSV, ต่อกลุ่ม vs ทั้ง load, รันซ้ำ, correct & reimport, ไฟล์ซ้ำ, scope, purge, worker crash
- [x] เทสต์ความปลอดภัย: zip bomb, ชื่อไฟล์ใน zip แบบ path traversal, .xlsm, ไฟล์ใหญ่เกิน, ค่าเริ่มด้วย = ในไฟล์แถวผิดที่ส่งกลับ

## มาตรฐานโค้ดและความปลอดภัย

- [x] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] ตรวจนามสกุล + magic bytes, ขนาด, จำนวนแถว ก่อน parse · ปฏิเสธ .xlsm/.xls
- [x] zip: จำกัดจำนวนไฟล์และขนาดหลังแตก, ชื่อไฟล์ต้องตรงชื่อ interface, ไม่แตกลงดิสก์ตามชื่อที่ส่งมา
- [x] อ่านเฉพาะค่าในเซลล์ ไม่ประมวลผลสูตร/macro/ลิงก์ภายนอก
- [x] ไฟล์ที่ระบบสร้างกลับใส่ ' นำหน้าค่าที่ขึ้นต้นด้วย = + - @ (กัน formula injection)
- [x] สิทธิ์แยกต่อขั้นต่อ object, audit ทุกขั้นพร้อม fingerprint, ไม่ log ค่าของแถว
- [ ] interface ที่มีข้อมูลอ่อนไหวเห็นเฉพาะบทบาทที่กำหนดและ purge เร็วกว่าปกติ
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments

- 2026-10-27: ทำเสร็จบน branch `36-file-based-data-import`
  - `ImportRegistry` ให้โมดูลลงทะเบียน import object (interface/คอลัมน์/ชนิด, เวอร์ชัน template, grouping, หน่วย import ทั้ง load หรือต่อกลุ่ม, สิทธิ์ต่อขั้น, limit, อายุ purge, handler validate/create)
  - template .xlsx สร้างอัตโนมัติ (แท็บ "คำแนะนำ" + แท็บ interface) · รับ .xlsx และ .zip ของ CSV ได้แถวเหมือนกัน · ไฟล์เก็บนอก web root (`IMPORT_STORE_DIR`) ชื่อระบบสร้าง fingerprint กันซ้ำ
  - งาน LOAD/IMPORT/PURGE ในตาราง `import.job` รันโดย `ImportRunner` ในโปรเซส API (`FOR UPDATE SKIP LOCKED`, heartbeat, requeue เมื่อหมดเวลา 2 นาที, หยุดพัก 30 วินาทีเมื่อฐานข้อมูลมีปัญหา) · LOAD ทั้งไฟล์หรือไม่โหลด · IMPORT ข้ามแถว PROCESSED ทำซ้ำได้ไม่ซ้ำเอกสาร
  - Correct Import Errors (ไฟล์แถวผิดมี `__ROW_ID` ซ่อน + คอลัมน์ ERRORS, กัน formula injection), Delete Import Data, Purge มือ/อัตโนมัติ, audit ทุกขั้นพร้อม fingerprint ไม่เก็บค่าแถว
  - หน้า "งานนำเข้า" `/imports` และ `/imports/:id` · REST เดียวกันใช้กับระบบภายนอก
  - **ยังไม่ทำ:** (1) การจำกัดผู้เห็น interface ที่มีข้อมูลอ่อนไหว — ทำพร้อม object แรกที่มีข้อมูลแบบนั้น (ticket 41) · (2) จำนวนแถวตรวจตอน LOAD หลัง parse ไม่ใช่ก่อน parse (ขนาดไฟล์และขนาดหลังแตก zip ตรวจก่อนแล้ว) · หน้าอัปโหลด/แก้แถวผิดยังทดลองในเบราว์เซอร์จริงไม่ได้จนมี object จริง (ticket 42) เทสต์ API ครอบคลุมแล้ว
