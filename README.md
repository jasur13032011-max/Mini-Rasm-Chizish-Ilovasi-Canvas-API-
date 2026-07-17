# Mini-Rasm-Chizish-Ilovasi-Canvas-API-
JavaScript Kod:
JavaScript
// ==========================================
// 1. NODE VA LINKEDLIST KLASSlari
// ==========================================

// Har bir qo'shiq (Node) obyekti
class Node {
    constructor(data) {
        this.data = data; // Qo'shiq nomi
        this.next = null; // Keyingi qo'shiqqa ko'rsatkich
        this.prev = null; // Oldingi qo'shiqqa ko'rsatkich (DLL uchun)
    }
}

// Pleylist (Doubly Linked List) klassi
class LinkedList {
    constructor() {
        this.head = null;         // Ro'yxat boshi
        this.tail = null;         // Ro'yxat oxiri
        this.current = null;      // Hozirgi ijro etilayotgan qo'shiq
    }

    // A. Pleylist oxiriga qo'shiq qo'shish
    append(songName) {
        const newNode = new Node(songName);
        if (!this.head) {
            this.head = newNode;
            this.tail = newNode;
        } else {
            this.tail.next = newNode;
            newNode.prev = this.tail;
            this.tail = newNode;
        }
        console.log(`🎵 "${songName}" pleylist oxiriga qo'shildi.`);
    }

    // B. Pleylist boshiga qo'shiq qo'shish
    prepend(songName) {
        const newNode = new Node(songName);
        if (!this.head) {
            this.head = newNode;
            this.tail = newNode;
        } else {
            newNode.next = this.head;
            this.head.prev = newNode;
            this.head = newNode;
        }
        console.log(`⏮ "${songName}" pleylist boshiga qo'shildi.`);
    }

    // C. Qo'shiqni nomi bo'yicha o'chirish
    remove(songName) {
        if (!this.head) {
            console.error("❌ Xato: Pleylist bo'sh, o'chirish imkonsiz!");
            return;
        }

        let curr = this.head;
        while (curr) {
            if (curr.data === songName) {
                // Agar o'chirilayotgan qo'shiq joriy ijro etilayotgan bo'lsa, uni keyingisiga suramiz
                if (this.current === curr) {
                    this.current = curr.next || curr.prev;
                }

                if (curr === this.head) {
                    this.head = curr.next;
                    if (this.head) this.head.prev = null;
                } else if (curr === this.tail) {
                    this.tail = curr.prev;
                    if (this.tail) this.tail.next = null;
                } else {
                    curr.prev.next = curr.next;
                    curr.next.prev = curr.prev;
                }

                console.log(`🗑 "${songName}" pleylistdan o'chirildi.`);
                return;
            }
            curr = curr.next;
        }
        console.warn(`⚠️ Ogohlantirish: "${songName}" topilmadi.`);
    }

    // D. Pleylistni konsolga chiqarish
    display() {
        if (!this.head) {
            console.log("Empty Playlist: []");
            return;
        }
        let curr = this.head;
        let listStr = "PlayList: ";
        while (curr) {
            listStr += `[${curr.data}] <-> `;
            curr = curr.next;
        }
        console.log(listStr + "NULL");
    }

    // E. Hozirgi ijro etilayotgan qo'shiqni ko'rish
    currentSong() {
        if (!this.head) {
            console.error("❌ Xato: Pleylist bo'sh! Hech qanday qo'shiq ijro etilmayapti.");
            return null;
        }
        if (!this.current) {
            this.current = this.head; // Agar hali boshlanmagan bo'lsa, birinchisini qo'yamiz
        }
        console.log(`▶️ Hozir ijro etilmoqda: "${this.current.data}"`);
        return this.current.data;
    }

    // F. Keyingi qo'shiqqa o'tish
    next() {
        if (!this.head) {
            console.error("❌ Xato: Pleylist bo'sh!");
            return;
        }
        if (!this.current) this.current = this.head;

        if (this.current.next) {
            this.current = this.current.next;
            console.log(`⏭ Keyingi qo'shiqqa o'tildi: "${this.current.data}"`);
        } else {
            console.log("🔁 Pleylist tugadi. Oxirgi qo'shiqdasi.");
        }
    }

    // G. Oldingi qo'shiqqa o'tish (DLL yordamida)
    prev() {
        if (!this.head) {
            console.error("❌ Xato: Pleylist bo'sh!");
            return;
        }
        if (!this.current) this.current = this.head;

        if (this.current.prev) {
            this.current = this.current.prev;
            console.log(`⏮ Oldingi qo'shiqqa o'tildi: "${this.current.data}"`);
        } else {
            console.log("🔕 Bu birinchi qo'shiq, undan oldin hech narsa yo'q.");
        }
    }
}


// ==========================================
// 2. SINAB KO'RISH (DEMO RUN)
// ==========================================

console.log("--- 1-TEST: Bo'sh pleylist holati ---");
const myPlaylist = new LinkedList();
myPlaylist.currentSong(); // Bo'sh bo'lgani uchun xato chiqaradi
myPlaylist.next();        // Xato chiqaradi

console.log("\n--- 2-TEST: Kamida 5 ta qo'shiq qo'shish ---");
myPlaylist.append("Qo'shiq 1: Uzb Rap");
myPlaylist.append("Qo'shiq 2: Lo-Fi Chill");
myPlaylist.append("Qo'shiq 3: Classical Piano");
myPlaylist.append("Qo'shiq 4: Pop Energy");
myPlaylist.prepend("Qo'shiq 0: Boshlang'ich Intro"); // prepend testi

myPlaylist.display(); // Pleylist tarkibini ko'rish

console.log("\n--- 3-TEST: Navigatsiya (Current, Next, Prev) ---");
myPlaylist.currentSong(); // Qo'shiq 0 ni chiqarishi kerak
myPlaylist.next();        // Qo'shiq 1 ga o'tadi
myPlaylist.next();        // Qo'shiq 2 ga o'tadi
myPlaylist.prev();        // Orqaga qaytadi: Qo'shiq 1 ga

console.log("\n--- 4-TEST: O'chirish (Remove) ---");
myPlaylist.remove("Qo'shiq 2: Lo-Fi Chill");
myPlaylist.display();
Kod qanday talablarni bajardi?
DLL ishlatilishi: Node klassiga this.prev xossasi qo'shildi. Bu prev() metodi chaqirilganda har bir qo'shiqdan orqaga osongina o'tishga imkon beradi.

Xatolar nazorati: Agar myPlaylist obyekti bo'sh bo'lsa (this.head === null), currentSong(), next(), prev() va remove() metodlari console.error orqali to'g'ri ogohlantirish beradi.

Metodlar to'liqligi: append, prepend, remove, display
