# Mini-Rasm-Chizish-Ilovasi-Canvas-API-
Python
# 1. 10+ mahsulotdan iborat dict ro'yxati
mahsulotlar = [
    {"nom": "Noutbuk", "narx": 12000000, "soni": 5, "kategoriya": "Elektronika"},
    {"nom": "Telefon", "narx": 6000000, "soni": 12, "kategoriya": "Elektronika"},
    {"nom": "Smart Soat", "narx": 1500000, "soni": 8, "kategoriya": "Elektronika"},
    {"nom": "Klaviatura", "narx": 450000, "soni": 20, "kategoriya": "Aksessuarlar"},
    {"nom": "Sichqoncha", "narx": 250000, "soni": 25, "kategoriya": "Aksessuarlar"},
    {"nom": "Stul", "narx": 1200000, "soni": 6, "kategoriya": "Mebel"},
    {"nom": "Stol", "narx": 1500000, "soni": 4, "kategoriya": "Mebel"},
    {"nom": "Monitor", "narx": 3500000, "soni": 7, "kategoriya": "Elektronika"},
    {"nom": "Quloqchin", "narx": 800000, "soni": 15, "kategoriya": "Aksessuarlar"},
    {"nom": "Chiroq", "narx": 150000, "soni": 30, "kategoriya": "Uy-ro'zg'or"},
    {"nom": "Gilam", "narx": 2500000, "soni": 3, "kategoriya": "Uy-ro'zg'or"},
]

# Yordamchi funksiya: Mahsulotlarni f-string formatda chiroyli chiqarish
def mahsulotlarni_chiqar(royxat, sarlavha):
    print(f"\n=== {sarlavha} ===")
    print(f"{'Nom':<15} | {'Narx (so`m)':<12} | {'Soni':<6} | {'Kategoriya':<12}")
    print("-" * 55)
    for p in royxat:
        print(f"{p['nom']:<15} | {p['narx']:<12,} | {p['soni']:<6} | {p['kategoriya']:<12}")

# --- 2. Narx bo'yicha saralash (Oshuvchi va Kamayuvchi) ---
oshuvchi = sorted(mahsulotlar, key=lambda p: p['narx'])
mahsulotlarni_chiqar(oshuvchi, "Narxlar oshib borish tartibida")

kamayuvchi = sorted(mahsulotlar, key=lambda p: p['narx'], reverse=True)
mahsulotlarni_chiqar(kamayuvchi, "Narxlar kamayib borish tartibida")

# --- 3. Bir nechta mezon bilan saralash ---
# Narxi bo'yicha kamayuvchi (oldinga minus qo'yilgan), narxi teng bo'lsa nomi bo'yicha alifbo tartibida (oshib boruvchi)
murakkab_saralash = sorted(mahsulotlar, key=lambda p: (-p['narx'], p['nom']))
mahsulotlarni_chiqar(murakkab_saralash, "Narxi bo'yicha kamayuvchi, nomi bo'yicha oshuvchi saralash")

# --- 4. min va max yordamida eng arzon va eng qimmatni topish ---
eng_arzon = min(mahsulotlar, key=lambda p: p['narx'])
eng_qimmat = max(mahsulotlar, key=lambda p: p['narx'])

print(f"\n💰 Eng arzon mahsulot: {eng_arzon['nom']} — {eng_arzon['narx']:,} so'm")
print(f"💎 Eng qimmat mahsulot: {eng_qimmat['nom']} — {eng_qimmat['narx']:,} so'm")

# --- 5. filter + lambda bilan kategoriya bo'yicha izlash ---
kategoriya_nomi = "Elektronika"
filtr hisoblangan = list(filter(lambda p: p['kategoriya'] == kategoriya_nomi, mahsulotlar))
mahsulotlarni_chiqar(filtr hisoblangan, f"'{kategoriya_nomi}' kategoriyasidagi mahsulotlar")

# --- 6. map ishlatib umumiy summalarni hisoblash ---
# Har bir mahsulotning umumiy qiymatini (narx * soni) hisoblaymiz
summalar = list(map(lambda p: p['narx'] * p['soni'], mahsulotlar))
jami_summa = sum(summalar)

print(f"\n📊 Do'kondagi barcha mahsulotlarning umumiy qiymati: {jami_summa:,} so'm")
Kodda bajarilgan talablar tahlili:
Ma'lumotlar ombori: mahsulotlar ro'yxatida 11 ta mahsulot (nom, narx, soni, kategoriya kalitlari bilan) shakllantirildi.

Saralash: sorted() va lambda orqali ham oddiy (oshib/kamayuvchi), ham bir nechta shartli (-p['narx'], p['nom']) tartiblash amallari bajarildi.

Ekstremumlar: min() va max() funksiyalariga mos key berilib, kerakli elementlar topildi.

Filtrlash va Xaritalash: filter() yordamida ma'lum toifadagi tovarlar ajratildi, map() yordamida esa umumiy aylanma mablag' oson hisoblab olindi.

Vizualizatsiya: Barcha ma'lumotlar f-string formatlash kodlari ({val:<width} va raqamlarni ajratuvchi {val:,}) yordamida chiroyli jadval ko'rinishida konsolga chiqarildi.
