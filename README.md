# 🧠 PKM Engine — ระบบจัดการความรู้ส่วนตัว

> Single-user MVP · เวอร์ชันปัจจุบัน **v1.3.1** (2026-08-01) · Frontend ไฟล์เดียว + Google Sheets เป็น Database

---

## 1. โปรเจกต์นี้คืออะไร ใช้แก้ปัญหาอะไร

PKM Engine เป็นเว็บแอปจัดการความรู้ส่วนตัว (Personal Knowledge Management) สำหรับผู้ใช้คนเดียว ใช้แก้ปัญหาที่พบบ่อยของการเรียนรู้ด้วยตัวเอง คือเก็บความรู้ไว้เยอะแต่ไม่ได้ย่อย ลืมเร็ว และไม่ได้เอาไปใช้จริง ระบบจึงพาโน้ตแต่ละชิ้นเดินตาม pipeline แบบ Kanban 5 ขั้น (เข้าใหม่ → วางแผนแล้ว → กำลังเรียน → สรุปแล้ว → พร้อมใช้) ตามแนวคิด CODE Method / Second Brain และมี WIP limit กันไม่ให้เรียนหลายเรื่องพร้อมกัน เมื่อโน้ตถึงขั้น "สรุปแล้ว" จะเข้าระบบทบทวนแบบ flashcard ที่ใช้อัลกอริทึม SM-2 spaced repetition นอกจากนี้ยังทำ cross-reference และ backlinks ระหว่างโน้ต (แนว Zettelkasten) เขียนสรุปเป็น Markdown ได้ บันทึกจำนวนครั้งที่นำความรู้ไปใช้ และดูสถิติได้ ข้อมูลทั้งหมดเก็บใน Google Sheets ผ่าน Google Apps Script API จึงเปิดดูหรือแก้ข้อมูลดิบใน Sheet ได้โดยตรง

---

## 2. Tech stack

| ส่วน | เทคโนโลยี | หมายเหตุ |
|---|---|---|
| Frontend | HTML + CSS + **Vanilla JavaScript** (ES2020+: `async/await`, optional chaining) | ไม่ใช้ framework ไม่มี build step ไม่มี `package.json` |
| Styling | CSS custom properties (design tokens), Flexbox, Grid | มี Light/Dark theme |
| Font | IBM Plex Sans Thai + IBM Plex Mono (Google Fonts) | โหลดจาก CDN ต้องต่อ internet |
| Backend / API | **Google Apps Script** Web App (`script.google.com/macros/s/.../exec`) | ⚠️ source code ของ backend **ไม่อยู่ใน repo นี้** |
| Database | **Google Sheets** | ชื่อ tab และลำดับคอลัมน์ต้องยืนยัน (ดูหัวข้อ 4.3) |
| Authentication | API Token แบบ shared secret เก็บใน `localStorage` (`pkm_token`) | ส่งไปกับทุก request |
| Algorithm | SM-2 spaced repetition (quality 1/3/4/5) | เขียนเองในไฟล์ ไม่ใช้ library |
| Markdown | Mini parser ที่เขียนเอง (`mdToHtml`) | รองรับ syntax พื้นฐานเท่านั้น |
| AI helper | Gemini Gem (ปุ่ม ✨) | เป็นแค่ลิงก์เปิดแท็บใหม่ ไม่มี API integration |
| Hosting | GitHub Pages (อ้างอิงจาก changelog v1.0.0) | URL จริงต้องยืนยัน |
| Browser APIs | Fetch, `localStorage`, HTML5 Drag and Drop, `window.storage` | `window.storage` ไม่ใช่ API มาตรฐานของ browser (ดูหัวข้อ 5) |

---

## 3. โครงสร้างโฟลเดอร์

```
pkm-bk/
├── index.html   # ทั้งแอปอยู่ในไฟล์นี้: HTML + CSS + JavaScript (~1,495 บรรทัด, ~90 KB)
└── README.md    # เอกสารนี้
```

**ไฟล์หรือส่วนที่โค้ดอ้างถึงแต่ไม่มีใน repo**

