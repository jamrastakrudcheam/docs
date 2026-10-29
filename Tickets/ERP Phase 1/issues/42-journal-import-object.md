---
ticket: 42
phase: GL & Tax
module: GL
screens: ["G03 Journal: ดาวน์โหลด template / อัปโหลด", "งานนำเข้า"]
refs: ["D13", "T08"]
blocked_by: [12, 36]
tags:
  - erp/ticket
  - erp/gl-&-tax
---

# 42: Journal Import (import object แรกของบริการกลาง)

**Status:** ready-for-agent
**Type:** task
**Blocked by:** [12 GL: คีย์ Journal, อนุมัติ, Post, Reverse และ Trial Balance](12-gl-journal-entry-posting.md), [36 บริการนำเข้าไฟล์กลาง (แบบ Oracle FBDI)](36-file-based-data-import.md)
**Spec:** [spec](../spec.md) · **มาตรฐาน:** [CODING_STANDARDS](../../../docs/CODING_STANDARDS.md)

## What to build

ลงทะเบียน import object `GL_JOURNAL_IMPORT` กับบริการกลาง (ticket 36) แบบ Fusion Journal Import: interface `GL_INTERFACE`, รวมแถวเป็น Journal ตาม batch name + journal name + บริษัท + source + category + accounting date, ทั้ง load ผ่านหรือไม่ผ่าน, สร้าง Journal (source = Spreadsheet) ผ่าน service ของ GL (ticket 12) (gl-process §5, import-framework §2–3)

หน้าจอ: G03 Journal: ดาวน์โหลด template / อัปโหลด, งานนำเข้า
อ้างอิง: D13, T08

## ขอบเขต

- template: [journal-upload-template.xlsx](../../../docs/templates/journal-upload-template.xlsx) (แท็บ `GL_INTERFACE`) — เมื่อ ticket 36 เสร็จ ให้สร้างจาก registry แทนไฟล์ที่ทำมือ
- handler: กฎตรวจเดียวกับหน้าจอคีย์ (ticket 12) + header ในกลุ่มต้องตรงกัน + ข้อมูลภาษีตาม gl-process §8
- ตัวเลือกหลัง import: ค้าง Unposted, submit ทันที, หรือ import and post (เฉพาะผู้มีสิทธิ์ Post และ Journal ที่ไม่ต้องอนุมัติ) ตาม Fusion

## Acceptance criteria

- [ ] ไฟล์ที่มีแถวผิดแม้แถวเดียวไม่สร้าง Journal ใดเลย และบอกเลขแถว/คอลัมน์/เหตุผลเป็นภาษาไทย
- [ ] ไฟล์ถูกต้องสร้าง Journal ตามจำนวนกลุ่ม ยอดและข้อมูลภาษีตรงกับไฟล์
- [ ] Journal ที่ไม่ balance ทำให้ทั้ง load ไม่ผ่าน (เหมือน workbook upload ของ Fusion)
- [ ] Journal จาก import แก้ในหน้าจอได้ตามสถานะปกติ และไล่กลับถึง load request ได้

## เทสต์ที่ต้องมี

- [ ] e2e: ไฟล์ถูก, แถวผิดหลายชนิด, ไม่ balance ต่อกลุ่ม, header ไม่ตรงในกลุ่มเดียว, บริษัทนอกสิทธิ์, import and post ตามสิทธิ์
- [ ] ใช้ไฟล์ทดสอบที่สร้างจาก template จริง

## มาตรฐานโค้ดและความปลอดภัย

- [ ] ผ่าน checklist ทั่วไปใน CODING_STANDARDS (transaction เดียว, string decimal, zod, error contract, scope จาก session, audit, CI เขียว)
- [ ] import and post ต้องผ่านกฎอนุมัติเหมือนคีย์มือ ไม่มีทางลัด
- [ ] ผู้อัปโหลดต้องมีสิทธิ์ GL ในบริษัทของทุกแถว
- [ ] ผ่าน `/code-review` ทั้ง Standards และ Spec

## Comments
