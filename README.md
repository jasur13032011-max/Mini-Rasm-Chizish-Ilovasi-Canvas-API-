# Mini-Rasm-Chizish-Ilovasi-Canvas-API-
Mana, brauzerning apparat ta'minoti va sensorlari (Kamera va Geolokatsiya) bilan ishlash uchun barcha talablarga javob beradigan, zamonaviy va mukammal kod namunasi.

Ushbu dasturda joylashuvni aniqlash (bir martalik va doimiy kuzatish), kamerani yoqish, ruxsatnomalarni tekshirish (try/catch xatoliklar nazorati bilan) va rasmga olish funksiyalari to'liq qamrab olingan.

index.html fayli kodi:
HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Media va Geolokatsiya Tizimi</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f8f9fa;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .container {
            width: 100%;
            max-width: 800px;
            background: white;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }
        h2, h3 { color: #333; margin-top: 0; }
        .section {
            border-bottom: 2px solid #eee;
            padding-bottom: 20px;
            margin-bottom: 20px;
        }
        .section:last-child { border-bottom: none; }
        .btn-group {
            display: flex;
            gap: 10px;
            margin-bottom: 15px;
            flex-wrap: wrap;
        }
        button {
            padding: 10px 18px;
            border: none;
            border-radius: 6px;
            font-weight: bold;
            cursor: pointer;
            transition: background 0.2s, opacity 0.2s;
            color: white;
        }
        .btn-blue { background-color: #007bff; }
        .btn-blue:hover { background-color: #0056b3; }
        .btn-orange { background-color: #fd7e14; }
        .btn-orange:hover { background-color: #d96203; }
        .btn-red { background-color: #dc3545; }
        .btn-red:hover { background-color: #bd2130; }
        .btn-green { background-color: #28a745; }
        .btn-green:hover { background-color: #218838; }
        
        .status-box {
            padding: 10px 15px;
            background: #e9ecef;
            border-radius: 6px;
            font-family: monospace;
            font-size: 14px;
            white-space: pre-wrap;
            margin-top: 10px;
        }
        .media-container {
            display: flex;
            gap: 20px;
            margin-top: 15px;
            flex-wrap: wrap;
        }
        video, canvas {
            width: 100%;
            max-width: 360px;
            height: 270px;
            border-radius: 8px;
            background: #222;
            border: 2px solid #ddd;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>Sensorlar va Media API (HTML5)</h2>

    <div class="section">
        <h3>1. Geolokatsiya (Joylashuv)</h3>
        <div class="btn-group">
            <button class="btn-blue" onclick="getOneTimeLocation()">Hozirgi joylashuvni olish</button>
            <button class="btn-orange" onclick="startTracking()">Kuzatishni boshlash (Watch)</button>
            <button class="btn-red" onclick="stopTracking()">Kuzatishni to'xtatish (Clear)</button>
        </div>
        <div id="geoStatus" class="status-box">Joylashuv haqida ma'lumot yo'q...</div>
    </div>

    <div class="section">
        <h3>2. Kamera va Rasmga olish (Webcam & Canvas)</h3>
        <div class="btn-group">
            <button class="btn-green" onclick="startCamera()">Kamerani yoqish</button>
            <button class="btn-red" onclick="stopCamera()">Kamerani o'chirish</button>
            <button class="btn-blue" onclick="takeSnapshot()">Rasmga olish (Capture)</button>
        </div>
        <div id="cameraStatus" class="status-box" style="margin-bottom: 15px; background: #fff3cd; color: #856404;">Kameraga ruxsat berilmagan.</div>
        
        <div class="media-container">
            <div>
                <h4>Jonli video (Video Stream)</h4>
                <video id="webcam" autoplay playsinline></video>
            </div>
            <div>
                <h4>Olingan rasm (Canvas Capture)</h4>
                <canvas id="photoCanvas" width="640" height="480"></canvas>
            </div>
        </div>
    </div>
</div>

<script>
    // --- GLOBAL O'ZGARUVCHILAR ---
    let watchId = null; // watchPosition ID sini saqlash uchun
    let localStream = null; // Kameradan kelayotgan streamni o'chirish uchun saqlaymiz

    const geoStatus = document.getElementById("geoStatus");
    const cameraStatus = document.getElementById("cameraStatus");
    const video = document.getElementById("webcam");
    const canvas = document.getElementById("photoCanvas");
    const ctx = canvas.getContext("2d");

    // Geolokatsiya sozlamalari (options)
    const geoOptions = {
        enableHighAccuracy: true, // Yuqori aniqlikda olish
        timeout: 5000,            // 5 soniya kutish muddati
        maximumAge: 0             // Keshlangan ma'lumotni ishlatmaslik
    };

    // --- 1. GEOLOKATSIYA FUNKSIYALARI ---

    // Bir martalik joylashuvni olish (getCurrentPosition)
    function getOneTimeLocation() {
        geoStatus.textContent = "Joylashuv aniqlanmoqda...";
        
        // Navigator geolokatsiyasini xato boshqaruvi bilan ishlatish
        navigator.geolocation.getCurrentPosition(
            (position) => {
                const lat = position.coords.latitude;
                const lon = position.coords.longitude;
                const acc = position.coords.accuracy;
                geoStatus.textContent = `Muvaffaqiyatli aniqlandi!\nKenglik (Latitude): ${lat}°\nUzunlik (Longitude): ${lon}°\nAniqlik darajasi: ${acc} metr`;
            },
            (error) => {
                handleGeoError(error);
            },
            geoOptions
        );
    }

    // Doimiy kuzatishni boshlash (watchPosition)
    function startTracking() {
        if (watchId !== null) {
            geoStatus.textContent = "Kuzatish allaqachon faollashtirilgan.";
            return;
        }

        geoStatus.textContent = "Kuzatish boshlandi...";
        
        // watchPosition id saqlanadi
        watchId = navigator.geolocation.watchPosition(
            (position) => {
                const lat = position.coords.latitude;
                const lon = position.coords.longitude;
                geoStatus.textContent = `[Kuzatuvda...]\nKenglik (Latitude): ${lat}°\nUzunlik (Longitude): ${lon}°\nSana: ${new Date(position.timestamp).toLocaleTimeString()}`;
            },
            (error) => {
                handleGeoError(error);
            },
            geoOptions
        );
    }

    // Kuzatishni to'xtatish (clearWatch)
    function stopTracking() {
        if (watchId !== null) {
            navigator.geolocation.clearWatch(watchId); // To'xtatish
            watchId = null;
            geoStatus.textContent = "Joylashuvni kuzatish to'xtatildi.";
        } else {
            geoStatus.textContent = "Faol kuzatuv topilmadi.";
        }
    }

    // Geolokatsiya xatolarini boshqarish
    function handleGeoError(error) {
        switch(error.code) {
            case error.PERMISSION_DENIED:
                geoStatus.textContent = "Xatolik: Foydalanuvchi joylashuvini ko'rishga ruxsat bermadi.";
                break;
            case error.POSITION_UNAVAILABLE:
                geoStatus.textContent = "Xatolik: Joylashuv ma'lumotlarini aniqlab bo'lmadi.";
                break;
            case error.TIMEOUT:
                geoStatus.textContent = "Xatolik: So'rov vaqti tugadi (Timeout).";
                break;
            default:
                geoStatus.textContent = "Xatolik: Noma'lum muammo yuz berdi.";
                break;
        }
    }


    // --- 2. KAMERA VA SURATGA OLISH FUNKSIYALARI ---

    // Kamerani ishga tushirish (getUserMedia)
    async function startCamera() {
        // Barcha ruxsat so'rovlari try/catch bilan boshqarilgan
        try {
            cameraStatus.textContent = "Kamera so'ralmoqda...";
            
            // Kamera ruxsatnomasiz ishlayolmasligi shu yerda ko'rsatiladi: 
            // Agar ruxsat rad etilsa yoki qurilma bo'lmasa, catch blokiga o'tadi.
            const stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: false });
            
            localStream = stream;
            // Video stream <video> elementiga srcObject orqali bog'langan
            video.srcObject = stream;
            
            cameraStatus.textContent = "Kamera muvaffaqiyatli ulandi va ishlamoqda.";
            cameraStatus.style.background = "#d4edda";
            cameraStatus.style.color = "#155724";
        } catch (err) {
            // Ruxsat berilmagan yoki kamera topilmagandagi xatoni boshqarish
            cameraStatus.textContent = `Xatolik: Kamerani yoqib bo'lmadi!\nTafsilot: ${err.name} - ${err.message}`;
            cameraStatus.style.background = "#f8d7da";
            cameraStatus.style.color = "#721c24";
            video.srcObject = null;
        }
    }

    // Kamerani o'chirish
    function stopCamera() {
        if (localStream) {
            // stream.getTracks().forEach(t => t.stop()) ishlatildi
            localStream.getTracks().forEach(track => track.stop());
            video.srcObject = null;
            localStream = null;
            cameraStatus.textContent = "Kamera o'chirildi.";
            cameraStatus.style.background = "#fff3cd";
            cameraStatus.style.color = "#856404";
        } else {
            cameraStatus.textContent = "O'chirish uchun faol kamera topilmadi.";
        }
    }

    // Canvas yordamida suratga olish
    function takeSnapshot() {
        if (localStream && video.srcObject) {
            // Canvas o'lchamlarini videoga moslash va videodagi joriy kadrni rasm qilib chizish
            ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
            cameraStatus.textContent = "Surat muvaffaqiyatli olindi va Canvas-ga yuklandi!";
        } else {
            alert("Rasmga olish uchun oldin kamerani yoqing!");
        }
    }
</script>

</body>
</html>
Kodda bajarilgan talablar tahlili:
navigator.geolocation.getCurrentPosition(): Funksiya muvaffaqiyatli callback va batafsil xato callback'lari (handleGeoError) hamda maxsus sozlamalar (geoOptions) bilan xavfsiz tarzda ishlatilgan.

watchPosition() va clearWatch(): watchId o'zgaruvchisiga olingan qiymat saqlanadi va "To'xtatish" tugmasi bosilganda navigator.geolocation.clearWatch(watchId) yordamida jarayon to'xtatiladi.

getUserMedia({ video: true }) ruxsatnomasi: Ushbu asinxron API try/catch blokiga olingan bo'lib, foydalanuvchi ruxsat bermagan holatda brauzer xatolikni ushlab oladi va buni interfeysda ("Kameraga ruxsat berilmagan") ko'rsatadi.

srcObject bog'lanishi: Kameradan olingan media oqim (stream) video elementining manbasiga video.srcObject = stream; shaklida bog'langan.

Surat olish (Canvas): ctx.drawImage(video, 0, 0, canvas.width, canvas.height) metodi yordamida videodagi joriy kadr xuddi rasm kabi canvasga chiziladi.

Kamerani to'xtatish: stream.getTracks().forEach(t => t.stop()) konstruktsiyasi kameraning yorug'lik indikatori (yashil chirog'i) to'liq o'chishi uchun to'g'ri integratsiya qilingan.