- `DEPLOY.md` มีคอมเมนต์ในโค้ดอ้างถึง ("ใส่ค่าจากขั้นที่ 5-7 ใน DEPLOY.md") แต่ไม่มีไฟล์นี้ใน repo
- Google Apps Script (backend) ที่รับ action `ping`, `list`, `saveAll`, `logReview`
- Google Sheet ที่เป็น Database (ลิงก์อยู่ในค่าคงที่ `SHEET_URL`)

### โครงสร้างภายใน `index.html`

| บรรทัด (โดยประมาณ) | ส่วน | คำอธิบาย |
|---|---|---|
| 1–13 | Header comment | ชื่อระบบ + **Changelog** (v1.0.0 → v1.3.1) |
| 14–17 | `<link>` | Favicon 🧠 (inline SVG) + Google Fonts |
| 18–385 | `<style>` | Design tokens (light/dark), สไตล์ของ header, board, review, stats, ตาราง, modal, Markdown editor, หน้า login |
| 387–408 | `<header>` | เมนู 4 แท็บ, sync pill, due pill, ปุ่ม theme / เพิ่มโน้ต / 📊 Sheet / ✨ Gem / 🔓 logout |
| 410–430 | `<main>` + overlay | หน้า login (กรอก token), container ของ 4 view, modal |
| 433–480 | **Config** | `STATES` (5 ขั้น pipeline), `NOTE_TYPES` (4 ประเภท), `WIP_LIMIT`, `PKM_VERSION`, `SHEET_URL`, `GEM_URL` |
| 482–583 | **API / Storage layer** | `API.url`, `apiGet`, `apiPost`, `loadData`, `saveNotes` (ส่ง `saveAll`), `pushReviewLog` |
| 585–616 | Date helpers + **SM-2** | `todayISO`, `addDaysISO`, `isDue`, `sm2()` |
| 618–788 | แท็บ **บอร์ด** (Kanban) | Drag-and-drop ข้ามคอลัมน์ และเรียงลำดับในคอลัมน์ (`sortOrder`), ตรวจ WIP limit, ปุ่ม ← → |
| 790–836 | แท็บ **ทบทวน** | Flashcard เปิดดูคำตอบ แล้วให้คะแนน 4 ระดับ → คำนวณ SM-2 → บันทึก review log |
| 838–922 | แท็บ **สถิติ** | KPI 4 ช่อง, กราฟคาดการณ์การทบทวน, กราฟแยกตาม state / หมวดหมู่, top 5 โน้ตที่ถูกใช้บ่อย |
| 924–1087 | แท็บ **โน้ตทั้งหมด** | Dashboard card, ช่องค้นหา, filter (สถานะ/หมวด/แหล่ง/ประเภท), ตารางที่ sort ได้ |
| 1089–1373 | **Modal ฟอร์มโน้ต** | เพิ่ม/แก้ไข/คัดลอก/ลบ, cross-reference + backlinks, โน้ตแม่ (overview) รวมโน้ตลูก, ลิงก์เสริม + ผลงาน (มี image preview สำหรับ Google Drive/Imgur), ปุ่มบันทึกการใช้งาน |
| 1375–1427 | Mini Markdown parser | `mdInline`, `mdToHtml`, split preview, `safeUrl` (อนุญาตเฉพาะ http/https) |
| 1429–1450 | Seed data | โน้ตตัวอย่าง 5 รายการ (ใช้เฉพาะ local mode) |
| 1451–1494 | Login + init | `submitToken` (ping), `logout`, `startApp`, `init()` |

---

## 4. วิธีติดตั้งและรันบนเครื่อง

### 4.1 สิ่งที่ต้องมี

- Git
- Web browser รุ่นใหม่ (Chrome / Edge / Firefox / Safari)
- Python 3 **หรือ** Node.js สำหรับเปิด local web server (ไม่ต้องติดตั้ง dependency ใด ๆ)
- Internet (โหลด Google Fonts และเรียก Apps Script API)
- **API Token** ของ backend (ขอจากเจ้าของระบบ ส่วนตำแหน่งที่เก็บ token ฝั่ง Apps Script **ต้องยืนยัน**)

