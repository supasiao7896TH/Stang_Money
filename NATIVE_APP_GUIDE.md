# GiGi Money — Native App Development Guide
> เอกสารอ้างอิงฉบับสมบูรณ์สำหรับพัฒนา Native Android App สานต่อจาก Web App

---

## 1. ภาพรวมของ Web App

**GiGi Money** เป็น Personal Finance App แบบ Single-HTML-File สร้างด้วย:
- **React 18** (minified bundle, JSX transform via Babel standalone)
- **localStorage** เก็บข้อมูลทั้งหมดในคีย์เดียว
- **Google Gemini API** วิเคราะห์รายจ่ายและอ่านใบเสร็จ
- ภาษาแสดงผล: **ภาษาไทย** + ปีพุทธศักราช (ค.ศ. + 543)

---

## 2. โครงสร้างข้อมูล (Data Models)

### 2.1 รูปแบบการเก็บข้อมูล

```
localStorage key: "stang_money_v1"
value: JSON string ของ object ด้านล่าง
```

### 2.2 Root Data Object

```json
{
  "transactions": [ ...Transaction[] ],
  "settings": { ...Settings },
  "goals": [ ...Goal[] ]
}
```

---

### 2.3 Transaction (รายการธุรกรรม)

```typescript
interface Transaction {
  id: string;          // UUID หรือ timestamp-based unique ID
  date: string;        // ISO 8601 → "2026-05-30T14:30:00.000Z"
  amount: number;      // จำนวนเงิน (บาท) เช่น 150.50
  type: "expense" | "income";  // รายจ่าย หรือ รายรับ
  categoryId: string;  // ดูตาราง Category IDs ด้านล่าง
  note: string;        // หมายเหตุ / ชื่อร้าน (string ว่างได้)
}
```

**ตัวอย่าง:**
```json
{
  "id": "1748563234567",
  "date": "2026-05-30T07:15:00.000Z",
  "amount": 85,
  "type": "expense",
  "categoryId": "e_breakfast",
  "note": "ข้าวต้มร้านแม่สุด"
}
```

---

### 2.4 Settings (การตั้งค่า)

```typescript
interface Settings {
  geminiKey: string;         // Google AI Studio API Key
  geminiModel: string;       // model ID เช่น "gemini-2.5-flash"
  monthStartDay: number;     // วันเริ่มต้นรอบบิล (1–28), default = 1
}
```

---

### 2.5 Goal (เป้าหมายการออม)

```typescript
interface Goal {
  id: string;
  name: string;      // ชื่อเป้าหมาย เช่น "เที่ยวญี่ปุ่น"
  target: number;    // เงินที่ต้องการ (บาท)
  saved: number;     // เงินที่สะสมได้แล้ว (บาท)
  icon: string;      // emoji
  deadline?: string; // ISO date string (optional)
}
```

---

## 3. Category IDs ทั้งหมด

### 3.1 รายจ่าย (Expense) — prefix `e_`

