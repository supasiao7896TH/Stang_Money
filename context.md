# context.md — GiGi Money

ภาพรวมโปรเจกต์สำหรับใครก็ตาม (คนหรือ AI agent) ที่เข้ามาทำงานในโค้ดนี้ครั้งแรก

## โปรเจกต์คืออะไร

**GiGi Money** (Stang_Money) เป็นแอปบันทึกรายรับ-รายจ่ายส่วนบุคคล ภาษาไทย รันบนเบราว์เซอร์ล้วน
ๆ ไม่มี backend, ไม่มี build step — ทั้งแอปอยู่ใน HTML ไฟล์เดียว

## โครงสร้างไฟล์

| ไฟล์ | หน้าที่ |
|---|---|
| `Stang_Money.html` | **ตัวแอปจริง** — React 18 (production minified bundle) + JSX ทั้งหมดอยู่ในไฟล์เดียว, โหลดผ่าน Babel standalone ในเบราว์เซอร์ ไม่มีขั้นตอน build |
| `index.html` | หน้า redirect เปล่า ๆ ไปที่ `Stang_Money.html` (ไว้ใช้กับ GitHub Pages root) |
| `manifest.json` | PWA manifest (ติดตั้งเป็นแอปบนมือถือได้) |
| `icon.svg` | ไอคอนแอป |
| `mockup_assets.html` | mockup/ร่าง UI ของฟีเจอร์ทรัพย์สิน (asset tracking) — **ไม่ใช่ production code** ใช้สำหรับดูตัวอย่างก่อนทำจริงเท่านั้น |
| `NATIVE_APP_GUIDE.md` | เอกสารอ้างอิงละเอียดสำหรับสร้าง native Android app ต่อจาก web app นี้ (data model, business logic, Gemini API, screens ฯลฯ) — อ่านเพิ่มถ้าต้องการรายละเอียดลึก |

ไม่มี `package.json`, ไม่มี dependency manager, ไม่มี test suite — แก้ `Stang_Money.html` แล้วเปิดในเบราว์เซอร์ได้ทันที

## สแตกเทคโนโลยี

- **React 18** — โหลดจาก CDN, ใช้ JSX ผ่าน Babel standalone (compile ใน runtime)
- **localStorage** — เก็บข้อมูลทั้งหมดไว้ในคีย์เดียว `stang_money_v1` (ไม่มี server, ไม่มี database)
- **Google Gemini API** (`generativelanguage.googleapis.com`) — ใช้วิเคราะห์รายจ่าย/อ่านใบเสร็จ ผู้ใช้ต้องใส่ API key เอง (เก็บใน settings ใน localStorage)
- ไม่มี TypeScript จริง (มีแค่ type comment/interface ใน docs เพื่ออธิบาย shape ของข้อมูล)
- ภาษาที่แสดงผล: ไทยล้วน, ใช้ปีพุทธศักราช (ค.ศ. + 543) สำหรับวันที่

## โมเดลข้อมูล (สรุปย่อ — รายละเอียดเต็มดู NATIVE_APP_GUIDE.md หัวข้อ 2)

เก็บทั้งหมดเป็น JSON object เดียวใน `localStorage["stang_money_v1"]`:

```json
{
  "transactions": [ /* Transaction[] */ ],
  "settings": { /* Settings */ },
  "goals": [ /* Goal[] */ ]
}
```

- **Transaction**: `id, date (ISO 8601), amount, type ("expense"|"income"), categoryId, note`, มี `recurringId` สำหรับรายการประจำ
- **Settings**: `geminiKey, geminiModel, monthStartDay` (รองรับรอบบิลที่ไม่เริ่มวันที่ 1)
- **Goal**: เป้าหมายการออม `id, name, target, saved, icon, deadline?`
- **Category IDs**: รายจ่ายขึ้นต้นด้วย `e_`, รายรับน่าจะขึ้นต้นด้วย `i_` (เช่น `i_asset_sale`) — รายการเต็มดูใน NATIVE_APP_GUIDE.md หัวข้อ 3

## ฟีเจอร์หลัก (เท่าที่ตรวจพบในโค้ด)

- หน้าหลัก (home) — สรุปยอดรายรับ-รายจ่าย
- รายการ (list) — list ธุรกรรม + รายการประจำ (`RecurringCard`, ย้ายมาอยู่หน้านี้ล่าสุด)
- เป้าหมาย (goal) — เป้าหมายการออม
- ทรัพย์สิน (assets) — ระบบบันทึกทรัพย์สิน (`AssetView`, `AssetRow`, `AssetDetail`, `AssetSheet`) ฟีเจอร์ใหม่
- ตั้งค่า (settings) — ตั้ง Gemini API key, model, วันเริ่มรอบบิล
- รองรับ "รอบบิล" (billing cycle) แบบกำหนดวันเริ่มต้นเองได้ (1–28)

## วิธีรันแอป / ทดสอบ

ไม่มี build/dev server — เปิดไฟล์ `Stang_Money.html` ตรง ๆ ในเบราว์เซอร์ หรือ serve ด้วย static server ธรรมดา เช่น

```sh
python3 -m http.server 8000
```

แล้วเข้า `http://localhost:8000/Stang_Money.html`

GitHub Pages ใช้ `index.html` (root) เป็นทางเข้า ซึ่ง redirect ไป `Stang_Money.html`

## หมายเหตุสำคัญเวลาแก้โค้ด

- `Stang_Money.html` เป็นไฟล์เดียวขนาดใหญ่ (~360KB) — โค้ด React/JSX ส่วนใหญ่ผ่านการ minify ตัวแปรสั้น ๆ (`A, J, K, N, P, R, T, _` ฯลฯ) จึงควรใช้ search (`grep`/text search) หาจุดที่จะแก้ แทนการอ่านทั้งไฟล์
- ข้อมูลผู้ใช้ทั้งหมดอยู่ใน `localStorage` ฝั่ง client — ไม่มี backend ให้แก้ไข
- `mockup_assets.html` เป็นแค่ draft UI ห้ามถือเป็น production code หรือ deploy
- ดู `NATIVE_APP_GUIDE.md` ก่อนถ้าต้องเข้าใจ business logic เชิงลึก (billing cycle, การกรอง transaction ตามรอบบิล, Gemini API integration, category IDs ทั้งหมด)
