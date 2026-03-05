# 📘 Traffix Client API - Ishlab chiquvchilar uchun To'liq Qo'llanma

Ushbu hujjat API orqali Traffix Node tarmog'idan SOCKS5 proksilarni sotib oluvchi va ulanuvchi xaridorlar (Client/Consumer) uchun mo'ljallangan. Barcha API so'rovlarni o'z dasturlaringiz (Python, Node.js) yoki terminalda `cURL` orqali ishlatishingiz mumkin.

*Asosiy manzil:* `https://api.traffix.uz`

---

## 1. 🔑 Autentifikatsiya va Raqamli Kalit (API Key) Olish

Xaridorlar API-dan foydalanish uchun dastlab ro'yxatdan o'tishlari va `API Key` hamda `Secret Key` olishlari kerak.
- Barcha qolgan API so'rovlarining sarlavhasida (header) `X-API-Key: sizning_kalitingiz` ko'rsatilishi shart!
- Olingan `Secret Key` sizning barcha SOCKS5 proksilaringizga ulanish uchun yagona parol (password) vazifasini ham bajaradi.

### 📝 Yangi hisob yaratish (Terminal orqali cURL)
**So'rov:**  `POST /api/v1/client/register` (Bu ochiq endpoint, kalit kerak emas).

```bash
curl -X POST https://api.traffix.uz/api/v1/client/register \
  -H "Content-Type: application/json" \
  -d '{
        "name": "Abdulla Qodiriy", 
        "email": "abdulla@example.com", 
        "plan": "premium"
      }'
```
**Natija:**
```json
{
  "success": true,
  "client_id": 1,
  "api_key": "tk_1234567890abcdef1234567890abcdef",
  "secret_key": "abc123fed456...",
  "plan": "premium",
  "bandwidth": "100 GB"
}
```

*(Quyidagi barcha so'rovlarda API tokenni tezkor ishlatish uchun Linux/Mac terminalida `API_KEY` o'zgaruvchisini saqlab qo'yamiz):*
```bash
export API_KEY="tk_1234567890abcdef1234567890abcdef"
```

---

## 2. 🌍 Proksi (SOCKS5) Olish

Istagan vaqtingiz bo'sh va tayyor IP proksini so'rash! API sizga IP kodi va Portlarni qaytaradi.

### 🎲 Bitta tasodifiy proxy olish
```bash
curl -X GET https://api.traffix.uz/api/v1/client/proxy \
  -H "X-API-Key: $API_KEY"
```

### 🎯 Filtrlangan proxy olish (Aniq davlat va tashkilot)
Masalan faqat O'zbekiston, Toshkent shahri hamda aynan "Ucell" operatoridan kerak:
```bash
curl -X GET "https://api.traffix.uz/api/v1/client/proxy?country=UZ&city=Tashkent&isp=Ucell" \
  -H "X-API-Key: $API_KEY"
```

---

## 3. 🔄 IP ni Yangilash (Rotate)
IP bloklanganini sezsangiz, ushbu manzil orqali yangi toza IP'ni so'rab olishingiz mumkin. Eski proxy yopilib o'rniga yangisi beriladi.
```bash
curl -X GET "https://api.traffix.uz/api/v1/client/proxy/rotate?country=UZ" \
  -H "X-API-Key: $API_KEY"
```

---

## 4. 📋 Proksilar Ro'yxatini Olish (Birdaniga ko'p)
Botlar va skriptlar uchun bir necha marta murojaat qilmaslik maqsadida bitta request bilan o'nlab proxy IP'larni qabul qilish.
Mana bu misolda O'zbekistondan `count=5` ta (5 ta IP) so'ralmoqda:
```bash
curl -X GET "https://api.traffix.uz/api/v1/client/proxy/list?count=5&country=UZ" \
  -H "X-API-Key: $API_KEY"
```

---

## 5. 🧲 Yopishqoq Sessiya (Sticky Session / Muzlatish)
Sessiya yoki hisob raqamlarga loginda bitta IP ma'lum vaqt o'zgarmay turishi muhim. Buning uchun bitta proxy'ni ma'lum vaqtga muzlatib (band qilib) qo'yasiz.

**1-qadam: IP ni muzlatish va ID olish:**
```bash
curl -X GET "https://api.traffix.uz/api/v1/client/session/create?country=US" \
  -H "X-API-Key: $API_KEY"
```
*Tizim sizga `session_id` qaytaradi. Masalan `sess_Abcde`*

**2-qadam: IP'ga qayta-qayta murojaat qilish (Session amal qilish vaqtida):**
```bash
curl -X GET "https://api.traffix.uz/api/v1/client/proxy?session=sess_Abcde" \
  -H "X-API-Key: $API_KEY"
```

**3-qadam: Sessiyani o'z vaqtidan oldin yopish (bo'shatish):**
```bash
curl -X POST "https://api.traffix.uz/api/v1/client/session/close" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"session_id": "sess_Abcde"}'
```