| ID | กลุ่ม | ชื่อ |
|----|-------|------|
| `e_breakfast` | อาหาร | อาหารเช้า |
| `e_lunch` | อาหาร | อาหารกลางวัน |
| `e_dinner` | อาหาร | อาหารเย็น |
| `e_snack` | อาหาร | ขนม / ของว่าง |
| `e_coffee` | อาหาร | กาแฟ / เครื่องดื่ม |
| `e_bobatea` | อาหาร | ชานมไข่มุก |
| `e_restaurant` | อาหาร | ร้านอาหาร / ภัตตาคาร |
| `e_delivery` | อาหาร | Food Delivery |
| `e_ingredients` | อาหาร | วัตถุดิบทำอาหาร |
| `e_grocery` | อาหาร | ของชำ / Supermarket |
| `e_alcohol` | อาหาร | เครื่องดื่มแอลกอฮอล์ |
| `e_fruit` | อาหาร | ผลไม้ |
| `e_ev_charge` | รถ EV | ชาร์จรถ EV |
| `e_ev_home_charge` | รถ EV | ค่าไฟชาร์จที่บ้าน |
| `e_ev_service` | รถ EV | บำรุงรักษารถ EV |
| `e_ev_membership` | รถ EV | สมาชิกสถานีชาร์จ |
| `e_fuel` | เดินทาง | น้ำมันรถ |
| `e_tollway` | เดินทาง | ค่าทางด่วน |
| `e_parking` | เดินทาง | ค่าจอดรถ |
| `e_bts` | เดินทาง | BTS / MRT |
| `e_bus` | เดินทาง | รถเมล์ / ตุ๊กตุ๊ก |
| `e_taxi` | เดินทาง | แท็กซี่ / Grab |
| `e_flight` | เดินทาง | ตั๋วเครื่องบิน |
| `e_train` | เดินทาง | ตั๋วรถไฟ / รถทัวร์ |
| `e_motorcycle` | เดินทาง | วินมอเตอร์ไซค์ |
| `e_carservice` | เดินทาง | บำรุงรักษารถ |
| `e_carinsurance` | เดินทาง | ประกันรถ |
| `e_carregister` | เดินทาง | ต่อทะเบียน / พ.ร.บ. |
| `e_car_rental` | เดินทาง | เช่ารถ |
| `e_boat` | เดินทาง | เรือ / เฟอร์รี่ |
| `e_travel_insurance` | เดินทาง | ประกันการเดินทาง |
| `e_rent` | ที่อยู่อาศัย | ค่าเช่าบ้าน / หอ |
| `e_mortgage` | ที่อยู่อาศัย | ผ่อนบ้าน / คอนโด |
| `e_water` | ที่อยู่อาศัย | ค่าน้ำ |
| `e_electricity` | ที่อยู่อาศัย | ค่าไฟ |
| `e_internet` | ที่อยู่อาศัย | อินเทอร์เน็ต |
| `e_phone` | ที่อยู่อาศัย | ค่าโทรศัพท์ |
| `e_gas` | ที่อยู่อาศัย | ค่าแก๊สหุงต้ม |
| `e_common` | ที่อยู่อาศัย | ค่าส่วนกลาง |
| `e_repair` | ที่อยู่อาศัย | ซ่อมบ้าน / ติดตั้ง |
| `e_furniture` | ที่อยู่อาศัย | เฟอร์นิเจอร์ |
| `e_household` | ที่อยู่อาศัย | ของใช้ในบ้าน |
| `e_cleaning` | ที่อยู่อาศัย | น้ำยาทำความสะอาด |
| `e_drinking_water` | ที่อยู่อาศัย | ค่าน้ำดื่ม (ถังน้ำ) |
| `e_housekeeper` | ที่อยู่อาศัย | ค่าแม่บ้าน / คนสวน |
| `e_cable_tv` | ที่อยู่อาศัย | ทีวีดาวเทียม / เคเบิล |
| `e_clothes` | ช้อปปิ้ง | เสื้อผ้า |
| `e_shoes` | ช้อปปิ้ง | รองเท้า |
| `e_bag` | ช้อปปิ้ง | กระเป๋า |
| `e_accessory` | ช้อปปิ้ง | เครื่องประดับ |
| `e_cosmetic` | ช้อปปิ้ง | เครื่องสำอาง |
| `e_perfume` | ช้อปปิ้ง | น้ำหอม / Skincare |
| `e_personal` | ช้อปปิ้ง | ของใช้ส่วนตัว |
| `e_gift` | ช้อปปิ้ง | ของขวัญให้คนอื่น |
| `e_electronics` | ช้อปปิ้ง | เครื่องใช้ไฟฟ้า |
| `e_gadget` | ช้อปปิ้ง | มือถือ / Gadget |
| `e_online` | ช้อปปิ้ง | Shopee / Lazada |
| `e_hospital` | สุขภาพ | ค่ารักษาพยาบาล |
| `e_medicine` | สุขภาพ | ยา / วิตามิน |
| `e_gym` | สุขภาพ | ฟิตเนส / ออกกำลังกาย |
| `e_checkup` | สุขภาพ | ตรวจสุขภาพ |
| `e_dental` | สุขภาพ | ทำฟัน |
| `e_health_insurance` | สุขภาพ | ประกันสุขภาพ |
| `e_beauty_clinic` | สุขภาพ | คลินิกความงาม |
| `e_haircut` | สุขภาพ | ทำผม / ตัดผม |
| `e_massage` | สุขภาพ | นวด / สปา |
| `e_eyewear` | สุขภาพ | แว่นตา / คอนแทคเลนส์ |
| `e_movie` | บันเทิง | ดูหนัง / โรงหนัง |
| `e_concert` | บันเทิง | คอนเสิร์ต / อีเวนต์ |
| `e_game` | บันเทิง | เกม / ของในเกม |
| `e_netflix` | บันเทิง | Netflix / Streaming |
| `e_music` | บันเทิง | Spotify / Music |
| `e_ebook` | บันเทิง | หนังสือ / E-book |
| `e_hobby` | บันเทิง | งานอดิเรก |
| `e_sport` | บันเทิง | กีฬา / อุปกรณ์กีฬา |
| `e_travel` | บันเทิง | ท่องเที่ยว |
| `e_hotel` | บันเทิง | โรงแรม / ที่พัก |
| `e_party` | บันเทิง | ปาร์ตี้ / สังสรรค์ |
| `e_tuition` | การศึกษา | ค่าเทอม |
| `e_book` | การศึกษา | หนังสือเรียน |
| `e_course` | การศึกษา | คอร์สเรียนออนไลน์ |
| `e_supplies` | การศึกษา | อุปกรณ์การเรียน |
| `e_tutor` | การศึกษา | ค่าติว / ครูสอนพิเศษ |
| `e_exam` | การศึกษา | ค่าสอบ / สมัครสอบ |
| `e_parents` | ครอบครัว | เงินให้พ่อแม่ |
| `e_kid_school` | ครอบครัว | ค่าเทอมลูก |
| `e_kid_stuff` | ครอบครัว | ของใช้ลูก |
| `e_family_allowance` | ครอบครัว | ค่าใช้จ่ายครอบครัว (ลูก/ภรรยา) |
| `e_tax` | การเงิน | ภาษี |
| `e_life_insurance` | การเงิน | ประกันชีวิต |
| `e_transfer` | การเงิน | ค่าโอนเงิน |
| `e_interest` | การเงิน | ดอกเบี้ยจ่าย |
| `e_invest_loss` | การเงิน | ขาดทุนลงทุน |
| `e_accident_insurance` | การเงิน | ประกันอุบัติเหตุ (PA) |
| `e_social_security` | ออมและลงทุน | ประกันสังคม (ม.33) |
| `e_pvd` | ออมและลงทุน | PVD / กองทุนสำรองเลี้ยงชีพ |
| `e_income_tax` | ออมและลงทุน | ภาษีเงินได้บุคคลธรรมดา |
| `e_ssf` | ออมและลงทุน | กองทุน SSF |
| `e_rmf` | ออมและลงทุน | กองทุน RMF |
| `e_gpf` | ออมและลงทุน | กบข. / กองทุนบำเหน็จบำนาญ |
| `e_cooperative_savings` | ออมและลงทุน | สหกรณ์ออมทรัพย์ |
| `e_buy_gold` | ออมและลงทุน | ซื้อทองคำ |
| `e_bonds` | ออมและลงทุน | หุ้นกู้ / ตราสารหนี้ |
| `e_savings_account` | ออมและลงทุน | ออมทรัพย์ธรรมดา |
| `e_fixed_deposit` | ออมและลงทุน | ฝากประจำ |
| `e_buy_stock` | ออมและลงทุน | ซื้อหุ้น / ETF ไทย |
| `e_mutual_fund` | ออมและลงทุน | ซื้อกองทุนรวม |
| `e_buy_crypto` | ออมและลงทุน | ซื้อ Bitcoin / คริปโต |
| `e_reits` | ออมและลงทุน | REITs / กองทุนอสังหาฯ |
| `e_savings_insurance` | ออมและลงทุน | ประกันสะสมทรัพย์ |
| `e_goch` | ออมและลงทุน | กอช. (กองทุนการออมแห่งชาติ) |
| `e_donation` | อื่นๆ | บริจาค / ทำบุญ |
| `e_ceremony` | อื่นๆ | งานบวช / งานแต่ง |
| `e_fine` | อื่นๆ | ค่าปรับ |
| `e_laundry` | อื่นๆ | ค่าซักรีด |
| `e_tip` | อื่นๆ | ค่าทิป |
| `e_charity` | อื่นๆ | ช่วยเหลือผู้อื่น |
| `e_misc` | อื่นๆ | เบ็ดเตล็ด |

