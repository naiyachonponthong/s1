# Part 5 — LIFF บน GitHub Pages (คู่มือ deploy)

หน้า LIFF สำหรับ **ผู้ปกครอง** (ยื่นใบลา) และ **ครู** (เช็คชื่อ + ยื่นลา) เสิร์ฟแยกบน GitHub Pages เพราะ GAS HtmlService รัน LIFF ไม่ได้

## ไฟล์ในโฟลเดอร์ liff/
```
liff/
├── config.js          ← ⚠️ แก้ 2 ค่า: GAS_API_URL, LIFF_ID
├── index.html         ← หน้าผู้ปกครอง (Flex ปุ่ม "ยื่นใบลา" เปิดหน้านี้)
├── teacher.html       ← หน้าครู (login + เช็คชื่อ + ยื่นลา)
├── style.css
└── README-deploy.md
```

## ขั้นตอน deploy

### 1) อัปขึ้น GitHub Pages
1. สร้าง repo ใหม่ (เช่น `attendance-liff`) แล้วอัปไฟล์ในโฟลเดอร์ `liff/` ขึ้นไป (ให้ `index.html` อยู่ราก repo หรือใน `/docs`)
2. Settings > Pages > Source = branch `main` (โฟลเดอร์ราก หรือ `/docs`)
3. ได้ URL เช่น `https://USERNAME.github.io/attendance-liff/`

### 2) สร้าง LIFF app (LINE Developers)
1. เข้า LINE Developers > Channel (Messaging API เดิม) > แท็บ **LIFF** > Add
2. Endpoint URL = `https://USERNAME.github.io/attendance-liff/`  (หน้าผู้ปกครอง)
3. Size = Full, Scope = `profile`, ติ๊ก Bot link ถ้าต้องการ
4. คัดลอก **LIFF ID** ที่ได้

### 3) ตั้งค่า config.js
```js
const GAS_API_URL = "https://script.google.com/macros/s/XXXX/exec";  // URL /exec ของ Web App
const LIFF_ID     = "1234567890-abcdEFGH";                            // LIFF ID
```
แล้ว push ขึ้น GitHub อีกครั้ง

### 4) ผูกปุ่มใน Flex แจ้งเตือน
- ในเว็บ > ตั้งค่า > ช่อง **URL หน้า LIFF (GitHub Pages)** ใส่ `https://liff.line.me/<LIFF_ID>`
  (หรือ URL GitHub Pages ตรงๆ ก็ได้ แต่ใช้ `liff.line.me/<id>` จะเปิดในแอป LINE ลื่นกว่า)
- ปุ่ม "ยื่นใบลา" ในข้อความแจ้งเตือนขาด/สาย จะเปิดหน้าผู้ปกครองนี้

## การทำงาน
- **ผู้ปกครอง (index.html):** liff.init → ดึง userId → เรียก `parentInit` → แสดงบุตรหลาน + ประวัติลา → ยื่นใบลา (เลือกบุตรหลาน/ประเภท/วันที่/เหตุผล/แนบรูป)
  - ถ้ายังไม่เชื่อมบัญชี จะบอกให้ไปพิมพ์รหัสนักเรียนในแชทก่อน
- **ครู (teacher.html):** login ด้วย username/password เดิม → เช็คชื่อ (เลือกชั้น/วันที่ + ปุ่มมาครบทั้งห้อง) + ยื่นลาของตัวเอง + ดูสถานะใบลา

## การเชื่อมต่อ API (เลี่ยง CORS)
ทุกหน้าเรียก GAS ผ่าน:
```js
fetch(GAS_API_URL, { method:'POST', body: JSON.stringify({action, ...}) })
```
ไม่ตั้ง header `Content-Type` → เป็น text/plain → ไม่เกิด CORS preflight
ฝั่ง GAS `doPost` อ่าน `e.postData.contents` แล้ว route ตาม `action`

## หมายเหตุ
- ใบลาที่ยื่นจะสร้างคำขอ + สายอนุมัติตาม `APPROVAL_FLOWS` ทันที (สถานะ "รออนุมัติ")
- การ **อนุมัติ/ปฏิเสธ** เต็มรูปแบบ (เว็บ + LINE) + อัปเดต Attendance อัตโนมัติ อยู่ใน **Part 6**

## ถัดไป — Part 6
ระบบลา: อนุมัติหลายชั้น (เว็บ + LINE), อัปเดต Attendance เป็น "ลา" เมื่ออนุมัติ, โควต้าครู (toggle)
