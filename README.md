# Mini-Rasm-Chizish-Ilovasi-Canvas-API-Mana, barcha talablaringizga to'liq javob beradigan, sodda va chiroyli dizaynga ega bo'lgan rasm chizish dasturi (Canvas Paint).

Ushbu kod bitta HTML faylida ishlaydi. Unda erkin chizish, to'g'ri chiziq, doira va to'rtburchak chizish imkoniyatlari mavjud bo'lib, rang va qalinlikni dinamik ravishda o'zgartirish hamda chizilgan rasmni PNG formatida yuklab olish funksiyalari qo'shilgan.

index.html fayli kodi:
HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HTML5 Canvas Rasm Chizish</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #f0f2f5;
            margin: 0;
            padding: 20px;
        }
        h1 {
            color: #333;
            margin-bottom: 10px;
        }
        .toolbar {
            display: flex;
            gap: 15px;
            background: white;
            padding: 10px 20px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            margin-bottom: 15px;
            align-items: center;
            flex-wrap: wrap;
        }
        .tool-group {
            display: flex;
            align-items: center;
            gap: 8px;
        }
        canvas {
            background: white;
            border-radius: 8px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.15);
            cursor: crosshair;
        }
        button {
            padding: 8px 15px;
            border: none;
            border-radius: 5px;
            background-color: #007bff;
            color: white;
            font-weight: bold;
            cursor: pointer;
            transition: background 0.2s;
        }
        button:hover {
            background-color: #0056b3;
        }
        select, input[type="color"], input[type="range"] {
            cursor: pointer;
        }
    </style>