---

### 3.2 รายรับ (Income) — prefix `i_`

| ID | กลุ่ม | ชื่อ |
|----|-------|------|
| `i_salary` | รายได้หลัก | เงินเดือน |
| `i_bonus` | รายได้หลัก | โบนัส |
| `i_ot` | รายได้หลัก | OT / ค่าล่วงเวลา |
| `i_commission` | รายได้หลัก | คอมมิชชั่น |
| `i_perdiem` | รายได้หลัก | เบี้ยเลี้ยง |
| `i_pension` | รายได้หลัก | เงินบำนาญ / บำเหน็จ |
| `i_freelance` | รายได้เสริม | งาน Freelance |
| `i_parttime` | รายได้เสริม | งาน Part-time |
| `i_business` | รายได้เสริม | รายได้จากธุรกิจ |
| `i_online_sell` | รายได้เสริม | ขายของออนไลน์ |
| `i_secondhand` | รายได้เสริม | ขายของมือสอง |
| `i_service` | รายได้เสริม | รับจ้าง / บริการ |
| `i_youtube` | รายได้เสริม | YouTube / Content |
| `i_affiliate` | รายได้เสริม | Affiliate / Referral |
| `i_tip_in` | รายได้เสริม | ทิปที่ได้รับ |
| `i_sponsorship` | รายได้เสริม | เงินสนับสนุน / สปอนเซอร์ |
| `i_rent` | การลงทุน | ค่าเช่า / อสังหา |
| `i_dividend` | การลงทุน | เงินปันผล |
| `i_interest` | การลงทุน | ดอกเบี้ยเงินฝาก |
| `i_stock` | การลงทุน | กำไรหุ้น |
| `i_crypto` | การลงทุน | กำไรคริปโต |
| `i_fund` | การลงทุน | กำไรกองทุน |
| `i_gold` | การลงทุน | กำไรขายทอง |
| `i_fd_interest` | การลงทุน | ดอกเบี้ยฝากประจำ |
| `i_reits_return` | การลงทุน | ผลตอบแทน REITs |
| `i_insurance_maturity` | การลงทุน | เงินครบกรมธรรม์ |
| `i_staking` | การลงทุน | กำไร Staking / Yield |
| `i_asset_sale` | การลงทุน | ขายทรัพย์สิน (บ้าน/รถ/ที่ดิน) |
| `i_refund_tax` | เงินคืน | เงินคืนภาษี |
| `i_cashback` | เงินคืน | Cashback / เงินคืนบัตร |
| `i_refund` | เงินคืน | คืนเงิน / Refund |
| `i_insurance_claim` | เงินคืน | เคลมประกัน |
| `i_social_security_benefit` | สวัสดิการ | สิทธิ์ประกันสังคม |
| `i_pvd_return` | สวัสดิการ | เงินคืน PVD |
| `i_welfare` | สวัสดิการ | สวัสดิการจากนายจ้าง |
| `i_government_benefit` | สวัสดิการ | เงินอุดหนุนรัฐ / บัตรสวัสดิการ |
| `i_gift` | อื่นๆ | ของขวัญ / เงินขวัญถุง |
| `i_family` | อื่นๆ | เงินจากพ่อแม่/ญาติ |
| `i_scholarship` | อื่นๆ | ทุนการศึกษา |
| `i_prize` | อื่นๆ | รางวัล / Prize |
| `i_lottery` | อื่นๆ | ถูกหวย / สลาก |
| `i_other_in` | อื่นๆ | รายรับเบ็ดเตล็ด |