> ⚠️ **ข้อควรระวัง:** `API.url` ใน `index.html` ชี้ไปที่ Apps Script ตัว production อยู่แล้ว ถ้ารันบนเครื่องด้วย token จริง **ทุกการแก้ไขจะเขียนลง Google Sheet จริงทันที** ถ้าต้องการทดลอง แนะนำให้ทำ backend สำเนาตามหัวข้อ 4.3

### 4.2 รันบนเครื่อง (ใช้ backend เดิม)

1. Clone repo

   ```bash
   git clone https://github.com/BK-19/pkm-bk.git
   cd pkm-bk
   ```

2. เปิด local web server (เลือกอย่างใดอย่างหนึ่ง)

   ```bash
   # Python 3
   python3 -m http.server 8000

   # หรือ Node.js
   npx --yes http-server -p 8000
   ```

3. เปิด browser ไปที่ `http://localhost:8000`
4. จะเจอหน้า **🔐 เชื่อมต่อระบบ** ให้กรอก API Token แล้วกด **เชื่อมต่อ**
   - ระบบจะเรียก `GET ?action=ping` เพื่อตรวจ token
   - ถ้าผ่าน จะเก็บ token ใน `localStorage` (key `pkm_token`) แล้วโหลดโน้ตจาก Sheet
5. ใช้งานได้ทันที ถ้าต้องการล้าง token ให้กดปุ่ม 🔓 มุมขวาบน

> การเปิดไฟล์ตรง ๆ ผ่าน `file://` ยังไม่ได้ทดสอบว่า Apps Script ยอมรับ request จาก origin `null` หรือไม่ (**ต้องยืนยัน**) จึงแนะนำให้ใช้ local server ตามขั้นที่ 2

### 4.3 ใช้ backend ของตัวเอง (ทางเลือก)

source code ของ Apps Script ไม่อยู่ใน repo ข้อมูลด้านล่างจึง **reverse จากโค้ดฝั่ง frontend** และต้องยืนยันกับ backend ตัวจริงอีกครั้ง

1. สร้าง Google Sheet สำหรับเก็บโน้ต และ review log (ชื่อ tab และลำดับคอลัมน์ **ต้องยืนยัน**)
2. สร้าง Apps Script ที่รองรับ API contract ต่อไปนี้

   | Method | Request | Response ที่ frontend คาดหวัง |
   |---|---|---|
   | `GET` | `?action=ping&token=…` | `{ "ok": true }` หรือ `{ "ok": false, "error": "…" }` |
   | `GET` | `?action=list&token=…` | `{ "ok": true, "notes": [ … ] }` |
   | `POST` | body (`text/plain`, JSON): `{ token, action: "saveAll", notes: [ … ] }` | `{ "ok": true, "saved": <จำนวน> }` |
   | `POST` | body: `{ token, action: "logReview", entry: { noteId, date, quality, efAfter, intervalAfter, nextReview } }` | `{ "ok": true }` |

   > POST ส่งเป็น `text/plain` เพื่อเลี่ยง CORS preflight (ตามคอมเมนต์ในโค้ด)

3. Deploy เป็น Web App (ค่าที่ใช้จริง เช่น *Execute as* และ *Who has access* **ต้องยืนยัน**)
4. แก้ค่าคงที่ใน `index.html`
   - `API.url` → URL `/exec` ของ Web App ใหม่
   - `SHEET_URL` → ลิงก์ Sheet ใหม่ (ใช้กับปุ่ม 📊)
   - `GEM_URL` → ลิงก์ Gemini Gem (ใช้กับปุ่ม ✨)

**Data model ของโน้ต 1 รายการ** (object ใน `notes[]`)