</head>
<body>

    <h1>Erkin Chizish Doskasi</h1>

    <div class="toolbar">
        <div class="tool-group">
            <label for="colorPicker">Rang:</label>
            <input type="color" id="colorPicker" value="#000000">
        </div>

        <div class="tool-group">
            <label for="lineWidth">Qalinlik (px):</label>
            <input type="range" id="lineWidth" min="1" max="50" value="5">
            <span id="widthVal">5</span>
        </div>

        <div class="tool-group">
            <label for="toolSelect">Asbob:</label>
            <select id="toolSelect">
                <option value="brush">Erkin chizish (Brush)</option>
                <option value="line">To'g'ri chiziq (Line)</option>
                <option value="rect">To'rtburchak (Rect)</option>
                <option value="circle">Doira (Circle)</option>
            </select>
        </div>

        <button id="clearBtn" style="background-color: #dc3545;">Tozalash</button>
        <button id="downloadBtn" style="background-color: #28a745;">PNG Yuklash</button>
    </div>

    <canvas id="paintCanvas" width="800" height="500"></canvas>

    <script>
        const canvas = document.getElementById("paintCanvas");
        const ctx = canvas.getContext("2d");

        // Elementlarni olish
        const colorPicker = document.getElementById("colorPicker");
        const lineWidthInput = document.getElementById("lineWidth");
        const widthVal = document.getElementById("widthVal");
        const toolSelect = document.getElementById("toolSelect");
        const clearBtn = document.getElementById("clearBtn");
        const downloadBtn = document.getElementById("downloadBtn");

        // Boshlang'ich holat o'zgaruvchilari
        let isDrawing = false;
        let startX = 0;
        let startY = 0;
        let snapshot; // Shakllarni chizishda eski holatni saqlab turish uchun

        // Chiziq uchi yumaloq bo'lishi uchun
        ctx.lineCap = "round";
        ctx.lineJoin = "round";

        // Sichqoncha bosilganda
        function startDraw(e) {
            isDrawing = true;
            startX = e.offsetX;
            startY = e.offsetY;

            // Har yangi shakl yoki chiziq oldidan beginPath() chaqiriladi
            ctx.beginPath();
            
            // Chiziq sozlamalarini o'rnatish
            ctx.strokeStyle = colorPicker.value;
            ctx.fillStyle = colorPicker.value;
            ctx.lineWidth = lineWidthInput.value;

            if (toolSelect.value === "brush") {
                // Erkin chizish boshlanishi
                ctx.moveTo(startX, startY);
                ctx.lineTo(startX, startY);
                ctx.stroke();
            } else {
                // To'rtburchak/Doira chizayotganda oldingi holatni rasm sifatida saqlab turish (harakat davomida ortiqcha iz qolmasligi uchun)
                snapshot = ctx.getImageData(0, 0, canvas.width, canvas.height);
            }
        }

        // Sichqoncha harakatlanganda
        function drawing(e) {
            if (!isDrawing) return;

            const currentX = e.offsetX;
            const currentY = e.offsetY;

            if (toolSelect.value === "brush") {
                // Erkin chizishda sichqoncha yo'li bo'ylab chizish
                ctx.lineTo(currentX, currentY);
                ctx.stroke();
            } else {
                // Shakllarni chizayotganda ekranni qayta tiklab turamiz
                ctx.putImageData(snapshot, 0, 0);

                if (toolSelect.value === "line") {
                    ctx.beginPath(); // Yangi yo'l
                    ctx.moveTo(startX, startY);
                    ctx.lineTo(currentX, currentY);
                    ctx.stroke();
                } 
                else if (toolSelect.value === "rect") {
                    ctx.beginPath(); // Yangi yo'l
                    const width = currentX - startX;
                    const height = currentY - startY;
                    ctx.fillRect(startX, startY, width, height); // To'ldirilgan to'rtburchak
                } 
                else if (toolSelect.value === "circle") {
                    ctx.beginPath(); // Yangi yo'l
                    // Radiusni hisoblash (Pifagor teoremasi bo'yicha masofa)
                    const radius = Math.sqrt(Math.pow(currentX - startX, 2) + Math.pow(currentY - startY, 2));
                    ctx.arc(startX, startY, radius, 0, Math.PI * 2);
                    ctx.fill(); // Ichini bo'yash
                }
            }
        }

        // Sichqoncha qo'yib yuborilganda
        function stopDraw() {
            isDrawing = false;
        }

        // Hodisalarni tinglash (Mouse Events)
        canvas.addEventListener("mousedown", startDraw);
        canvas.addEventListener("mousemove", drawing);
        canvas.addEventListener("mouseup", stopDraw);
        canvas.addEventListener("mouseleave", stopDraw);

        // Qalinlik o'zgarganda matnini ham yangilash
        lineWidthInput.addEventListener("input", () => {
            widthVal.textContent = lineWidthInput.value;
        });

        // Tozalash tugmasi
        clearBtn.addEventListener("click", () => {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        });

        // PNG formatida yuklab olish
        downloadBtn.addEventListener("click", () => {
            // Orqa fon shaffof bo'lib qolmasligi uchun (ixtiyoriy, lekin oq fon bergan yaxshi)
            const tempCanvas = document.createElement("canvas");
            tempCanvas.width = canvas.width;
            tempCanvas.height = canvas.height;
            const tempCtx = tempCanvas.getContext("2d");
            
            // Oq fon chizish
            tempCtx.fillStyle = "#ffffff";
            tempCtx.fillRect(0, 0, tempCanvas.width, tempCanvas.height);
            // Asl canvasdagi rasmni vaqtincha canvasga o'tkazish
            tempCtx.drawImage(canvas, 0, 0);

            // PNG formatga o'tkazish va yuklash
            const link = document.createElement("a");
            link.download = "canvas_rasm.png";
            link.href = tempCanvas.toDataURL("image/png");
            link.click();
        });
    </script>
</body>
</html>
Kodda bajarilgan shartlar tahlili:
canvas.getContext("2d"): Canvas konteksti to'g'ri olindi va barcha amallar shu orqali bajarildi.

Erkin chizish: Sichqoncha hodisalari (mousedown, mousemove, mouseup) orqali brush rejimi to'liq ishlaydi.

Dinamik Sozlamalar: ctx.strokeStyle va ctx.lineWidth input elementlariga (colorPicker, lineWidth) bog'langan.

Talab qilingan barcha metodlar ishlatildi:

beginPath(): Har bir yangi chiziq yoki shakl boshlanishidan oldin chaqiriladi.

moveTo() & lineTo(): Erkin chizish va to'g'ri chiziq chizish uchun qo'llanildi.

stroke(): Chiziqlarni ekranga chiqarish uchun ishlatildi.

fillRect(): To'rtburchak chizish funksiyasida ishlatildi.

arc() & fill(): Doira chizish va uning ichini rang bilan to'ldirish uchun ishlatildi.

PNG yuklab olish: ctx.toDataURL("image/png") yordamida chizilgan rasm oq fon bilan sifatli PNG shaklida yuklab olinadi.