---

### 3.3 Category Groups พิเศษ (สำหรับคำนวณภาษี)

```kotlin
// กลุ่มลดหย่อนภาษี
val TAX_SAVING_IDS = listOf("e_ssf","e_rmf","e_gpf","e_pvd","e_goch","e_savings_insurance")

// ประกันชีวิต (ลดหย่อนได้)
val LIFE_INSURANCE_IDS = listOf("e_life_insurance")

// ประกันสุขภาพ (ลดหย่อนได้)
val HEALTH_INSURANCE_IDS = listOf("e_health_insurance")
```

---

## 4. Business Logic สำคัญ

### 4.1 Billing Cycle (รอบบิล)

Web App รองรับการตั้งวันเริ่มต้นรอบบิลที่ไม่ใช่วันที่ 1

```kotlin
data class BillingRange(
    val start: Date,
    val end: Date,
    val label: String?  // null ถ้า monthStartDay == 1
)

fun calcBillingRange(year: Int, month: Int, startDay: Int): BillingRange {
    // month: 0-based (0=มกราคม, 11=ธันวาคม) เหมือน Java Calendar
    val safeDay = startDay.coerceIn(1, 28)
    
    if (safeDay == 1) {
        // รอบปกติ: 1 ม.ค. - 31 ม.ค.
        val start = Calendar.getInstance().apply {
            set(year, month, 1, 0, 0, 0)
            set(Calendar.MILLISECOND, 0)
        }.time
        val end = Calendar.getInstance().apply {
            set(year, month + 1, 0, 23, 59, 59)  // วันสุดท้ายของเดือน
            set(Calendar.MILLISECOND, 999)
        }.time
        return BillingRange(start, end, null)
    }
    
    // รอบ custom เช่น 25 พ.ค. - 24 มิ.ย.
    val start = Calendar.getInstance().apply {
        set(year, month, safeDay, 0, 0, 0)
        set(Calendar.MILLISECOND, 0)
    }.time
    
    var endYear = year
    var endMonth = month + 1
    if (endMonth > 11) { endMonth = 0; endYear++ }
    
    val end = Calendar.getInstance().apply {
        set(endYear, endMonth, safeDay - 1, 23, 59, 59)
        set(Calendar.MILLISECOND, 999)
    }.time
    
    return BillingRange(start, end, label = buildLabel(month, endMonth, safeDay, endYear))
}
```

### 4.2 การกรอง Transaction ตามเดือน/รอบบิล

```kotlin
fun filterTransactionsByBillingCycle(
    transactions: List<Transaction>,
    year: Int,
    month: Int,  // 0-based
    monthStartDay: Int
): List<Transaction> {
    val range = calcBillingRange(year, month, monthStartDay)
    return transactions.filter { tx ->
        val date = tx.date.toDate()  // parse ISO 8601
        date >= range.start && date <= range.end
    }
}
```

### 4.3 สรุปรายเดือน 6 เดือนย้อนหลัง