---

## 6. 🌐 Qo'shimcha Ma'lumotlarni Tekshirish (Diktatorlar tarmog'i)
Hozirda Traffix Node ichida (foydalanuvchilar qayoqdan ulanayotgan) qaysi davlatlardan internet oltsa bo'lishini bilish.

🔹 **Davlatlarni ro'yxati:**
```bash
curl -X GET https://api.traffix.uz/api/v1/client/proxy/countries \
  -H "X-API-Key: $API_KEY"
```
🔹 **Shaharlar ro'yxati ("UZ" misolida):**
```bash
curl -X GET "https://api.traffix.uz/api/v1/client/proxy/cities?country=UZ" \
  -H "X-API-Key: $API_KEY"
```
🔹 **Mavjud Operator / ISPlar:**
```bash
curl -X GET "https://api.traffix.uz/api/v1/client/proxy/isps?country=UZ" \
  -H "X-API-Key: $API_KEY"
```

---

## 7. 📊 Moliya va Statistika
Tarif bilan berilgan Bandwidth (GB trafikingiz) miqdori hamda tizimdagi sarf qilingan miqdorlar haqida statistikalar:

🔹 **Joriy balansingiz va trafikingizni ko'rish:**
```bash
curl -X GET https://api.traffix.uz/api/v1/client/stats \
  -H "X-API-Key: $API_KEY"
```
🔹 **Oxirgi 7 kundagi sarf-xarajatingiz grafiklari:**
```bash
curl -X GET "https://api.traffix.uz/api/v1/client/usage?days=7" \
  -H "X-API-Key: $API_KEY"
```

---

## 8. 💻 Dasturlash tillarida kod misollari

### 🐍 Python (Requests orqali Proxy olib internetga ulanish)
```python
import requests

API_KEY = "tk_1234567890abcdef1234567890abcdef"
URL = "https://traffix.uz/api/v1/client/proxy"

headers = {"X-API-Key": API_KEY}
params = {"country": "UZ"} # UZ davlatidan

response = requests.get(URL, headers=headers, params=params)

if response.status_code == 200:
    data = response.json()
    p = data['proxy']
    print(f"✅ Topildi: {p['host']}:{p['port']} ({p['isp']})")
    
    # Endilikda ushbu IP dan ishlatamiz
    socks_proxy = f"socks5://{p['username']}:{p['password']}@{p['host']}:{p['port']}"
    proxies = {"http": socks_proxy, "https": socks_proxy}
    
    ip_check = requests.get("https://api.ipify.org?format=json", proxies=proxies)
    print("Sizning endigi IP dresingiz:", ip_check.json())
```

### ☕ Node.js (Async/await Fetch orqali SOCKS5 olish)
```javascript
const API_KEY = "tk_1234567890abcdef1234567890abcdef";

async function runMyBot() {
  const response = await fetch("https://traffix.uz/api/v1/client/proxy?country=US", {
    headers: { "X-API-Key": API_KEY }
  });

  const data = await response.json();
  if (data.success) {
    console.log("Men AQSH ipisini oldim!", data.proxy.host);
    
    // Puppeteer kabi asboblarda proxy ni qotiramiz:
    // args: [`--proxy-server=socks5://${data.proxy.host}:${data.proxy.port}`]
  }
}
runMyBot();
```
