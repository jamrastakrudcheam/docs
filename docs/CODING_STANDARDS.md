# มาตรฐานการเขียนโค้ดและความปลอดภัย — ICC ERP

ใช้กับทุก ticket ใน `.scratch/` และเป็นเกณฑ์ฝั่ง **Standards** ของ `/code-review`
ข้อที่ขึ้นต้นด้วย **ต้อง** เป็นข้อบังคับ ข้อที่เป็น "ควร" ใช้ดุลยพินิจได้ แต่ต้องอธิบายใน PR ถ้าไม่ทำ

อ้างอิง: ข้อตัดสินใจ D01–D15 / P01–P12 ใน `01-scope-decisions.md`, ADR ใน `docs/adr/`

---

## 1. สถาปัตยกรรม

- **ต้อง** ใช้ Modular Monolith: NestJS ตัวเดียว + PostgreSQL ตัวเดียว (D04, ADR-0001) ห้ามเพิ่ม message broker, outbox, หรือเรียก HTTP หาตัวเอง
- **ต้อง** ให้แต่ละโมดูลเป็นเจ้าของ schema ของตัวเอง (`org`, `iam`, `inventory`, `ap`, `gl` …) โมดูลอื่นห้ามเขียนตารางข้ามโมดูลตรง ๆ ให้เรียกผ่าน service ของเจ้าของ การอ่านข้ามโมดูลทำได้ผ่าน service หรือ view ที่เจ้าของเปิดให้
- **ต้อง** ให้คำสั่ง final (post, confirm, finalize, approve ที่เปลี่ยนยอด) ใช้ **transaction เดียว** ส่ง `trx` ผ่านทุก service ที่แตะ รวม SLA/GL ถ้าขั้นใดล้มต้อง rollback ทั้งชุด ห้าม service เปิด transaction ของตัวเองซ้อน
- **ต้อง** ล็อกทรัพยากรก่อนใช้ยอด (budget bucket, credit bucket, stock bucket, open item) และล็อกตามลำดับ key เดียวกันเสมอเพื่อเลี่ยง deadlock
- **ต้อง** แก้ข้อมูลที่ผ่านบัญชีแล้วด้วยเอกสารกลับรายการหรือเอกสารปรับปรุงที่อ้างต้นทาง ห้าม UPDATE/DELETE stock movement, journal ที่ post แล้ว หรือ settlement ที่ apply แล้วย้อนหลัง
- **ต้อง** แยก ICC/ITK เสมอ ทุก query ที่เกี่ยวกับยอดหรือรายงานต้องกรองด้วย company/ledger (ADR-0002) ห้ามรวมยอดข้ามบริษัท
- ควรให้ module มี interface เล็กแต่ลึก (deep module) ซ่อนกฎธุรกิจไว้ใน service ไม่กระจายไปที่ controller หรือ UI

## 2. ข้อมูลและฐานข้อมูล

- **ต้อง** เปลี่ยน schema ผ่าน migration ใหม่เท่านั้น (ลำดับเลข 4 หลัก) ห้ามแก้ migration ที่ merge แล้ว ต้องลงทะเบียนใน migrator และมี `down`
- **ต้อง** ใช้ `numeric(20,2)` สำหรับจำนวนเงิน และ `numeric(24,6)` สำหรับจำนวน/ต้นทุนต่อหน่วย (P08) ส่งเป็น **string** ทั้ง API และหน้าเว็บ ห้ามคำนวณยอดบัญชีด้วย JS `number`
- **ต้อง** ใช้ `date` สำหรับวันที่ทางธุรกิจ (business_date, gl_date) และคิดตามวันไทย (Asia/Bangkok) ส่วน `timestamptz` ใช้กับเวลาที่เกิดเหตุการณ์
- **ต้อง** ใช้ FK และ unique constraint กันข้อมูลผิด ไม่พึ่งแค่การเช็กในโค้ด เช่น ห้ามรับ/ส่ง/ออกบิลเกินยอด, AccountingEvent unique ต่อ source revision
- **ต้อง** สืบทอด company/ledger/OU ของบรรทัดจาก header และตรวจความสอดคล้องด้วย composite FK เมื่อทำได้ (ดูแบบอย่างที่ `org.operating_unit`)
- **ต้อง** มี `version` สำหรับข้อมูลที่แก้ได้ และใช้ `expected_version` ตรวจ stale edit (409)
- **ต้อง** ใช้รหัสเอกสาร/รหัสข้อมูลหลักเป็น string เพื่อรักษาเลขศูนย์นำหน้า
- ควรใช้ uuid เป็น primary key และไม่เปิดเลขลำดับภายในให้ผู้ใช้เดา