```kotlin
data class MonthlySummary(
    val year: Int,
    val month: Int,   // 0-based
    val income: Double,
    val expense: Double,
    val net: Double   // income - expense
)

fun getLast6MonthsSummary(
    transactions: List<Transaction>,
    monthStartDay: Int
): List<MonthlySummary> {
    val result = mutableListOf<MonthlySummary>()
    val cal = Calendar.getInstance()
    // ถอยหลัง 5 เดือน (ได้ 6 เดือนรวมเดือนปัจจุบัน)
    cal.add(Calendar.MONTH, -5)
    
    repeat(6) {
        val year = cal.get(Calendar.YEAR)
        val month = cal.get(Calendar.MONTH)
        val range = calcBillingRange(year, month, monthStartDay)
        
        val monthTx = transactions.filter { tx ->
            val d = tx.date.toDate()
            d >= range.start && d <= range.end
        }
        val income = monthTx.filter { it.type == "income" }.sumOf { it.amount }
        val expense = monthTx.filter { it.type == "expense" }.sumOf { it.amount }
        result.add(MonthlySummary(year, month, income, expense, income - expense))
        cal.add(Calendar.MONTH, 1)
    }
    return result
}
```

---

## 5. Gemini API Integration

### 5.1 Models ที่รองรับ

| Model ID | ความสามารถ | ความเร็ว |
|----------|-----------|---------|
| `gemini-2.5-flash` | Text + Vision | เร็ว (แนะนำ default) |
| `gemini-2.5-pro` | Text + Vision | ช้าแต่แม่นยำกว่า |
| `gemini-2.0-flash` | Text + Vision | เร็วมาก |
| `gemini-1.5-flash` | Text + Vision | รุ่นเก่า |
| `gemini-1.5-pro` | Text + Vision | รุ่นเก่า |

### 5.2 Endpoint

```
https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent?key={apiKey}
```

### 5.3 OCR ใบเสร็จ / Slip (Vision API)

```kotlin
// Android: แปลง Bitmap เป็น Base64
fun bitmapToBase64(bitmap: Bitmap): String {
    val stream = ByteArrayOutputStream()
    bitmap.compress(Bitmap.CompressFormat.JPEG, 85, stream)
    return Base64.encodeToString(stream.toByteArray(), Base64.NO_WRAP)
}

// สร้าง Prompt
fun buildReceiptPrompt(categoryIds: List<String>): String {
    val expenseIds = categoryIds.filter { it.startsWith("e_") }.joinToString(",")
    val incomeIds = categoryIds.filter { it.startsWith("i_") }.joinToString(",")
    return """
วิเคราะห์ใบเสร็จหรือสลิปในภาพนี้ แล้วคืนค่า JSON เท่านั้น ไม่มีข้อความอื่น
format: {"amount":number,"type":"expense"|"income","note":"string","categoryId":"string","date":"YYYY-MM-DD"}
- amount: จำนวนเงิน (ตัวเลขบาท)
- type: expense หรือ income
- note: ชื่อร้าน/รายการหลัก (ไม่เกิน 50 ตัวอักษร)
- categoryId: เลือกจากรายการนี้เท่านั้น
  expense: $expenseIds
  income: $incomeIds
- date: วันที่บนสลิป รูปแบบ YYYY-MM-DD (ถ้าไม่มีให้ใช้วันนี้)
ถ้าอ่านไม่ได้ให้คืน {"error":"อ่านไม่ได้"}
""".trimIndent()
}

// Request Body
data class GeminiRequest(
    val contents: List<Content>,
    val generationConfig: GenerationConfig
)

// เรียก API
suspend fun scanReceipt(imageBase64: String, mimeType: String, apiKey: String, model: String): ReceiptResult {
    val requestBody = JSONObject().apply {
        put("contents", JSONArray().put(JSONObject().apply {
            put("parts", JSONArray().apply {
                put(JSONObject().put("text", buildReceiptPrompt(ALL_CATEGORY_IDS)))
                put(JSONObject().apply {
                    put("inline_data", JSONObject().apply {
                        put("mime_type", mimeType)   // "image/jpeg"
                        put("data", imageBase64)
                    })
                })
            })
        }))
        put("generationConfig", JSONObject().apply {
            put("temperature", 0.1)
            put("maxOutputTokens", 300)
            put("responseMimeType", "application/json")
        })
    }
    
    val url = "https://generativelanguage.googleapis.com/v1beta/models/$model:generateContent?key=$apiKey"
    // ... HTTP POST, parse response
}
```

### 5.4 AI วิเคราะห์การเงิน (Text API)

