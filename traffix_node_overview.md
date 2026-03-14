# 🌐 Traffix Node - Tizimning To'liq Tahlili va Ishlash Prinsipi

**Traffix Node** — bu oddiy Android smartfonlarni yuqori tezlikdagi rezident proksi tugunlariga (nodes) aylantirish orqali markazlashmagan internet tarmog'ini (DePIN) yaratishga qaratilgan innovatsion platforma.

## 👥 1. Tizimning Asosiy Qatnashuvchilari

Traffix Node ekotizimi ikkita asosiy qatnashuvchidan iborat:

*   **Provider (Konchi/Sotuvchi):** O'zining Android telefoniga Traffix Node ilovasini o'rnatib, ishlatilmayotgan (ortiqcha) Wi-Fi yoki mobil internet traffigini tarmoqqa ulashuvchi oddiy foydalanuvchi. Ular internetini ulashgani uchun avtomatik tarzda balansiga **USDT** (kriptovalyuta) ishlab borishadi (DePIN Mining).
*   **Consumer (Xaridor / Mijoz):** Geografik to'siqlarni aylanib o'tish, bot tarmog'ini yurgizish, ma'lumot yig'ish (parsing) kabi turli maqsadlar uchun xalqaro darajadagi (haqiqiy odamlarning) IP-manziliga muhtoj IT-kompaniyalar, laboratoriyalar. Ular ushbu proksilar (Pocket+, SOCKS5) xizmatidan foydalanish uchun obuna to'lovini amalga oshiradilar.

## ⚙️ 2. Tizimning Ishlash Prinsipi (Qanday bog'lanadi?)

Tizim ishlash jarayoni markaziy chok (Gateway Server) orqali boshqariladi:

