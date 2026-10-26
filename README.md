# Paynie LIFF - Phase 1 (เลือกแผน & ตารางจ่ายรายเดือน)

เว็บแอปพลิเคชัน **LINE Front-end Framework (LIFF)** สำหรับ **Paynie (เพย์นี่)** ถอดแบบฟังก์ชันและดีไซน์จาก Prototype ใน Phase 1 ครบถ้วน:
- **หัวข้อ 3: เลือกแผน & ตารางจ่าย**: สไลเดอร์ปรับงบรายเดือน, การ์ดเปรียบเทียบ 3 รูปแบบ (Minimum Trap vs Avalanche vs Snowball) พร้อมคำนวณลดต้นลดดอกจริง, ตารางจัดลำดับจ่ายหนี้ประจำเดือน, ปุ่มยืนยันส่งผลเข้าแชท LINE และปุ่มปรึกษาคลินิกแก้หนี้ กทม. (ทีมชัชชาติ)
- **แท็บรายการหนี้**: สรุปยอดหนี้รวม, แสดงการ์ดหนี้แต่ละใบพร้อมปุ่มลบ, ฟอร์มเพิ่มหนี้ใหม่แบบ Interactive ที่คำนวณซิงค์กลับไปยังตารางจ่ายทันที
- **รองรับ Input ข้อมูลหนี้จริงจาก LINE OA / n8n Workflow** ผ่าน URL Query Parameter ทั้ง Base64 และ JSON

---

## 📱 โครงสร้างไฟล์
```text
.
├── index.html           # ไฟล์ Single-Page LIFF หลัก (HTML5 + Tailwind CDN + FontAwesome + Vanilla JS + LIFF SDK v2)
├── README.md            # เอกสารประกอบและคู่มือการเชื่อมต่อกับ n8n Workflow
├── paynie_line_oa_prototype_phase_1_phase_2.html # ไฟล์ Prototype ต้นแบบ
└── rich menu - res/     # รูปภาพ Rich Menu สำหรับ LINE OA
```

---

## 🚀 วิธีนำไปใช้งาน (Deployment)

1. **Deploy ขึ้น Web Server ที่รองรับ HTTPS**
   - สามารถนำไฟล์ `index.html` ไปเปิดใช้งานบน **GitHub Pages**, **Vercel**, **Netlify**, หรือ **Firebase Hosting** ได้ทันทีโดยไม่ต้อง Build
   - *ตัวอย่าง URL:* `https://praisepany.github.io/paynie-liff/`

2. **สร้าง LIFF App ใน LINE Developers Console**
   - เข้า [LINE Developers Console](https://developers.line.biz/)
   - เลือก Provider และ LINE Login Channel ของ Paynie
   - ไปที่แท็บ **LIFF** > กด **Add**
   - ตั้งค่า:
     - **Size**: `Full` หรือ `Tall`
     - **Endpoint URL**: ระบุ URL ของเว็บ (เช่น `https://praisepany.github.io/paynie-liff/`)
     - **Scopes**: ติ๊ก `profile`, `chat_message.write`
     - **Bot prompt**: `Normal` หรือ `None`
   - คัดลอก **LIFF ID** (เช่น `2001234567-abcdefgh`)

3. **กำหนด LIFF ID**
   - นำ LIFF ID ไปใส่ใน `CONFIG.DEFAULT_LIFF_ID` ในไฟล์ `index.html` หรือ
   - ส่งมาใน URL Parameter เช่น `https://liff.line.me/{LIFF_ID}`

---

## 🔄 การเชื่อมต่อกับ n8n Workflow

### 1. ฝั่ง n8n ส่งข้อมูลหนี้ที่ผู้ใช้กรอก เข้าสู่ LIFF
เมื่อ n8n ทำการรับข้อมูลหนี้จากผู้ใช้ผ่าน LINE Chat (เช่น จากข้อความพิมพ์ หรือ AI OCR จากสลิป/ใบแจ้งหนี้) n8n สามารถสร้างลิงก์ LIFF เพื่อส่งกลับไปให้ผู้ใช้กดเปิดได้ดังนี้:

#### โค้ดใน n8n Code Node (JavaScript):
```javascript
// ข้อมูลหนี้จริงที่ผู้ใช้กรอก/ตรวจพบผ่าน LINE
const debts = [
  { creditor: "บัตรเครดิต KBank", balance: 35000, apr: 16.0, minPct: 8.0, dueDay: 25 },
  { creditor: "บัตรกดเงินสด SCB Speedy Cash", balance: 22000, apr: 25.0, minPct: 5.0, dueDay: 28 },
  { creditor: "สินเชื่อส่วนบุคคล UOB", balance: 18000, apr: 22.0, minPct: 6.0, dueDay: 15 }
];

// แปลงเป็น Base64 ป้องกันปัญหาตัวอักษรภาษาไทยและอักขระพิเศษใน URL
const jsonString = JSON.stringify(debts);
const base64Data = Buffer.from(unescape(encodeURIComponent(jsonString))).toString('base64');

const LIFF_ID = "YOUR_LIFF_ID"; // แทนที่ด้วย LIFF ID ของคุณ
const liffUrl = `https://liff.line.me/${LIFF_ID}?data=${encodeURIComponent(base64Data)}`;

return [{ json: { liffUrl } }];
```

จากนั้นใน n8n ส่ง Flex Message หรือปุ่ม Button Template ที่มี Action เป็น `URI`: `{{ $json.liffUrl }}` ไปยังผู้ใช้ใน LINE

---

### 2. ฝั่ง LIFF ส่งผลลัพธ์การเลือกแผนกลับเข้า n8n Workflow
เมื่อผู้ใช้ปรับงบ สลับเลือกแผน (Avalanche หรือ Snowball) แล้วกดปุ่ม:
> **"เลือกแผนนี้ & ส่งตารางจ่ายหนี้เข้าแชท LINE"**

LIFF จะทำงาน 2 ทาง:
1. **ผ่าน LINE In-App Messaging (`liff.sendMessages`)**:
   - LIFF จะส่งข้อความสรุปผลในนามของผู้ใช้กลับเข้าห้องแชท LINE OA
   - LINE Messaging API จะยิง **Webhook Event (`message`)** ไปยัง **n8n LINE Webhook Trigger** โดยอัตโนมัติ
   - n8n สามารถดักจับข้อความที่ขึ้นต้นด้วย `🎯 ยืนยันเลือก:` เพื่อบันทึกลง Database หรือรัน Step ถัดไป
2. **Direct Webhook ไปยัง n8n (Optional)**:
   - สามารถระบุ Webhook URL ของ n8n ใน `CONFIG.N8N_WEBHOOK_URL` ใน `index.html` เพื่อให้ LIFF ยิง Structured JSON ตรงเข้า n8n ทันที

---

## 📊 ตัวอย่างฟอร์แมตข้อมูลหนี้ (Supported Schema)

แต่ละ Item ใน Array รองรับคีย์ต่อไปนี้:
```json
[
  {
    "creditor": "บัตรเครดิต KBank",
    "balance": 35000,
    "apr": 16.0,
    "minPct": 8.0,
    "dueDay": 25
  }
]
```
*(รองรับคีย์ `"name"` แทน `"creditor"` หรือ `"interestRate"` แทน `"apr"` โดยอัตโนมัติ)*