```kotlin
fun buildFinancialAnalysisPrompt(
    summaries: List<MonthlySummary>,
    monthStartDay: Int
): String {
    val bilCycleNote = if (monthStartDay > 1) "(รอบบิลเริ่ม $monthStartDay ของทุกเดือน)" else ""
    val data = summaries.joinToString("\n") { s ->
        "เดือน ${s.month+1}/${s.year}: รายรับ ${s.income} รายจ่าย ${s.expense} คงเหลือ ${s.net}"
    }
    return """
คุณคือที่ปรึกษาการเงินส่วนตัว วิเคราะห์ข้อมูล 6 เดือนนี้ $bilCycleNote
$data
ให้คำแนะนำ 3-5 ข้อ เป็นภาษาไทย กระชับ เข้าใจง่าย
""".trimIndent()
}
```

---

## 6. หน้าจอของ App (Screens)

### 6.1 โครงสร้างหน้าจอ

```
App
├── HomeScreen        (tab: "home")  — สรุปเดือน + รายการล่าสุด
├── StatsScreen       (tab: "stats") — กราฟ 6 เดือน + รายจ่ายแบ่งตามหมวด
├── AIScreen          (tab: "ai")    — Gemini วิเคราะห์การเงิน
├── GoalScreen        (tab: "goal")  — เป้าหมายการออม
├── SettingsScreen    (tab: "settings") — ตั้งค่า API Key, รอบบิล, Model
└── TransactionForm   (modal/bottom sheet) — เพิ่ม/แก้ไขรายการ
        └── OCR Button (📷) — สแกนใบเสร็จ (เฉพาะโหมดสร้างใหม่)
```

### 6.2 HomeScreen

แสดงข้อมูลประจำเดือน:
- **Header**: ปุ่ม `<` `>` เปลี่ยนเดือน + ชื่อเดือน (หรือช่วงรอบบิล เช่น "25 พ.ค. - 24 มิ.ย. 69")
- **Summary Cards**: รายรับรวม / รายจ่ายรวม / คงเหลือ
- **Transaction List**: รายการย้อนหลังในรอบเดือนนั้น จัดกลุ่มตามวัน
- **FAB**: ปุ่ม `+` เปิด TransactionForm

### 6.3 TransactionForm

Fields:
| Field | ประเภท | หมายเหตุ |
|-------|--------|---------|
| type | toggle | "expense" หรือ "income" |
| amount | number | ทศนิยมได้ |
| categoryId | picker | dropdown/grid แบ่งตาม group |
| note | text | optional |
| date | date picker | default = วันนี้ |

ปุ่มพิเศษ (เฉพาะสร้างใหม่): 📷 เปิด gallery เพื่อ OCR

---

## 7. การ Migrate ข้อมูลไป Native App

### 7.1 Export/Import JSON

Web App เก็บข้อมูลใน localStorage เดียว → Export เป็น JSON ได้ง่าย

```kotlin
// Android: อ่านไฟล์ JSON ที่ user export มาจาก Web App
fun importFromWebApp(jsonString: String): AppData {
    val root = JSONObject(jsonString)
    val transactions = root.getJSONArray("transactions").let { arr ->
        (0 until arr.length()).map { i ->
            val obj = arr.getJSONObject(i)
            Transaction(
                id = obj.getString("id"),
                date = obj.getString("date"),
                amount = obj.getDouble("amount"),
                type = obj.getString("type"),
                categoryId = obj.getString("categoryId"),
                note = obj.optString("note", "")
            )
        }
    }
    val settings = root.optJSONObject("settings")?.let { s ->
        Settings(
            geminiKey = s.optString("geminiKey", ""),
            geminiModel = s.optString("geminiModel", "gemini-2.5-flash"),
            monthStartDay = s.optInt("monthStartDay", 1)
        )
    } ?: Settings()
    // ... parse goals
    return AppData(transactions, settings)
}
```

### 7.2 รูปแบบข้อมูลที่ Export จาก Web App

เมื่อ user กด Export ใน Web App จะได้ไฟล์ JSON ที่มีโครงสร้างเดียวกับ Section 2.2

---

## 8. Native Android: ฟีเจอร์ที่ Web App ทำไม่ได้

### 8.1 Auto Slip Detection (เหมือน เหมียวจด)

วิธีที่ เหมียวจด ของ KBank ใช้คือ **NotificationListenerService** ซึ่งดักอ่าน notification ของแอปธนาคาร

```kotlin
// AndroidManifest.xml
<service
    android:name=".NotificationListener"
    android:permission="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE"
    android:exported="true">
    <intent-filter>
        <action android:name="android.service.notification.NotificationListenerService"/>
    </intent-filter>
</service>
```

