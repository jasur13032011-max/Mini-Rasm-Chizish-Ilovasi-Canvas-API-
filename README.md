# Mini-Rasm-Chizish-Ilovasi-Canvas-API-
Mana, fayllarni sudrab kelib tashlash (Drag and Drop) tizimi uchun barcha shartlarga to'liq javob beradigan, chiroyli dizayn va to'liq JavaScript funksionalligiga ega kod namunasi.

Ushbu loyihada rasmlar ekranda vizual ko'rinadi, matnli fayllar (.txt, .json va h.k.) tarkibi esa o'qilib, maxsus qutida ko'rsatiladi.

index.html fayli kodi:
HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Drag and Drop Fayl Yuklagich</title>
    <style>
        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            background-color: #f7f9fc;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        h2 { color: #333; }
        
        /* Drag and Drop Hududi */
        .drop-zone {
            width: 100%;
            max-width: 600px;
            height: 200px;
            border: 3px dashed #007bff;
            border-radius: 12px;
            background-color: #fff;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: #007bff;
            cursor: pointer;
            transition: all 0.3s ease;
            margin-bottom: 20px;
        }
        /* Sichqoncha ustiga kelgandagi holat */
        .drop-zone.drag-over {
            background-color: #e6f2ff;
            border-color: #0056b3;
            transform: scale(1.02);
        }
        .drop-zone p {
            margin: 10px 0 0 0;
            font-size: 16px;
            font-weight: bold;
        }

        /* Natijalar ro'yxati */
        .file-list {
            width: 100%;
            max-width: 600px;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }
        .file-card {
            background: white;
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.05);
            border-left: 5px solid #28a745;
        }
        .file-info {
            font-size: 14px;
            color: #555;
            margin-bottom: 10px;
        }
        .file-info strong { color: #222; }
        
        /* Fayl kontenti uchun vizualizatsiya */
        .preview-img {
            max-width: 100%;
            max-height: 200px;
            border-radius: 6px;
            margin-top: 5px;
            border: 1px solid #ddd;
        }
        .preview-text {
            background: #f1f3f5;
            padding: 10px;
            border-radius: 6px;
            max-height: 150px;
            overflow-y: auto;
            font-family: monospace;
            font-size: 13px;
            white-space: pre-wrap;
            border: 1px solid #e2e8f0;
        }
        .error-card {
            border-left-color: #dc3545;
            color: #dc3545;
        }
    </style>
</head>
<body>

    <h2>Fayllarni yuklash (Drag & Drop)</h2>
    
    <div id="dropZone" class="drop-zone">
        <svg width="48" height="48" fill="currentColor" viewBox="0 0 16 16">
            <path d="M.5 9.9a.5.5 0 0 1 .5.5v2.5a1 1 0 0 0 1 1h12a1 1 0 0 0 1-1v-2.5a.5.5 0 0 1 1 0v2.5a2 2 0 0 1-2 2H2a2 2 0 0 1-2-2v-2.5a.5.5 0 0 1 .5-.5z"/>
            <path d="M7.646 1.146a.5.5 0 0 1 .708 0l3 3a.5.5 0 0 1-.708.708L8.5 2.707V11.5a.5.5 0 0 1-1 0V2.707L5.354 4.854a.5.5 0 1 1-.708-.708l3-3z"/>
        </svg>
        <p>Fayllarni shu yerga sudrab keling (Faqat rasm va matnlar)</p>
    </div>

    <div id="fileList" class="file-list"></div>

<script>
    const dropZone = document.getElementById("dropZone");
    const fileList = document.getElementById("fileList");

    // --- 1. HODISALARNI BOSHQARISH (Drag & Drop) ---

    // dragover: fayl zona ustida turganda
    dropZone.addEventListener("dragover", (e) => {
        e.preventDefault(); // Brauzer faylni ochib yubormasligi uchun shart
        dropZone.classList.add("drag-over");
    });

    // dragleave: fayl zonadan chiqib ketganda
    dropZone.addEventListener("dragleave", () => {
        dropZone.classList.remove("drag-over");
    });

    // drop: fayl zonaga tashlanganda
    dropZone.addEventListener("drop", (e) => {
        e.preventDefault(); // Brauzer defolt amalini to'xtatish
        dropZone.classList.remove("drag-over");

        // e.dataTransfer.files orqali bir vaqtning o'zida bir nechta fayllarni olish
        const files = e.dataTransfer.files;
        
        if (files.length > 0) {
            handleFiles(files);
        }
    });

    // --- 2. FAYLLARNI QAYTA ISHLASH FUNKSIYASI ---
    function handleFiles(files) {
        // Har bir faylni siklda aylanish (Bir nechta fayl tushirilganda ishlaydi)
        Array.from(files).forEach(file => {
            createFileCard(file);
        });
    }

    // --- 3. FAYL HAJMING FORMATLASH (KB/MB) ---
    function formatFileSize(bytes) {
        if (bytes === 0) return '0 Bytes';
        const k = 1024;
        const sizes = ['Bytes', 'KB', 'MB', 'GB'];
        const i = Math.floor(Math.log(bytes) / Math.log(k));
        return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
    }

    // --- 4. FAYL KARTASINI YARATISH VA O'QISH ---
    function createFileCard(file) {
        const card = document.createElement("div");
        card.className = "file-card";

        const formattedSize = formatFileSize(file.size);
        const lastModDate = new Date(file.lastModified).toLocaleDateString();

        // Boshlang'ich ma'lumotlar strukturasi (file.name, file.size, file.type, file.lastModified)
        card.innerHTML = `
            <div class="file-info">
                <strong>Nomi:</strong> ${file.name} <br>
                <strong>Hajmi:</strong> ${formattedSize} <br>
                <strong>Turi:</strong> ${file.type || 'Noma\'lum'} <br>
                <strong>O'zgartirilgan sana:</strong> ${lastModDate}
            </div>
            <div class="preview-container">Yuklanmoqda...</div>
        `;
        
        fileList.appendChild(card);
        const previewContainer = card.querySelector(".preview-container");

        // --- 5. file.type TEKSHIRUVI (Faqat rasm va matn) ---
        const reader = new FileReader();

        if (file.type.startsWith("image/")) {
            // Rasm bo'lsa: readAsDataURL() ishlatiladi
            reader.readAsDataURL(file);
            
            reader.onload = function(e) {
                previewContainer.innerHTML = `<img src="${e.target.result}" class="preview-img" alt="Preview">`;
            };
            reader.onerror = function() {
                previewContainer.innerHTML = "<span style='color:red;'>Rasmni o'qishda xatolik!</span>";
            };

        } else if (file.type.startsWith("text/") || file.name.endsWith(".json") || file.name.endsWith(".js")) {
            // Matnli fayl bo'lsa: readAsText() ishlatiladi
            reader.readAsText(file);
            
            reader.onload = function(e) {
                // Xavfsizlik uchun matn ichidagi < va > belgilarini almashtiramiz (HTML injection oldini olish)
                const safeText = e.target.result.replace(/</g, "&lt;").replace(/>/g, "&gt;");
                previewContainer.innerHTML = `<pre class="preview-text">${safeText}</pre>`;
            };
            reader.onerror = function() {
                previewContainer.innerHTML = "<span style='color:red;'>Matnni o'qishda xatolik!</span>";
            };

        } else {
            // Agar rasm yoki matn bo'lmasa, rad etiladi
            card.classList.add("error-card");
            previewContainer.innerHTML = `<strong>Xatolik:</strong> Ushbu fayl turi qo'llab-quvvatlanmaydi (Faqat rasm va matn yuklang).`;
        }
    }
</script>

</body>
</html>
Kodda bajarilgan talablar tahlili:
Hodisalar (dragover, dragleave, drop): To'liq qo'shildi. dragover va drop ichida e.preventDefault() qo'llanib, brauzer faylni o'z-o'zidan yangi tabda ochib yuborishining oldi olindi.

e.dataTransfer.files: Tashlangan barcha fayllar ro'yxatini massiv ko'rinishida ushlab olish uchun muvaffaqiyatli ishlatildi.

FileReader metodlari:

file.type.startsWith("image/") orqali rasm aniqlansa, reader.readAsDataURL(file) chaqiriladi.

Matnli fayllar uchun esa reader.readAsText(file) ishga tushadi.

Hajmni formatlash: formatFileSize() funksiyasi yordamida har bir fayl hajmi dinamik ravishda KB yoki MB o'lchov birligiga o'tkaziladi.

file.type tekshiruvi: Tizim rasm va matndan boshqa formatdagi fayllarni qabul qilmaydi va foydalanuvchiga qizil ogohlantirish kartasini (error-card) ko'rsatadi.

Ko'p sonli fayllar (Multiple Files): Array.from(files).forEach(...) yordamida foydalanuvchi bir vaqtning o'zida 3-4 ta faylni birdiga tashlasa ham, har biri alohida kartada muvaffaqiyatli qayta ishlanadi.