| Field | ความหมาย |
|---|---|
| `id` | รหัสโน้ต รูปแบบ `N0001` (สร้างจากเลขสูงสุดที่มีอยู่ + 1) |
| `status` | `inbox` / `planned` / `learning` / `distilled` / `active` |
| `title`, `noteType`, `source`, `priority`, `category` | หัวข้อ, ประเภท (`overview`/`technique`/`idea`/`reference`), แหล่ง (`web`/`youtube`/`book`/`course`/`other`), ความสำคัญ (`high`/`med`/`low`), หมวดหมู่ |
| `url` | ลิงก์แหล่งที่มาหลัก |
| `attachments[]`, `outputs[]` | ลิงก์เสริม / ผลงานที่สร้าง เก็บเป็น string `"ป้ายชื่อ\tURL"` |
| `summary`, `technique` | สรุป (Markdown), เทคนิคที่นำไปใช้ได้ |
| `links[]` | ID ของโน้ตที่เชื่อมโยง (backlinks คำนวณตอนแสดงผล) |
| `created`, `learnDate` | วันที่สร้าง, วันที่วางแผนเรียน (`YYYY-MM-DD`) |
| `reps`, `ef`, `interval`, `lastReview`, `nextReview`, `lastQuality` | สถานะของ SM-2 |
| `timesUsed`, `lastUsed` | สถิติการนำไปใช้ |
| `sortOrder` | ลำดับการ์ดในคอลัมน์ Kanban |

### 4.4 ปรับแต่งค่าที่ใช้บ่อย (ใน `index.html`)

- `WIP_LIMIT` กำหนดจำนวนการ์ดสูงสุดต่อคอลัมน์ (ค่าปัจจุบัน `planned: 10`, `learning: 3`)
- `NOTE_TYPES` ใช้เพิ่มหรือแก้ประเภทโน้ต (key ต้องไม่ซ้ำ)
- `STATES` คือขั้นของ pipeline ถ้าแก้ต้องแก้ CSS variable `--s-<key>` ให้ตรงกันด้วย
- `PKM_VERSION` ให้ bump ทุกครั้งที่แก้โค้ด และเพิ่มบรรทัดใน Changelog ที่หัวไฟล์

### 4.5 Deploy ขึ้น GitHub Pages

1. ไปที่ **Settings → Pages** ของ repo
2. Source เลือก **Deploy from a branch** → branch `main` / folder `/ (root)`
3. รอ build แล้วเปิด URL ที่ได้ (น่าจะเป็น `https://bk-19.github.io/pkm-bk/` แต่ **ต้องยืนยัน**)

---

## 5. สิ่งที่ยังไม่สมบูรณ์หรือควรปรับปรุง

รายการนี้มาจากการอ่านโค้ด `index.html` v1.3.1 ข้อที่ระบุว่า "ยืนยันแล้ว" ได้ทดสอบจริงด้วย Node.js หรือ headless Chromium

### 🔴 ควรแก้ก่อน (กระทบความถูกต้องของข้อมูล)

1. **Bug เรื่อง timezone ใน date helper (ยืนยันแล้ว)**
   `todayISO()` และ `addDaysISO()` ใช้ `toISOString()` ซึ่งเป็นเวลา **UTC** ทั้งที่คอมเมนต์เขียนว่า "day-level, local" เมื่อรันใน timezone ไทย (UTC+7) จะเกิดปัญหาดังนี้
   - `addDaysISO('2026-09-25', 1)` ได้ `'2026-09-25'` (ควรเป็น `'2026-09-26'`) ทุก interval จึง**สั้นลง 1 วัน**
   - กด "ลืมแล้ว" หรือ "ยาก" (interval = 1) แล้ว `nextReview` กลายเป็นวันนี้ โน้ตจึงยังค้างสถานะถึงกำหนดทบทวน
   - ช่วง 00:00–06:59 น. `todayISO()` คืนค่าเป็น**วันของเมื่อวาน**
   - วันที่บนแกน x ของกราฟคาดการณ์ และ `learnDate` (ที่ตั้งใจให้เป็นพรุ่งนี้) คลาดไป 1 วัน
   - แนวทางแก้: สร้างสตริงวันที่จาก `getFullYear()`/`getMonth()`/`getDate()` แทน `toISOString()`