1.  **Tayyorgarlik:** Telefon (Provider) to'g'ridan-to'g'ri ochiq port saqlay olmaydi (NAT ortida bo'lgani uchun). Buning o'rniga, Traffix Node ilovasi **Backend Serverga** maxsus Tunnel protokoli / SOCKS5 orqali doimiy ulanib turadi va "Men tayyorman" degan xabar yuboradi.
2.  **Navbatga qo'shish:** Server bu telefon uchun tunel ajratadi va uning IP manzilini **"Online"** qurilmalar bazasiga (`Redis Pool`) saqlab qo'yadi.
3.  **So'rovni Yo'naltirish:** Xaridor (Consumer) kerakli manzilga (masalan biron veb-saytga) kirishga harakat qilganida, Backend Server unga avtomatik ravishda tayyor turgan navbatdagi Provider (telefon) tunnelini biriktiradi. Xaridorning so'rovi to'g'ridan to'g'ri ana shu provyader ulanishi orqali yakuniy manzilga yetib boradi.
4.  **Billing (Hisob-kitob):** Shu zanjir orqali o'tkazilgan har bir ma'lumot qabuli va uzatmasi (Megabayt va Gigabaytlar) Backend xotirasida soniyasma-soniya hisoblanib boradi va Providerning balansiga mos ravishda USDT qo'shiladi.

## 🧰 3. Asosiy Modullar (Tizimning ichki mexanizmlari)

*   **SOCKS5 & Tunnel Manager (Tunnel Menejeri):** Barcha jarayonning yuragi. Xaridor trafigi ushbu "SOCKS5 qabul qiluvchi" orqali kelib, qat'iy filtrlashdan o'tadi ([isDestinationBlocked](file:///home/admin/opt/Traffix_Node/backend/internal/modules/receiver/full_receiver.go#263-306)). Buzg'unchi saytlar (masalan porn, LAN, SMTP, kazino va h.k.) va qoidalarga zid bo'lgan portlar bloklanadi. Ddos yoki tarmoq xakerlari ([isIPBlocked](file:///home/admin/opt/Traffix_Node/backend/internal/modules/receiver/full_receiver.go#244-262)) mexanizmi orqali 10 daqiqalik jazoga (block IPs) tushadi.
*   **Traffic Radar (Tarmoq Radari):** Tizim sifati bo'yicha "nazoratchi miya". U doimiy ravishda (har 60 soniyada) provayder tunnellari tezligi va haqqoniyligini "ping" qilib tekshirib turadi; server markazlariga (hostinglarga) tegishli IP-larni aniqlasa yoki interneti pasayib qolsa, sessiyani va pul to'lanishini [red](file:///home/admin/opt/Traffix_Node/backend/internal/core/context.go#14-15) flag qilib cheklaydi (Jazolaydi).
*   **Batch Billing & Finance (Moliya dvigateli):** Mb va GB larni hisoblashdagi zo'riqishga qarshi qilingan tizim. Tarmoq orqali o'tgan har bir Mb uchun bazaga so'rov yuborilsa, tizim qotadi (deadlock). Shuning uchun server MB larni `atomic.Int64` yordamida tezkor RAM-da yig'ib borib, faqatgina chegara hajmdan oshganda yoki har n-* soniyada umumiy holatni bir harakatda **PostgreSQL** gayozi boladi.
*   **API Gateway & Autentifikatsiya:** Tizimga xavfsiz ulanish darvozasi. Barcha mijoz va operatorlar ma'lumotlari (`sessions`, `tokens`) eng xavfsiz JWT (JSON Web Token) usulida va Telegram avtorizatsiya orqali boshqariladi.
*   **AI Compute Engine (Rivojlanish bosqichi):** Oddiy telefon faqat tarmoq bermasdan, o'z protsesor resurslarini (NPU/GPU) sun'iy intellekt hisob-kitoblari (Deep learning tasks) ga taqsimlab ishlashi uchun qurilayotgan superkompyuter klasteri.

## 🛠 4. Ishlatilgan Texnologiyalar (Tech Stack)

Asosiy kompotnentlar yuqori yuklamalarga chidamli tizimlar ustiga qurilgan:
*   **Golang (Go 1.24+):** Butun Backend yadrosi. Ulkan sondagi (minglab) tunnels parallel ulanishlarini uzluksiz ta'minlovchi asinxron tillar orasida yetakchiligi uchun tanlangan.
*   **PostgreSQL:** Moliyaviy va doimiy ma'lumotlar saqlanadigan xavfsiz (relational) ma'lumotlar bazasi.
*   **Redis:** Juda tezlikni talab qiladigan operatsiyalar - (qaysi IP hozir online, spam hujumdan himoya - Rate Limit) kabilar uchun ishlatiladigan operativ omborxona.
*   **sslh va Nginx:** Tarmoq uzellari. Dasturda `sslh` yagona (443) portni eshitib, qanday turdagi so'rov kelsa o'z joyiga (HTTPS -> Nginx'ga, Tunnel -> Go Backend'ga) adashmasdan yo'naltiruvchi maxsus eshik vazifasini bajaradi. Nginx esa yordamchi saytlarga reverse-proxy vazifasid.
*   **Kotlin (Jetpack Compose):** Android ilova yaratilayotgan ilg'or mobil til va arxitektura.

## 📦 5. Loyihadagi Asosiy Mahsulotlar (Ekotizim)

Traffix Node ushbu 4 ta muhim bo'g'inning yaxlitligidan iboratdir:
1.  **Traffix Node Mobil Ilovasi (Android):** Orqa fonga o'tib, konchi vazifasini bajaruvchi, foydalanuvchiga balans, statistika va pul yechish hisobini ko'rsatib turuvchi asosiy pul ishlab-topuvchi interfeys dasturi.
2.  **Traffix Backend Server:** Butun tizimni va moliyaviy travezalarni hisoblovchi uzluksiz "Miya" (Gateway).
3.  **Mijozlar Portali (Traffix Pocket+):** Boshqa davlat proksilarini obuna asosida sotib oluvchi xaridorlar va ularni API usuliga bog'laydigan mahsulotlar (B2B, B2C yo'nalishi).
4.  **Boshqaruv Panellari (Veb Potalar):**
    *   `admin.traffix.uz`: Sayt egalari pul yechilganlikni tasdiqlaydigan, statistikalarni ko'radigan yopiq qism.
    *   `call.traffix.uz`: Call-center xodimlari pul yechishida yuzaga kelgan mijoz muammolarini kuzatib, yordam xati yozishuvchatini olib boradigan portal.
    *   `traffix.uz`: Loyihani ommaga reklama qiluvchi asosiy tushuntirish sahifasi va Mobil ilovani ko'chirib olish (Download) oynasi.

---
*Xulosa qilib aytganda, Traffix Node - uy telefonlari armiyasini birlashtiruvchi juda chidamli, ishonchli VPN va potentsial hisoblash qudratiga ega tijoriy markazsizlashgan Broker infrastrukturasi.*