## 3. API

- **ต้อง** ทำตามสัญญาใน `05-api-contract.md`: prefix `/api/v1`, list มี `{items,page,page_size,total}` และจำกัด `page_size ≤ 100`, sort จาก allowlist
- **ต้อง** ตั้งชื่อ endpoint ตาม intent (`/{id}/submit`, `/{id}/post`, `/{id}/reverse`) ห้ามมี generic `PATCH status=POSTED`
- **ต้อง** validate body ทุกตัวด้วย zod ผ่าน `parseBody` และ id ผ่าน `parseId` ห้ามรับ object ดิบเข้าฐานข้อมูล
- **ต้อง** คืน error รูปแบบ `{code, message, fields:[{path, rule, message}], correlation_id}` ใช้ helper ใน `common/errors` (`fieldRule`, `notFound`, `staleVersion`, `conflict`) และใช้รหัส: 400 malformed, 401, 403, 404, 409 state/stale/idempotency, 422 กฎธุรกิจ
- **ต้อง** ให้คำสั่ง final รับ `Idempotency-Key` บันทึกผลใน transaction เดียวกับงาน: key เดิม + payload เดิม คืนผลเดิม, key เดิม + payload ต่าง คืน 409
- **ต้อง** คำนวณยอดเงิน ภาษี และ Dr/Cr ฝั่ง server จาก snapshot ที่อนุมัติแล้ว ห้ามเชื่อยอดรวมที่ browser ส่งมา
- **ต้อง** ตอบ success เฉพาะเมื่อ commit สำเร็จแล้ว
- ข้อความ error ที่ผู้ใช้เห็นเป็นภาษาไทย ส่วน `code`/`rule` เป็นภาษาอังกฤษแบบ snake/upper case เพื่อให้โปรแกรมอ่านได้

## 4. สิทธิ์และการแบ่งแยกหน้าที่ (Authorization / SoD)

- **ต้อง** ปฏิเสธเป็นค่าเริ่มต้น: ทุก endpoint ต้องมี session ยกเว้นที่ติด `@Public()` ซึ่งต้องมีเหตุผลเขียนไว้
- **ต้อง** ตรวจสิทธิ์ที่ API ทุกครั้ง การซ่อนปุ่มบนหน้าเว็บไม่นับเป็นการควบคุมสิทธิ์
- **ต้อง** ตรวจบทบาท + บริษัท + OU จาก scope grant ที่มีผลวันนี้ (`effectiveScopes`) และใช้ company/OU จาก session หรือจากเอกสารที่ตรวจแล้ว ห้ามเชื่อ company/OU ที่ browser ส่งมาโดยไม่ตรวจ
- **ต้อง** บังคับ maker ≠ checker: ผู้สร้างหรือผู้แก้ล่าสุดอนุมัติเอกสารตัวเองไม่ได้ รวมถึงผ่านการมอบอำนาจ (D07)
- **ต้อง** ทำให้การแก้สาระสำคัญหลังส่งอนุมัติสร้าง revision ใหม่และยกเลิกการอนุมัติเดิม
- **ต้อง** ไม่ให้ System Admin เป็นผู้อนุมัติธุรกรรมการเงินโดยปริยาย
- ควรตั้งชื่อ permission ตามการกระทำ เช่น `po.approve`, `gl.journal.post` เมื่อเริ่มมีโมดูลธุรกรรม