2. **`saveAll` ส่งโน้ตทั้งหมดทุกครั้งที่บันทึก และอาจเขียนทับข้อมูล**
   - ถ้า `loadData()` ล้มเหลว (เช่น network หลุด) ระบบจะตั้ง `notes = []` แล้วยังให้ใช้งานต่อได้ ถ้าผู้ใช้เพิ่มโน้ตตอนนั้น `saveAll` อาจเขียนทับ Sheet ให้เหลือแค่โน้ตใหม่ (พฤติกรรมจริงขึ้นกับ backend ซึ่ง**ต้องยืนยัน**)
   - ไม่มี conflict control ถ้าเปิดหลายแท็บหรือหลายเครื่อง ข้อมูลที่บันทึกทีหลังจะทับของก่อนหน้า
   - payload โตตามจำนวนโน้ต ยิ่งโน้ตเยอะยิ่งบันทึกช้า
3. **Backend และเอกสาร deploy ไม่อยู่ใน repo**
   ไม่มี source ของ Apps Script, schema ของ Sheet และ `DEPLOY.md` (ที่โค้ดอ้างถึง) จึง deploy ใหม่หรือกู้ระบบจาก repo นี้อย่างเดียวไม่ได้
4. **ความปลอดภัยของ token และ URL**
   - `GET` (`ping`, `list`) ส่ง token ผ่าน query string ทำให้ token ไปปรากฏใน URL log ได้
   - token เก็บใน `localStorage` แบบ plaintext
   - `API.url`, `SHEET_URL` และ Sheet ID ถูก hardcode ไว้ในโค้ด ถ้า repo เป็น public คนอื่นจะเห็นด้วย (สถานะ public/private ของ repo และสิทธิ์การแชร์ Sheet **ต้องยืนยัน**)

### 🟡 ควรปรับปรุง (Bug เล็ก / UX / คุณภาพโค้ด)

5. **HTML ไม่มีโครงสร้างมาตรฐาน (ยืนยันแล้ว)** ไฟล์ไม่มี `<!DOCTYPE html>`, `<html lang="th">`, `<head>`, `<meta charset="utf-8">`, `<meta name="viewport">` และ `<title>` ผลที่ตามมาคือ
   - browser render ใน **quirks mode** (`document.compatMode === "BackCompat"`)
   - แท็บ browser ไม่มีชื่อหน้า
   - บนมือถือ media query `max-width: 640px` ไม่ทำงานตามที่ออกแบบ เพราะไม่มี viewport meta
6. **`window.storage` ไม่มีใน browser ทั่วไป (ยืนยันแล้ว)** ตัวแปรนี้เป็น `undefined` เมื่อเปิดใน Chromium ปกติ (น่าจะเป็น storage API ที่ตกค้างมาจากตอนพัฒนาใน Claude Artifacts ซึ่ง**ต้องยืนยัน**) ผลคือ
   - ธีม Dark mode ที่เลือกไว้**ไม่ถูกจำ** เปิดหน้าใหม่จะกลับเป็น Light ทุกครั้ง
   - Local mode (เก็บในเครื่อง + seed data) **เข้าถึงไม่ได้จริง** เพราะ `init()` บังคับให้กรอก token ก่อนเสมอ และ `API.url` ถูก hardcode ไว้แล้ว คอมเมนต์ที่บอกว่า "เว้นว่างทั้งคู่ = เก็บในเครื่อง" จึงไม่ตรงกับพฤติกรรมจริง
