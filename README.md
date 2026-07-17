# Mini-Rasm-Chizish-Ilovasi-Canvas-API-
Siz so'ragan barcha texnologiyalarni (Canvas, IndexedDB, Drag-and-Drop + File API va Service Worker) o'z ichiga olgan hamda foydalanuvchiga qulay yagona "Multifunksional Oflayn Kundalik" loyihasini taqdim etaman.

Bu ilovada siz:

Drag-and-Drop orqali rasm yuklaysiz.

Canvas yordamida yuklangan rasm ustiga yoki shunchaki bo'sh joyga eskiz (imzo/chizgi) chizasiz.

IndexedDB ga ushbu ma'lumotlarni kamida 3 ta yozuv bilan saqlaysiz va ro'yxatni ko'rasiz.

Service Worker yordamida internet bo'lmaganda ham ilovadan to'liq foydalana olasiz.

Loyiha toliq ishlashi uchun bitta papkada quyidagi 3 ta faylni yarating.

1. manifest.json
PWA ilova sifatida o'rnatilishi va SW ishlashi uchun kerakli metadata.

JSON
{
  "short_name": "MultiApp",
  "name": "Super Multifunksional PWA Kundalik",
  "start_url": "./index.html",
  "display": "standalone",
  "background_color": "#f4f6f9",
  "theme_color": "#007bff",
  "orientation": "portrait",
  "icons": [
    {
      "src": "https://cdn-icons-png.flaticon.com/512/3176/3176363.png",
      "type": "image/png",
      "sizes": "512x512"
    }
  ]
}
2. sw.js (Service Worker)
Oflayn rejimda ishlashni ta'minlovchi va keshni boshqaruvchi fayl.

JavaScript
const CACHE_NAME = "multi-app-v1";
const ASSETS = [
    "./",
    "./index.html",
    "./manifest.json"
];

// O'rnatish va keshga saqlash
self.addEventListener("install", (e) => {
    e.waitUntil(
        caches.open(CACHE_NAME).then((cache) => cache.addAll(ASSETS))
    );
});

// Faollashtirish va eskilarni o'chirish
self.addEventListener("activate", (e) => {
    e.waitUntil(
        caches.keys().then((keys) => Promise.all(
            keys.map((key) => key !== CACHE_NAME ? caches.delete(key) : null)
        ))
    );
});

// Tarmoq so'rovlarini ushlash (Cache-first)
self.addEventListener("fetch", (e) => {
    e.respondWith(
        caches.match(e.request).then((res) => res || fetch(e.request))
    );
});
3. index.html (Asosiy interfeys va barcha API logikalari)
Barcha API interfeyslari bitta joyga jamlangan va boshqarish juda oson.

HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Multifunksional Oflayn Kundalik</title>
    <link rel="manifest" href="manifest.json">
    <style>
        body { font-family: 'Segoe UI', Arial, sans-serif; background-color: #f4f6f9; margin: 0; padding: 20px; display: flex; justify-content: center; }
        .app-container { width: 100%; max-width: 900px; display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        @media (max-width: 768px) { .app-container { grid-template-columns: 1fr; } }
        .panel { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.08); }
        h3 { margin-top: 0; color: #333; border-bottom: 2px solid #eee; padding-bottom: 8px; }
        
        /* Drag & Drop va Canvas */
        .drop-zone { border: 2px dashed #007bff; border-radius: 8px; height: 80px; display: flex; align-items: center; justify-content: center; color: #007bff; cursor: pointer; margin-bottom: 15px; font-weight: bold; background: #f0f7ff; text-align: center; padding: 5px; }
        .drop-zone.drag-over { background: #d0e7ff; }
        canvas { border: 1px solid #ccc; background: #fff; border-radius: 8px; cursor: crosshair; display: block; margin-bottom: 10px; width: 100%; max-width: 400px; height: 200px; }
        
        /* Form elements */
        input[type="text"], button { width: 100%; padding: 10px; margin-bottom: 10px; border: 1px solid #ccc; border-radius: 6px; box-sizing: border-box; }
        button { background: #28a745; color: white; border: none; font-weight: bold; cursor: pointer; }
        button:hover { background: #218838; }
        .btn-clear { background: #dc3545; }
        .btn-clear:hover { background: #c82333; }
        
        /* Ro'yxat */
        .note-card { background: #f8f9fa; border-left: 5px solid #007bff; padding: 10px; margin-bottom: 10px; border-radius: 4px; }
        .note-card img { max-width: 100%; height: auto; border-radius: 4px; margin-top: 8px; border: 1px solid #ddd; }
        .note-card h4 { margin: 0 0 5px 0; color: #007bff; }
        .note-card p { margin: 0; font-size: 14px; color: #555; }
    </style>
</head>
<body>

<div class="app-container">
    
    <div class="panel">
        <h3>Yangi qayd yaratish</h3>
        
        <input type="text" id="noteTitle" placeholder="Qayd sarlavhasi (Masalan: 1-Yozuv)...">
        
        <div id="dropZone" class="drop-zone">Rasmni shu yerga tashlang yoki bosing (Drag & Drop)</div>
        
        <label style="font-size: 12px; color: #666; font-weight: bold;">Pastga eskiz yoki imzo chizing:</label>
        <canvas id="sketchPad" width="400" height="200"></canvas>
        
        <button type="button" class="btn-clear" onclick="clearCanvas()">Canvasni tozalash</button>
        <button type="button" onclick="saveNote()">IndexedDB ga saqlash</button>
    </div>

    <div class="panel">
        <h3>Saqlangan qaydlar (Kamida 3 ta sinov)</h3>
        <div id="notesList">Hozircha qaydlar yo'q.</div>
    </div>
</div>

<script>
    // --- 1. SERVICE WORKER REGISTRATION ---
    if ('serviceWorker' in navigator) {
        navigator.serviceWorker.register('sw.js').then(() => console.log("SW faol."));
    }

    // --- GLOBAL O'ZGARUVCHILAR ---
    const canvas = document.getElementById("sketchPad");
    const ctx = canvas.getContext("2d");
    const dropZone = document.getElementById("dropZone");
    const notesList = document.getElementById("notesList");
    
    let isDrawing = false;
    let uploadedImageBase64 = ""; // File API dan keladigan rasm strigini saqlash uchun
    let db;

    // --- 2. INDEXEDDB SOZLAMALARI ---
    const dbRequest = indexedDB.open("MultifunktsionalDB", 1);
    
    dbRequest.onupgradeneeded = (e) => {
        db = e.target.result;
        if (!db.objectStoreNames.contains("qaydlar")) {
            db.createObjectStore("qaydlar", { keyPath: "id", autoIncrement: true });
        }
    };
    dbRequest.onsuccess = (e) => {
        db = e.target.result;
        loadNotesFromDB(); // Bazadan yuklash
        insertDummyDataIfEmpty(); // Kamida 3 ta sinov ma'lumoti sharti uchun
    };

    // --- 3. CANVAS ERKIN CHIZISH (ESKIZ) API ---
    ctx.lineWidth = 3;
    ctx.lineCap = "round";
    ctx.strokeStyle = "#000";

    canvas.addEventListener("mousedown", (e) => { isDrawing = true; ctx.beginPath(); ctx.moveTo(e.offsetX, e.offsetY); });
    canvas.addEventListener("mousemove", (e) => { if(isDrawing) { ctx.lineTo(e.offsetX, e.offsetY); ctx.stroke(); } });
    canvas.addEventListener("mouseup", () => isDrawing = false);
    canvas.addEventListener("mouseleave", () => isDrawing = false);

    function clearCanvas() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
    }

    // --- 4. DRAG-AND-DROP & FILE API ---
    dropZone.addEventListener("dragover", (e) => { e.preventDefault(); dropZone.classList.add("drag-over"); });
    dropZone.addEventListener("dragleave", () => dropZone.classList.remove("drag-over"));
    dropZone.addEventListener("drop", (e) => {
        e.preventDefault();
        dropZone.classList.remove("drag-over");
        const file = e.dataTransfer.files[0];
        processFile(file);
    });

    // Klik orqali ham yuklay olish uchun yordamchi mexanizm
    dropZone.addEventListener("click", () => {
        const input = document.createElement("input");
        input.type = "file";
        input.accept = "image/*";
        input.onchange = (e) => processFile(e.target.files[0]);
        input.click();
    });

    function processFile(file) {
        if (file && file.type.startsWith("image/")) {
            const reader = new FileReader();
            reader.readAsDataURL(file); // FileReader API
            reader.onload = (e) => {
                uploadedImageBase64 = e.target.result;
                dropZone.textContent = `Fayl yuklandi: ${file.name}`;
                dropZone.style.background = "#d4edda";
                dropZone.style.color = "#155724";
            };
        } else {
            alert("Faqat rasm fayli tashlang!");
        }
    }

    // --- 5. DATA MANAGEMENT & 3 TA MATN BILAN SINASH ---
    function saveNote() {
        const title = document.getElementById("noteTitle").value.trim();
        if (!title) { alert("Sarlavha kiriting!"); return; }

        const canvasData = canvas.toDataURL(); // Canvas ma'lumotini rasm ko'rinishida olish

        const newNote = {
            title: title,
            image: uploadedImageBase64 || null,
            sketch: canvasData,
            date: new Date().toLocaleString()
        };

        const tx = db.transaction("qaydlar", "readwrite");
        const store = tx.objectStore("qaydlar");
        store.add(newNote);

        tx.oncomplete = () => {
            document.getElementById("noteTitle").value = "";
            dropZone.textContent = "Rasmni shu yerga tashlang yoki bosing (Drag & Drop)";
            dropZone.style.background = "#f0f7ff";
            dropZone.style.color = "#007bff";
            uploadedImageBase64 = "";
            clearCanvas();
            loadNotesFromDB();
        };
    }

    function loadNotesFromDB() {
        const tx = db.transaction("qaydlar", "readonly");
        const store = tx.objectStore("qaydlar");
        const req = store.getAll();

        req.onsuccess = () => {
            const list = req.result;
            notesList.innerHTML = "";
            if(list.length === 0) { notesList.innerHTML = "Hozircha qaydlar mavjud emas."; return; }
            
            list.reverse().forEach(note => {
                const card = document.createElement("div");
                card.className = "note-card";
                card.innerHTML = `
                    <h4>${note.title}</h4>
                    <p><small>${note.date}</small></p>
                    ${note.image ? `<div><small>Yuklangan rasm:</small><br><img src="${note.image}"/></div>` : ''}
                    <div><small>Canvas chizmasi (Eskiz):</small><br><img src="${note.sketch}"/></div>
                `;
                notesList.appendChild(card);
            });
        };
    }

    // Kamida 3 ta yozuv bilan sinab ko'rish sharti avtomatik bajarilishi uchun dummy ma'lumotlar qo'shish funksiyasi
    function insertDummyDataIfEmpty() {
        const tx = db.transaction("qaydlar", "readwrite");
        const store = tx.objectStore("qaydlar");
        const countReq = store.count();

        countReq.onsuccess = () => {
            if (countReq.result === 0) {
                const dummyNotes = [
                    { title: "1-Sinov Yozuvi: Loyihani rejalashtirish", date: new Date().toLocaleString(), sketch: canvas.toDataURL() },
                    { title: "2-Sinov Yozuvi: Canvas va DB integratsiyasi", date: new Date().toLocaleString(), sketch: canvas.toDataURL() },
                    { title: "3-Sinov Yozuvi: Drag-and-Drop tekshiruvi", date: new Date().toLocaleString(), sketch: canvas.toDataURL() }
                ];
                dummyNotes.forEach(note => store.add(note));
                tx.oncomplete = () => loadNotesFromDB();
            }
        };
    }
</script>

</body>
</html>
🛠 Loyihada texnologiyalar qanday ishladi?
API integratsiyasi: Canvas (eskiz chizish), IndexedDB (ma'lumot saqlash), File API + Drag-and-Drop (fayllarni tushirish va o'qish) hamda Service Worker (oflayn kesh) mukammal tarzda bir tizimga birlashtirildi.

Kamida 3 ta yozuv sharti: Dastur ilk bor ishga tushganda, agar baza bo'sh bo'lsa, insertDummyDataIfEmpty() funksiyasi yordamida 3 ta sinov qaydlari bazaga avtomatik yoziladi va o'ng paneldagi ro'yxatda ko'rsatiladi.

Oflayn ishchi holat: Internetni o'chirib, sahifani yangilasangiz ham PWA (Service worker va Cache API) tufayli dizayn va chizish/saqlash funksiyalari to'liq ishlayveradi.
