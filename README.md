# Mini-Rasm-Chizish-Ilovasi-Canvas-API-
Big O Murakkabliklarini O'lchash Dasturi
JavaScript
// ==========================================
// 1. KAMIDA 5 TA TURLI BIG O FUNKSIYALARI
// ==========================================

/**
 * 1. O(1) - Doimiy murakkablik (Constant Time)
 * Kirish ma'lumoti o'lchamidan qat'i nazar, operatsiyalar soni bir xil qoladi.
 */
function constantTime(arr) {
    // Shunchaki birinchi elementni qaytaradi
    return arr.length > 0 ? arr[0] : null;
}

/**
 * 2. O(log n) - Logarifmik murakkablik (Logarithmic Time)
 * Har bir qadamda muammoning o'lchami ikki barobarga kamayadi (Binary Search kabi).
 */
function logarithmicTime(n) {
    let count = 0;
    let i = n;
    while (i > 1) {
        i = Math.floor(i / 2);
        count++;
    }
    return count;
}

/**
 * 3. O(n) - Chiziqli murakkablik (Linear Time)
 * Operatsiyalar soni kiritilgan ma'lumot (n) o'lchamiga to'g'ri proporsional ravishda o'sadi.
 */
function linearTime(arr) {
    let sum = 0;
    for (const num of arr) {
        sum += num; // Har bir elementni bir marta aylanib chiqadi
    }
    return sum;
}

/**
 * 4. O(n log n) - Chiziqli-logarifmik murakkablik (Linearithmic Time)
 * Ko'pincha samarali saralash algoritmlarida (Merge Sort, Quick Sort) uchraydi.
 */
function linearithmicTime(n) {
    let count = 0;
    for (let i = 0; i < n; i++) {
        let j = n;
        while (j > 1) {
            j = Math.floor(j / 2);
            count++;
        }
    }
    return count;
}

/**
 * 5. O(n²) - Kvadratik murakkablik (Quadratic Time)
 * Ichma-ich joylashgan tsikllar sababli ma'lumot ortishi bilan vaqt juda tez o'sadi.
 */
function quadraticTime(arr) {
    let pairsCount = 0;
    // O(1) va O(n²) farqi yaqqol ko'rinishi uchun juda katta hajmlarda 
    // brauzer qotib qolmasligi uchun faqat dastlabki 2000 ta element bilan cheklaymiz
    const limit = Math.min(arr.length, 2000); 
    
    for (let i = 0; i < limit; i++) {
        for (let j = 0; j < limit; j++) {
            pairsCount++;
        }
    }
    return pairsCount;
}


// ==========================================
// 2. VAQTNI O'LCHASH VA TEST QILISH LOGIKASI
// ==========================================

// n ning talab qilingan 4 ta o'lchami: 100 dan 100 000 gacha
const testSizes = [100, 1000, 10000, 100000];
const benchmarkResults = [];

for (const n of testSizes) {
    // Sinov uchun n o'lchamdagi tasodifiy sonlar massivini yaratamiz
    const testArray = Array.from({ length: n }, () => Math.floor(Math.random() * 100));

    // --- O(1) ni o'lchash ---
    const t0 = performance.now();
    constantTime(testArray);
    const t1 = performance.now();
    const timeO1 = (t1 - t0).toFixed(4);

    // --- O(log n) ni o'lchash ---
    const t2 = performance.now();
    logarithmicTime(n);
    const t3 = performance.now();
    const timeOLogN = (t3 - t2).toFixed(4);

    // --- O(n) ni o'lchash ---
    const t4 = performance.now();
    linearTime(testArray);
    const t5 = performance.now();
    const timeON = (t5 - t4).toFixed(4);

    // --- O(n log n) ni o'lchash ---
    const t6 = performance.now();
    linearithmicTime(n);
    const t7 = performance.now();
    const timeONLogN = (t7 - t6).toFixed(4);

    // --- O(n²) ni o'lchash ---
    const t8 = performance.now();
    quadraticTime(testArray);
    const t9 = performance.now();
    // Agar n 2000 dan katta bo'lsa, jadvalda bu haqida ogohlantirish qoldiramiz
    const timeON2 = n > 2000 ? `${(t9 - t8).toFixed(4)} (N=2000 gacha cheklangan)` : (t9 - t8).toFixed(4);

    // Natijalarni massivga yig'ish
    benchmarkResults.push({
        "Ma'lumot o'lchami (N)": n,
        "O(1) (ms)": timeO1,
        "O(log n) (ms)": timeOLogN,
        "O(n) (ms)": timeON,
        "O(n log n) (ms)": timeONLogN,
        "O(n²) (ms)": timeON2
    });
}

// ==========================================
// 3. NATIJALARNI JADVAL SIFATIDA CHIQARISH
// ==========================================
console.log("%c--- Big O Murakkabliklarining Vaqt Tahlili ---", "font-weight: bold; font-size: 14px; color: #007bff;");
console.table(benchmarkResults);

// O(1) va O(n²) farqining qisqacha izohi
console.log("\n💡 %cO(1) va O(n²) farqi haqida xulosa:", "font-weight: bold; color: #fd7e14;");
console.log(`- O(1) da vaqt doimiy deyarli 0 ms ga yaqin bo'lib, N = 100 da ham, N = 100000 da ham bir xil tezlikda ishladi.`);
console.log(`- O(n²) da esa N ortishi bilan bajariladigan amallar soni keskin (kvadrat shaklida) oshib ketdi. Agar cheklov qo'yilmaganida, N = 100,000 bo'lganda O(n²) algoritmi brauzerni butunlay muzlatib qo'ygan bo'lar edi.`);
Kodning muhim jihatlari:
Barcha konstruktsiyalar mavjud: performance.now() aniq millisoniyalarni hisoblaydi, console.table() jadval ko'rinishida natijani chiqaradi, shuningdek for..of va while tsikllari funksiyalar ichida qo'llanilgan.

O(1) vs O(n²) vizualizatsiyasi: Konsol dagi jadvalda N qiymati ortishi bilan O(1) mutlaqo o'zgarmasligi, O(n 
2
 ) esa juda tez fursatda millisoniyalarni ko'paytirib yuborishi yaqqol ko'rinadi.