7. ถ้า token หมดอายุหรือถูกเปลี่ยน ระบบแค่แสดง pill สีแดง ไม่พาผู้ใช้กลับไปหน้า login
8. **การบันทึกทำงานไม่สม่ำเสมอ** ปุ่ม ← → (`move()`) รอ `saveNotes()` ให้เสร็จก่อนแล้วค่อย render (UI หน่วงตาม network) ขณะที่ drag-drop render ก่อนแล้วค่อยบันทึก ส่วน `copyNote()` ไม่ได้ `await` การบันทึก
9. **การเตือนเมื่อยังไม่ได้บันทึกยังไม่ครอบคลุม** ตอนนี้เตือนเฉพาะตอนสลับไปเปิดโน้ตอื่น ถ้ากด "ยกเลิก" modal จะปิดทันทีและข้อมูลที่พิมพ์ไว้หายโดยไม่มีการยืนยัน
10. **Mini Markdown parser รองรับ syntax จำกัด** ไม่รองรับ code block (```` ``` ````), table, รูปภาพ, nested list และ strikethrough ถ้า Gem ✨ สร้าง Markdown ที่มี table หรือ code block ข้อความจะแสดงเป็นย่อหน้าธรรมดา (รูปแบบ output ของ Gem **ต้องยืนยัน**)
11. **ข้อความใน UI ไม่ตรงกับ logic**
    - ปุ่ม "ยาก" มีป้ายว่า "+1 วัน" แต่ SM-2 ให้ +1 วันเฉพาะรอบแรก รอบถัดไปคำนวณเป็น `interval × EF`
    - กราฟใช้หัวข้อว่า "คาดการณ์ 14 วันข้างหน้า" แต่ที่แสดงจริงคือเกินกำหนด + วันนี้ + อีก 12 วัน
12. **ข้อมูลไม่สม่ำเสมอ**
    - `copyNote()` ตั้ง `lastQuality: 0`, `learnDate: ''` และ `sortOrder: 0` ส่วนที่อื่นใช้ `null` และลำดับถัดไป
    - field `learnDate` และ `lastUsed` ถูกบันทึกแต่ไม่แสดงใน UI ที่ไหนเลย
13. **ค่าบางตัวแทรกลง HTML โดยไม่ผ่าน `esc()`** ได้แก่ `n.id`, `n.priority`, `n.created`, `n.nextReview` และ ID ใน `links[]` ถ้ามีคนแก้ค่าเหล่านี้ใน Sheet โดยตรง อาจทำให้ UI พังหรือเกิด XSS ได้ (ความเสี่ยงต่ำเพราะใช้คนเดียว แต่ Sheet แก้ไขได้โดยตรง)
14. **โค้ดที่ไม่ได้ใช้หรือค่าที่ค้างอยู่**
    - ค่าคงที่ `SORTABLE` และ `PKM_BUILD` ไม่ถูกใช้งาน และ `PKM_BUILD = '2026-07-29'` ยังไม่อัปเดตตาม v1.3.1 (2026-08-01)
    - CSS `.child-techs .ct-id` อ้าง `var(--mono)` ซึ่งไม่ได้ประกาศไว้
15. ช่องค้นหาไม่ค้นใน ID และ field `technique`
16. **Accessibility / Touch**
    - การ์ดเป็น `<div onclick>` จึงใช้ keyboard ไม่ได้
    - ปุ่ม ← → บนการ์ดจะแสดงเมื่อ hover เท่านั้น
    - HTML5 Drag and Drop รองรับบนอุปกรณ์ touch ได้จำกัด (การใช้งานบนมือถือจริง**ต้องยืนยัน**)

### 🔵 ด้าน Repo และกระบวนการพัฒนา

17. โค้ดทั้งหมด (CSS ~370 บรรทัด + JS ~1,060 บรรทัด) อยู่ในไฟล์เดียว ถ้าแยกเป็น `style.css` / `app.js` หรือแยก module จะดูแลง่ายกว่า
18. Commit history มีแต่ข้อความ "Add files via upload" / "Delete index.html" (upload ผ่านหน้าเว็บ GitHub) ทำให้ดูไม่ออกว่าแต่ละ commit เปลี่ยนอะไร ส่วน changelog อยู่ใน HTML comment แนะนำให้ใช้ git CLI, เขียน commit message ที่มีความหมาย, แยก `CHANGELOG.md` และติด git tag ตามเวอร์ชัน
19. ยังไม่มี test ทั้งที่ `sm2()` และ date helper เป็น pure function ที่เขียน unit test ได้ง่าย (และช่วยจับ bug ข้อ 1 ได้) นอกจากนี้ยังไม่มี linter, `LICENSE` และ `.gitignore`