```kotlin
class NotificationListener : NotificationListenerService() {
    
    // Package names ของแอปธนาคารไทยที่ต้องการดักจับ
    val BANK_PACKAGES = setOf(
        "com.kasikorn.retail.mbanking.wap",  // KBank
        "th.co.scb.mybankapp",               // SCB
        "com.bbl.mobilebanking",             // BBL
        "com.kiatnakin.mobilebank",          // KKP
        "th.co.ktb.smart",                  // KTB
        "com.tmb.oneapp",                    // TTB
        "th.co.bay.myayudhya.connect",      // Krungsri
        "com.ttbbank.oneapp"                 // TTB
    )
    
    override fun onNotificationPosted(sbn: StatusBarNotification) {
        val pkg = sbn.packageName
        if (pkg !in BANK_PACKAGES) return
        
        val extras = sbn.notification.extras
        val title = extras.getString(Notification.EXTRA_TITLE) ?: ""
        val text = extras.getString(Notification.EXTRA_TEXT) ?: ""
        val bigText = extras.getString(Notification.EXTRA_BIG_TEXT) ?: ""
        
        val fullText = "$title\n$text\n$bigText"
        
        // ส่งไป Gemini เพื่อ parse
        parseTransactionFromNotification(fullText)
    }
    
    fun parseTransactionFromNotification(text: String) {
        // ส่ง text ไป Gemini API (text-only, ไม่ต้องใช้ Vision)
        // prompt: "แยกข้อมูลการโอนเงิน: จำนวน, ประเภท, ชื่อผู้รับ/ร้านค้า จาก: $text"
        // ได้ JSON → สร้าง Transaction object → แสดง confirmation dialog → บันทึก
    }
}
```

```kotlin
// ขอสิทธิ์ NotificationListenerService
fun requestNotificationAccess(activity: Activity) {
    val intent = Intent(Settings.ACTION_NOTIFICATION_LISTENER_SETTINGS)
    activity.startActivity(intent)
}

// ตรวจสอบว่าได้รับสิทธิ์หรือยัง
fun isNotificationAccessGranted(context: Context): Boolean {
    val flat = Settings.Secure.getString(
        context.contentResolver,
        "enabled_notification_listeners"
    )
    return flat?.contains(context.packageName) == true
}
```

### 8.2 Auto Slip from Gallery (MediaStore Observer)

```kotlin
// ดักจับภาพใหม่ที่บันทึกลง gallery อัตโนมัติ
class SlipImageObserver(
    handler: Handler,
    private val contentResolver: ContentResolver,
    private val onNewImage: (Uri) -> Unit
) : ContentObserver(handler) {

    override fun onChange(selfChange: Boolean, uri: Uri?) {
        uri ?: return
        // กรอง: ต้องเป็นภาพในโฟลเดอร์ Pictures หรือ DCIM
        val cursor = contentResolver.query(
            uri,
            arrayOf(MediaStore.Images.Media.DATE_ADDED, MediaStore.Images.Media.BUCKET_DISPLAY_NAME),
            null, null, null
        )
        cursor?.use {
            if (it.moveToFirst()) {
                val bucket = it.getString(it.getColumnIndexOrThrow(MediaStore.Images.Media.BUCKET_DISPLAY_NAME))
                // ตรวจว่าเป็น slip (ชื่อโฟลเดอร์ของธนาคาร)
                if (isLikelyBankSlip(bucket)) {
                    onNewImage(uri)
                }
            }
        }
    }
}

// Register Observer
fun startWatchingGallery(context: Context) {
    val observer = SlipImageObserver(Handler(Looper.getMainLooper()), context.contentResolver) { uri ->
        // uri ของภาพใหม่ → ส่ง OCR → auto-fill form
        processSlipImage(uri)
    }
    context.contentResolver.registerContentObserver(
        MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
        true,
        observer
    )
}
```

### 8.3 Foreground Service สำหรับ Background Monitoring

```kotlin
// ให้ app ทำงานเบื้องหลังได้แม้ user ปิดหน้าจอ
class SlipMonitorService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // สร้าง notification channel + foreground notification
        startForeground(NOTIF_ID, buildPersistentNotification())
        return START_STICKY
    }
}
```

---

## 9. โครงสร้าง Android Project ที่แนะนำ

```
app/
├── data/
│   ├── model/
│   │   ├── Transaction.kt
│   │   ├── Category.kt
│   │   ├── Settings.kt
│   │   └── Goal.kt
│   ├── repository/
│   │   └── TransactionRepository.kt
│   └── local/
│       ├── AppDatabase.kt          (Room)
│       ├── TransactionDao.kt
│       └── SettingsDao.kt
├── service/
│   ├── NotificationListener.kt     (ดัก notification ธนาคาร)
│   └── SlipMonitorService.kt       (Foreground service)
├── api/
│   └── GeminiApiClient.kt          (Gemini REST)
├── ui/
│   ├── home/
│   │   └── HomeFragment.kt
│   ├── stats/
│   │   └── StatsFragment.kt
│   ├── ai/
│   │   └── AIFragment.kt
│   ├── transaction/
│   │   ├── TransactionFormFragment.kt
│   │   └── OcrViewModel.kt
│   ├── goals/
│   │   └── GoalsFragment.kt
│   └── settings/
│       └── SettingsFragment.kt
└── util/
    ├── BillingCycleUtil.kt         (calcBillingRange logic)
    ├── DateUtil.kt                 (Thai Buddhist Era format)
    └── CategoryUtil.kt             (Category lookup)
```

