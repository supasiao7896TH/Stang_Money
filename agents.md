# agents.md

คำแนะนำสำหรับ AI coding agent (เช่น Claude Code) ที่เข้ามาทำงานในโปรเจกต์นี้ อ่าน `context.md` ก่อนเพื่อความเข้าใจภาพรวม ไฟล์นี้เน้นกฎ/ขั้นตอนการทำงานจริง

## โครงสร้างโปรเจกต์แบบย่อ

- ไม่มี build step, ไม่มี package manager, ไม่มี test framework
- `Stang_Money.html` คือตัวแอปจริงทั้งหมด (React + JSX แบบ in-browser, ไม่ผ่าน bundler)
- ห้ามเพิ่ม build tooling (webpack, vite, npm scripts ฯลฯ) โดยไม่ได้รับการร้องขอจากผู้ใช้ก่อน — สถาปัตยกรรม "single HTML file ไม่มี build" เป็นการตัดสินใจที่ตั้งใจไว้

## วิธีรัน / ตรวจสอบงาน

ไม่มี automated test suite ตรวจความถูกต้องด้วยการรันแอปจริงในเบราว์เซอร์:

```sh
python3 -m http.server 8000
# เปิด http://localhost:8000/Stang_Money.html
```

ถ้าใช้ Playwright/headless browser ให้เปิดไฟล์นี้ตรง ๆ แล้วทดสอบ flow ที่เกี่ยวข้องกับสิ่งที่แก้ (เช่น เพิ่มรายการ, ดูสรุป, ตั้งค่า) ก่อนรายงานว่าเสร็จ

## กฎการแก้โค้ด

- แก้เฉพาะส่วนที่เกี่ยวข้องกับ task เท่านั้น — ไฟล์ใหญ่และ minified บางส่วน อย่า reformat/minify เพิ่มเติมโดยไม่จำเป็น
- ใช้ `grep`/text search หาตำแหน่งฟังก์ชัน/component ก่อนแก้ เพราะไฟล์ยาวและตัวแปรสั้น (`A, J, K, N, P, R, T, _`)
- ข้อความ UI ทั้งหมดเป็นภาษาไทย และใช้ปีพุทธศักราช (ค.ศ. + 543) สำหรับการแสดงวันที่ — รักษารูปแบบนี้ไว้เมื่อเพิ่ม/แก้ข้อความหรือ logic วันที่
- ข้อมูลทั้งหมดอยู่ใน `localStorage` คีย์ `stang_money_v1` เป็น object เดียว (`transactions`, `settings`, `goals`, ...) — ถ้าเพิ่ม field ใหม่ ต้องคิดเรื่อง backward compatibility กับข้อมูลเก่าของผู้ใช้ที่มีอยู่แล้ว (อย่าทำให้ข้อมูลเดิมพังหรือหาย)
- `mockup_assets.html` เป็น draft/reference เท่านั้น ห้ามอ้างอิงว่าเป็น production หรือ deploy แทน `Stang_Money.html`
- ถ้าเพิ่ม category ID ใหม่ ให้ตามรูปแบบเดิม: รายจ่ายขึ้นต้น `e_`, รายรับขึ้นต้น `i_`
- Gemini API key ของผู้ใช้เก็บใน `settings.geminiKey` (localStorage) — ห้าม hardcode API key ใด ๆ ลงในโค้ด และห้าม log/ส่ง key ออกไปที่อื่นนอกจาก Gemini API ตรง ๆ

## เอกสารอ้างอิงเพิ่มเติม

- `context.md` — ภาพรวมโปรเจกต์, โครงสร้างไฟล์, สแตก, โมเดลข้อมูลแบบย่อ
- `NATIVE_APP_GUIDE.md` — รายละเอียดเชิงลึก: data model เต็ม, category IDs ทั้งหมด, business logic (billing cycle, การกรอง transaction), Gemini API integration, รายชื่อหน้าจอ — อ่านก่อนแก้ business logic ที่ซับซ้อน (เช่น การคำนวณรอบบิล, สรุปรายเดือน)

## Git / commit

- repo นี้ commit message ส่วนใหญ่เป็นภาษาไทย ใช้ prefix แบบ Conventional Commits (`feat:`, `fix:`, `refactor:`, `chore:`) ตามที่เห็นใน `git log` — ทำตามรูปแบบเดิม
- ก่อน commit งานที่แตะ business logic (วันที่, รอบบิล, การคำนวณยอด) ให้ทดสอบใน browser จริงตามหัวข้อ "วิธีรัน / ตรวจสอบงาน" ก่อน เพราะไม่มี automated test คอยจับ regression
