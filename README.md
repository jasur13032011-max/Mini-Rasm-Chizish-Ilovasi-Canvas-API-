# Mini-Rasm-Chizish-Ilovasi-Canvas-API-

Bu kodni bitta HTML faylida ishlatishingiz mumkin:

HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>IndexedDB Do'kon Tizimi</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f6f9;
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 600px;
            margin: 0 auto;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        }
        h2 { text-align: center; color: #333; }
        .form-group {
            margin-bottom: 12px;
        }
        label { display: block; margin-bottom: 5px; font-weight: bold; }
        input[type="text"], input[type="number"] {
            width: 100%;
            padding: 8px;
            box-sizing: border-box;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        .btn-group {
            display: flex;
            gap: 10px;
            margin-top: 15px;
            flex-wrap: wrap;
        }
        button {
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            color: white;
        }
        .btn-add { background-color: #28a745; }
        .btn-get { background-color: #17a2b8; }
        .btn-update { background-color: #ffc107; color: #333; }
        .btn-delete { background-color: #dc3545; }
        .btn-list { background-color: #6c757d; width: 100%; margin-top: 10px; }
        .output {
            margin-top: 20px;
            padding: 10px;
            background: #e9ecef;
            border-radius: 4px;
            min-height: 50px;
            white-space: pre-wrap;
            font-family: monospace;
        }
    </style>
</head>
<body>

<div class="container">
    <h2>Do'kon Maxsulotlari (IndexedDB)</h2>
    
    <div class="form-group">
        <label for="prodId">Mahsulot IDsi (Olish, Yangilash va O'chirish uchun kerak):</label>
        <input type="number" id="prodId" placeholder="ID raqami">
    </div>
    <div class="form-group">
        <label for="prodName">Mahsulot Nomi:</label>
        <input type="text" id="prodName" placeholder="Masalan: Olma">
    </div>
    <div class="form-group">
        <label for="prodPrice">Narxi (so'm):</label>
        <input type="number" id="prodPrice" placeholder="Masalan: 15000">
    </div>

    <div class="btn-group">
        <button class="btn-add" onclick="addProduct()">Qo'shish (add)</button>
        <button class="btn-get" onclick="getProduct()">ID bo'yicha olish (get)</button>
        <button class="btn-update" onclick="updateProduct()">Yangilash (put)</button>
        <button class="btn-delete" onclick="deleteProduct()">O'chirish (delete)</button>
    </div>

    <button class="btn-list" onclick="listAllProducts()">Barcha mahsulotlarni chiqarish (Cursor)</button>

    <h3>Natija paneli:</h3>
    <div id="output" class="output">Ma'lumotlar bu yerda ko'rinadi...</div>
</div>

<script>
    let db;
    const dbName = "DokonDB";
    const dbVersion = 1;
    const storeName = "mahsulotlar";

    const outputDiv = document.getElementById("output");

    // Natijalarni ekranga chiqarish funksiyasi
    function printLog(message, isError = false) {
        outputDiv.style.color = isError ? "red" : "black";
        outputDiv.textContent = message;
    }

    // 1. IndexedDB ni ochish (indexedDB.open)
    const request = indexedDB.open(dbName, dbVersion);

    // Xatolik yuz berganda
    request.onerror = function(event) {
        printLog("Ma'lumotlar bazasini ochishda xatolik: " + event.target.errorCode, true);
    };

    // Muvaffaqiyatli ochilganda
    request.onsuccess = function(event) {
        db = event.target.result;
        printLog("Ma'lumotlar bazasiga muvaffaqiyatli ulanildi.");
    };

    // 2. Struktura yaratish yoki o'zgarish sodir bo'lganda (onupgradeneeded)
    request.onupgradeneeded = function(event) {
        const dbInstance = event.target.result;
        
        // createObjectStore() chaqirish
        if (!dbInstance.objectStoreNames.contains(storeName)) {
            const store = dbInstance.createObjectStore(storeName, { keyPath: "id", autoIncrement: true });
            
            // createIndex() chaqirish (Mahsulot nomi bo'yicha qidiruv indeksi)
            store.createIndex("nom", "nom", { unique: false });
            printLog("Omborcha va indekslar muvaffaqiyatli yaratildi.");
        }
    };

    // --- TRANSAKSIYALAR VA METODLAR ---

    // A. Mahsulot qo'shish (add) - readwrite transaksiya
    function addProduct() {
        const nom = document.getElementById("prodName").value;
        const narx = parseFloat(document.getElementById("prodPrice").value);

        if (!nom || isNaN(narx)) {
            printLog("Iltimos, mahsulot nomi va narxini to'g'ri kiriting!", true);
            return;
        }

        const transaction = db.transaction([storeName], "readwrite");
        const store = transaction.objectStore(storeName);
        
        const newProduct = { nom: nom, narx: narx };
        const req = store.add(newProduct); // add() metodu

        req.onsuccess = function(e) {
            printLog(`Mahsulot qo'shildi! ID raqami: ${e.target.result}`);
            clearInputs();
        };

        req.onerror = function(e) {
            printLog("Xatolik yuz berdi: " + e.target.error, true);
        };
    }

    // B. Mahsulotni ID bo'yicha olish (get) - readonly transaksiya
    function getProduct() {
        const id = parseInt(document.getElementById("prodId").value);
        if (isNaN(id)) {
            printLog("Iltimos, olish uchun mahsulot ID raqamini kiriting!", true);
            return;
        }

        const transaction = db.transaction([storeName], "readonly");
        const store = transaction.objectStore(storeName);
        
        const req = store.get(id); // get() metodu

        req.onsuccess = function() {
            if (req.result) {
                printLog(`Topilgan mahsulot:\nID: ${req.result.id}\nNomi: ${req.result.nom}\nNarxi: ${req.result.narx} so'm`);
                // Ma'lumotlarni inputga joylashtirish (tahrirlash uchun qulaylik)
                document.getElementById("prodName").value = req.result.nom;
                document.getElementById("prodPrice").value = req.result.narx;
            } else {
                printLog(`ID: ${id} bo'lgan mahsulot topilmadi.`, true);
            }
        };

        req.onerror = function() {
            printLog("Mahsulotni olishda xatolik yuz berdi.", true);
        };
    }

    // C. Mahsulotni yangilash (put) - readwrite transaksiya
    function updateProduct() {
        const id = parseInt(document.getElementById("prodId").value);
        const nom = document.getElementById("prodName").value;
        const narx = parseFloat(document.getElementById("prodPrice").value);

        if (isNaN(id) || !nom || isNaN(narx)) {
            printLog("Iltimos, yangilash uchun ID, yangi nom va narxni kiriting!", true);
            return;
        }

        const transaction = db.transaction([storeName], "readwrite");
        const store = transaction.objectStore(storeName);

        const updatedProduct = { id: id, nom: nom, narx: narx };
        const req = store.put(updatedProduct); // put() metodu

        req.onsuccess = function() {
            printLog(`ID: ${id} bo'lgan mahsulot muvaffaqiyatli yangilandi.`);
            clearInputs();
        };

        req.onerror = function(e) {
            printLog("Yangilashda xatolik yuz berdi: " + e.target.error, true);
        };
    }

    // D. Mahsulotni o'chirish (delete) - readwrite transaksiya
    function deleteProduct() {
        const id = parseInt(document.getElementById("prodId").value);
        if (isNaN(id)) {
            printLog("Iltimos, o'chirish uchun ID raqamini kiriting!", true);
            return;
        }

        const transaction = db.transaction([storeName], "readwrite");
        const store = transaction.objectStore(storeName);

        const req = store.delete(id); // delete() metodu

        req.onsuccess = function() {
            printLog(`ID: ${id} bo'lgan mahsulot ma'lumotlar bazasidan o'chirildi.`);
            clearInputs();
        };

        req.onerror = function() {
            printLog("O'chirishda xatolik yuz berdi.", true);
        };
    }

    // E. IDBCursor bilan barcha yozuvlarni aylanib chiqish - readonly transaksiya
    function listAllProducts() {
        const transaction = db.transaction([storeName], "readonly");
        const store = transaction.objectStore(storeName);
        
        let resultText = "Barcha mahsulotlar ro'yxati:\n-------------------------\n";
        let count = 0;

        // openCursor() chaqirilishi
        const req = store.openCursor(); 

        req.onsuccess = function(event) {
            const cursor = event.target.result;
            if (cursor) {
                const item = cursor.value;
                resultText += `ID: ${item.id} | Nomi: ${item.nom} | Narxi: ${item.narx} so'm\n`;
                count++;
                cursor.continue(); // Keyingi yozuvga o'tish
            } else {
                // Agar cursor null bo'lsa, demak barcha yozuvlar aylanib chiqildi
                if (count === 0) {
                    printLog("Bazada hech qanday mahsulot mavjud emas.");
                } else {
                    printLog(resultText);
                }
            }
        };

        req.onerror = function() {
            printLog("Cursor orqali o'qishda xatolik yuz berdi.", true);
        };
    }

    // Kiritish maydonlarini tozalash
    function clearInputs() {
        document.getElementById("prodId").value = "";
        document.getElementById("prodName").value = "";
        document.getElementById("prodPrice").value = "";
    }
</script>

</body>
</html>
Kodda bajarilgan talablar tahlili:
indexedDB.open("DokonDB", 1): Ma'lumotlar bazasi nomi va versiyasi to'g'ri belgilandi va ishga tushirildi.

onupgradeneeded:

createObjectStore(): id kalit maydoni va uning avtomatik oshib borishi (autoIncrement: true) bilan birga omborcha yaratildi.

createIndex(): Qidiruv ishlarini osonlashtirish uchun mahsulot nomi bo'yicha indeks yaratildi (store.createIndex("nom", "nom")).

Metodlar:

add(): Yangi mahsulotlarni qo'shadi.

get(): Kiritilgan ID bo'yicha mahsulot obyekti olinadi.

put(): Mavjud ma'lumotni kiritilgan ID bo'yicha qayta yozadi/yangilaydi.

delete(): ID bo'yicha kerakli qatorni o'chirib yuboradi.

IDBCursor: openCursor() hamda cursor.continue() yordamida bazadagi barcha yozuvlar birma-bir aylanib chiqilib, konsol va natija paneliga chiqariladi.

Transaksiyalar:

Ma'lumotni faqat o'qishda (get, openCursor) readonly rejimi to'g'ri ishlatildi.

Ma'lumot qo'shish, o'zgartirish va o'chirishda (add, put, delete) readwrite rejimi to'g'ri belgilandi.

Xatolar nazorati: Har bir so'rovda muvaffaqiyatli bajarilganlik holati (onsuccess) va yuzaga kelgan xatoliklar (onerror) to'liq ushlab olinib, foydalanuvchiga xabar beriladi.