### 9.1 Room Database Schema

```kotlin
@Entity(tableName = "transactions")
data class TransactionEntity(
    @PrimaryKey val id: String,
    val date: String,           // ISO 8601
    val amount: Double,
    val type: String,           // "expense" | "income"
    val categoryId: String,
    val note: String
)

@Entity(tableName = "goals")
data class GoalEntity(
    @PrimaryKey val id: String,
    val name: String,
    val target: Double,
    val saved: Double,
    val icon: String,
    val deadline: String?
)
```

---

## 10. Thai Date / Buddhist Era Utilities

```kotlin
object ThaiDateUtil {
    val SHORT_MONTHS_TH = listOf(
        "ม.ค.","ก.พ.","มี.ค.","เม.ย.","พ.ค.","มิ.ย.",
        "ก.ค.","ส.ค.","ก.ย.","ต.ค.","พ.ย.","ธ.ค."
    )
    val FULL_MONTHS_TH = listOf(
        "มกราคม","กุมภาพันธ์","มีนาคม","เมษายน","พฤษภาคม","มิถุนายน",
        "กรกฎาคม","สิงหาคม","กันยายน","ตุลาคม","พฤศจิกายน","ธันวาคม"
    )
    
    // ค.ศ. → พ.ศ.
    fun toBuddhistYear(year: Int) = year + 543
    
    // แสดง "25 พ.ค. 69"
    fun formatShort(date: Date): String {
        val cal = Calendar.getInstance().apply { time = date }
        val day = cal.get(Calendar.DAY_OF_MONTH)
        val month = cal.get(Calendar.MONTH)
        val year = toBuddhistYear(cal.get(Calendar.YEAR)) % 100
        return "$day ${SHORT_MONTHS_TH[month]} ${year.toString().padStart(2,'0')}"
    }
    
    // แสดง "พฤษภาคม 2569"
    fun formatMonthYear(year: Int, month: Int): String {
        return "${FULL_MONTHS_TH[month]} ${toBuddhistYear(year)}"
    }
    
    // แสดงช่วงรอบบิล "25 พ.ค. - 24 มิ.ย. 69"
    fun formatBillingRange(startDay: Int, startMonth: Int, endDay: Int, endMonth: Int, endYear: Int): String {
        val shortYear = (toBuddhistYear(endYear) % 100).toString().padStart(2, '0')
        return "$startDay ${SHORT_MONTHS_TH[startMonth]} - $endDay ${SHORT_MONTHS_TH[endMonth]} $shortYear"
    }
}
```

---

## 11. สิทธิ์ที่ต้องขอ (AndroidManifest Permissions)

```xml
<!-- จำเป็น -->
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>  <!-- Android 13+ -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>  <!-- Android 12- -->

<!-- สำหรับ auto-slip detection -->
<uses-permission android:name="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>  <!-- Android 13+ -->

<!-- optional: biometric lock -->
<uses-permission android:name="android.permission.USE_BIOMETRIC"/>
```

---

## 12. Stack เทคโนโลยีที่แนะนำสำหรับ Native App

| ส่วน | แนะนำ | เหตุผล |
|------|-------|--------|
| Language | **Kotlin** | Modern, concise, official |
| UI | **Jetpack Compose** | Declarative เหมือน React |
| Database | **Room** | Type-safe SQLite wrapper |
| HTTP | **Ktor Client** หรือ OkHttp | Coroutine-friendly |
| Image | **Coil** | Kotlin-first image loading |
| Navigation | **Navigation Component** | Back stack ง่าย |
| DI | **Hilt** | Simple dependency injection |
| Architecture | **MVVM + Repository** | Clean separation |
| Charts | **Vico** | Compose-native charts |

---

## 13. สรุป Flow ของ Native App

```
เปิดแอป
  ↓
ตรวจสอบ Settings (Gemini API Key มีหรือยัง?)
  ↓ ไม่มี → แสดง Setup Screen
  ↓ มี
HomeScreen (แสดงรอบเดือนปัจจุบัน)
  ↓ กด +
TransactionForm
  ├── กรอกเอง → บันทึก
  └── กด 📷 → เลือกภาพ → Gemini OCR → auto-fill → บันทึก

Background (ถ้า user อนุญาต):
  NotificationListener รับ notification ธนาคาร
  → parse ด้วย Gemini text API
  → แสดง notification: "พบการโอน 500฿ บันทึกหรือไม่?"
  → user กด Yes → บันทึกอัตโนมัติ
```

---

*เอกสารนี้สร้างจาก Web App GiGi Money (Stang_Money) เวอร์ชัน ณ วันที่ 30 พฤษภาคม 2569*