## 5. ความปลอดภัย

### ตัวตนและ session
- **ต้อง** เก็บรหัสผ่านด้วย scrypt ผ่าน `auth/credentials` เท่านั้น ห้าม log, คืนค่า, หรือใส่ hash ลง audit
- **ต้อง** ใช้ session cookie แบบ `HttpOnly` + `SameSite=Lax` และเปิด `Secure` เมื่อรันบน HTTPS (`COOKIE_SECURE=true`) ห้ามเก็บ token ใน localStorage
- **ต้อง** ตัด session ทุกเครื่องเมื่อรีเซ็ตรหัสผ่าน ปิดใช้งานผู้ใช้ หรือถอนสิทธิ์สำคัญ
- **ต้อง** ตอบ login ผิดด้วยข้อความเดียวกันไม่ว่าชื่อผู้ใช้มีอยู่หรือไม่

### อินพุตและ query
- **ต้อง** ใช้ Kysely หรือ template `sql\`\`` ที่ผูกพารามิเตอร์เสมอ ห้ามใช้ `sql.raw` หรือต่อ string กับค่าที่มาจากผู้ใช้ ชื่อตาราง/คอลัมน์แบบ dynamic ต้องมาจาก allowlist ในโค้ด
- **ต้อง** จำกัดความยาวและรูปแบบของทุก field ใน schema zod
- **ต้อง** ให้ redirect หลัง login (`next`) เป็น path ภายในเท่านั้น

### ข้อมูลอ่อนไหว
- **ต้อง** ไม่ commit ความลับ (`.env`, key, รหัสผ่านจริง) ค่าตัวอย่างใส่ใน `.env.example` เท่านั้น
- **ต้อง** ไม่ใส่ข้อมูลส่วนบุคคลหรือข้อมูลการเงินใน URL query string และใน log
- **ต้อง** แสดงเลขบัญชีธนาคารผู้ขายแบบ mask และจำกัดผู้แก้ไขให้แยกจากผู้สร้าง Supplier (03, authorization matrix)
- **ต้อง** ไม่ส่ง stack trace กลับไปที่ client ให้ log ฝั่ง server พร้อม `correlation_id`

### audit
- **ต้อง** บันทึก audit ทุกการ create/update/revoke/post/approve ใน transaction เดียวกับงาน พร้อม actor จริง ข้อมูล before/after ห้ามมีความลับ
- audit เป็น append-only สำหรับแอป (ไม่มี endpoint แก้/ลบ)

### นำเข้าไฟล์
- **ต้อง** นำเข้าไฟล์ทุกชนิดผ่านบริการนำเข้ากลาง (ticket 36, `.scratch/erp-phase1/import-framework.md`) โดยลงทะเบียน import object ห้ามเขียน endpoint อัปโหลดแยกในโมดูล
- **ต้อง** ให้ handler ของ import object สร้างเอกสารผ่าน service ของโมดูลเจ้าของ และใช้กฎตรวจชุดเดียวกับหน้าจอคีย์
- **ต้อง** ทำตามข้อความปลอดภัยใน import-framework §5 (magic bytes, zip limits, ไม่ประมวลผลสูตร/macro, กัน formula injection ในไฟล์ที่ส่งกลับ)

### ระบบภายนอกและไฟล์
- **ต้อง** จำลองธนาคาร, POS, E-Tax ในเครื่องตาม D03 ห้ามเรียกระบบจริงหรือส่งอีเมลจริงภายใน transaction
- **ต้อง** ตรวจชนิด ขนาด และชื่อไฟล์ที่อัปโหลด เก็บนอก web root ใช้ชื่อไฟล์ที่ระบบสร้าง และเก็บ fingerprint ของไฟล์ statement ที่นำเข้า

