# Mini-Rasm-Chizish-Ilovasi-Canvas-API-
PWA (Progressive Web App) texnologiyasining eng muhim qismi — bu Service Worker va Manifest fayllari hisoblanadi. Quyida barcha talablarga javob beradigan, tarmoq uzilganda offline rejimda ishlovchi va eski keshlarni avtomatik tozalovchi to'liq tizim kodi keltirilgan.

Loyihani ishga tushirish uchun sizga 3 ta fayl kerak bo'ladi.

1. manifest.json
Ilovani telefonga yoki kompyuterga o'rnatish imkonini beruvchi barcha zarur maydonlar bilan shakllantirilgan manifest fayli.

JSON
{
  "short_name": "PWA_App",
  "name": "Mening Offline PWA Ilovam",
  "icons": [
    {
      "src": "icon-192.png",
      "type": "image/png",
      "sizes": "192x192"
    },
    {
      "src": "icon-512.png",
      "type": "image/png",
      "sizes": "512x512"
    }
  ],
  "start_url": "/index.html",
  "background_color": "#007bff",
  "theme_color": "#007bff",
  "display": "standalone",
  "orientation": "portrait"
}
(Eslatma: icon-192.png va icon-512.png rasmlari loyiha papkasida mavjud bo'lishi kerak)

2. index.html va offline.html
index.html (Asosiy sahifa va SW ro'yxatdan o'tkazish):

HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PWA Tizimi</title>
    <link rel="manifest" href="manifest.json">
    <style>
        body { font-family: sans-serif; text-align: center; background: #f4f6f9; padding: 50px; }
        .card { background: white; padding: 30px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); display: inline-block; }
        h1 { color: #007bff; }
    </style>
</head>
<body>

    <div class="card">
        <h1>Mening PWA Ilovam</h1>
        <p>Ushbu sahifa oflayn rejimda ham mukammal ishlaydi!</p>
    </div>

    <script>
        // Service Worker ni ro'yxatdan o'tkazish
        if ('serviceWorker' in navigator) {
            window.addEventListener('load', () => {
                navigator.serviceWorker.register('sw.js')
                    .then(reg => console.log('Service Worker muvaffaqiyatli ro\'yxatdan o\'tdi:', reg.scope))
                    .catch(err => console.error('SW ro\'yxatdan o\'tishda xatolik:', err));
            });
        }
    </script>
</body>
</html>
offline.html (Internet o'chganda ko'rsatiladigan zaxira sahifa):

HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Oflayn Rejim</title>
    <style>
        body { font-family: sans-serif; text-align: center; background: #f8d7da; color: #721c24; padding: 100px; }
        h1 { margin-bottom: 10px; }
    </style>
</head>
<body>
    <h1>Internet bilan aloqa yo'q</h1>
    <p>Iltimos, tarmoq ulanishini tekshiring. Hozircha siz oflayn rejimdasiz.</p>
</body>
</html>
3. sw.js (Service Worker fayli)
Kesh nomi o'zgarganda eskilarni o'chiruvchi, Cache-first strategiyasida ishlovchi va xatolikda offline.html sahifasini qaytaruvchi logika.

JavaScript
const CACHE_NAME = "v2"; // Yangi versiyaga o'tilganda eski keshlar o'chiriladi
const ASSETS_TO_CACHE = [
    "/index.html",
    "/offline.html",
    "/manifest.json"
];

// 1. INSTALL hodisasi - Resurslarni keshga saqlash
self.addEventListener("install", (e) => {
    console.log("SW: Install hodisasi ishga tushdi");
    e.waitUntil(
        caches.open(CACHE_NAME).then((cache) => {
            console.log("SW: Fayllar keshga yozilmoqda");
            return cache.addAll(ASSETS_TO_CACHE);
        }).then(() => self.skipWaiting()) // Yangi SW ni darhol faollashtirish
    );
});

// 2. ACTIVATE hodisasi - Eski keshlar ro'yxatini tozalash
self.addEventListener("activate", (e) => {
    console.log("SW: Activate hodisasi ishga tushdi");
    e.waitUntil(
        caches.keys().then((cacheNames) => {
            return Promise.all(
                cacheNames.map((cache) => {
                    // Agar mavjud kesh nomi joriy CACHE_NAME ga teng bo'lmasa, uni o'chiramiz
                    if (cache !== CACHE_NAME) {
                        console.log("SW: Eski kesh o'chirilmoqda:", cache);
                        return caches.delete(cache);
                    }
                })
            );
        }).then(() => self.clients.claim()) // Barcha ochiq tablarni nazoratga olish
    );
});

// 3. FETCH hodisasi - Cache-First strategiyasi va Offline rejim boshqaruvi
self.addEventListener("fetch", (e) => {
    e.respondWith(
        caches.match(e.request).then((cachedResponse) => {
            // Keshda fayl bo'lsa, uni qaytaramiz (Cache-first)
            if (cachedResponse) {
                return cachedResponse;
            }

            // Keshda bo'lmasa, tarmoqqa so'rov yuboramiz
            return fetch(e.request).catch(() => {
                // Tarmoq o'chib qolganda (Internet yo'q bo'lsa) offline.html sahifasini qaytarish
                if (e.request.mode === 'navigate') {
                    return caches.match("/offline.html");
                }
            });
        })
    );
});
💡 Kodning muhim qismlari qanday ishladi?
Eski keshni o'chirish: activate hodisasi ichida caches.keys() yordamida barcha keshlar tekshirildi va v2 (joriy nom) dan boshqalari caches.delete(cache) orqali o'chirib tashlandi.

Cache-First strategiyasi: fetch hodisasida avval caches.match(e.request) tekshirilib, keshda bor ma'lumot uzatildi, bo'lmasa fetch(e.request) orqali tarmoqdan yuklandi.

Oflayn zaxira (offline.html): Agar foydalanuvchida internet uzilsa va keshda yo'q yangi sahifaga o'tishga urinsa, .catch() bloki ishga tushib, zaxiradagi offline.html faylini taqdim etadi.

Eslatma: Service Worker xavfsizlik sababli faqat localhost da yoki haqiqiy serverda https:// protokoli ostida ishlaydi.