### dependency
- ควรรัน `npm audit` ใน CI และอัปเดตแพ็กเกจที่มีช่องโหว่ระดับ high ขึ้นไปก่อน merge
- ห้ามเพิ่มแพ็กเกจที่มี native build หรือ install script โดยไม่จำเป็น

## 6. Frontend (SvelteKit)

- **ต้อง** ใช้ Svelte 5 runes (`$state`, `$derived`, `$props`) และ TypeScript ไม่มี `any` โดยไม่มีเหตุผล
- **ต้อง** เรียก API ผ่าน `$lib/api` (จัดการ 401/403 และรูปแบบ error ให้แล้ว) ห้ามเรียก fetch ตรงในหน้า
- **ต้อง** แสดงสถานะด้วยข้อความ ไม่ใช้สีอย่างเดียว ฟอร์มต้องบอก field ที่จำเป็น แสดง error ใต้ field และเก็บค่าที่กรอกไว้เมื่อบันทึกไม่ผ่าน
- **ต้อง** ทำให้ตารางมีสถานะ loading/empty/error, ตัวเลขจัดชิดขวา, รหัสใช้ฟอนต์ mono
- **ต้อง** ใช้ `Modal` (native dialog) และให้ทุกปุ่ม/ช่องใช้คีย์บอร์ดได้ มี label ที่ screen reader อ่านได้
- **ต้อง** ทำให้หน้าธุรกรรมอ่านง่ายบน desktop/tablet และหน้าอนุมัติใช้ได้บนมือถือ (D06) ไม่มี horizontal scroll ทั้งหน้า
- **ต้อง** ไม่แสดงตัวเลขตัวอย่างเหมือนข้อมูลจริง ถ้ายังไม่มีข้อมูลให้แสดง empty state

## 7. การทดสอบ

- **ต้อง** เขียนเทสต์ที่ seam ของ REST API (Nest app + PostgreSQL จริงในฐาน `icc_erp_test`) เป็นหลัก ทดสอบพฤติกรรมภายนอก ไม่ทดสอบรายละเอียดภายใน
- **ต้อง** มีเทสต์ทุกกฎธุรกิจที่ ticket ระบุ รวมกรณีขอบ (ยอดพอดีวงเงิน, ส่งบางส่วน, ซ้ำ, ข้ามบริษัท, stale version, ไม่มีสิทธิ์)
- **ต้อง** มีเทสต์ concurrency หรือ double-submit สำหรับคำสั่ง final (idempotency, lock)
- **ต้อง** ใช้ข้อมูลทดสอบสังเคราะห์และรีเซ็ตได้ (`freshStart`, `companyWithOus`) ห้ามพึ่งข้อมูลใน dev DB
- ควรเขียนเทสต์ก่อน (`/tdd`) และตั้งชื่อเทสต์เป็นประโยคที่อธิบายกฎ
- **ต้อง** ให้ CI (GitHub Actions) เขียวก่อน merge: lint, typecheck API/web, e2e, build

## 8. Git และการทำงาน

- **ต้อง** แยก branch ต่อ ticket (`<NN>-<slug>`) และไม่ commit ตรงเข้า `main`
- **ต้อง** อ้างเลข ticket ใน PR และให้ PR ผ่าน `/code-review` ทั้งสองแกน (Standards, Spec)
- **ต้อง** อัปเดต `Status:` ของ ticket และเพิ่มบันทึกใต้ `## Comments` เมื่อเริ่มหรือจบงาน
- ควรเขียน commit message บรรทัดแรกเป็นภาษาอังกฤษแบบสั่งงาน (เช่น "Add PO revision with approval reset") และอธิบายเหตุผลในเนื้อหา
- ควรเขียน comment อธิบาย "ทำไม" ไม่ใช่ "ทำอะไร" และอ้างรหัส requirement (D/P/T) เมื่อโค้ดบังคับกฎนั้น
