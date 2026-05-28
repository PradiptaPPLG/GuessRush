# Question — Perfect Timer [2026-05-28]

Semua jawaban final dari Pradipta sudah tercatat di bawah. Ini jadi referensi saat implementasi.

---

## 1. Format angka target & display stopwatch — **FINAL**

Format **SS.HS** (detik.sentidetik). Contoh: target "08.00" = 8 detik 00 sentidetik. Bukan menit.

---

## 2. Range angka target random — **FINAL**

**3.00 - 10.00 detik** (bukan 3-15).

---

## 3. Stopwatch arah — **FINAL**

**NAIK** dari 0.00 → target. Max stopwatch 15.00 detik — kalau lewat 15s tanpa klik, auto-stop dengan akurasi 0%.

---

## 4. Formula akurasi & pemenang — **FINAL**

- **Akurasi (%) per pemain per ronde**: `max(0, 100 - (|target - clicked| / target * 100))`. Untuk skor display & tentuin winner ronde.
- **Pemenang RONDE** = pemain dengan akurasi LEBIH TINGGI di ronde itu. Kalau akurasi sama persis → ronde itu tie (0 wins untuk dua-duanya).
- **Pemenang MATCH** = **BEST-OF-ROUNDS** (pemain yang menang ronde lebih banyak). Bukan total akurasi.
- **Round select**: 1, 3, 5 (ganjil — supaya minim kemungkinan seri).

Contoh 3 ronde:
```
RONDE 1: P1 menang
RONDE 2: P1 menang
RONDE 3: P2 menang
→ P1 menang match (2-1)
```

---

## 5. Poin LB — **FINAL**

Fixed berdasarkan win/lose match (bukan akurasi):
- Winner match → **+200**
- Loser match → **+50**
- Seri match (rare — total wins sama) → **+150 / +150**

---

## 6. Flow per ronde — **FINAL**

Sesuai default:
1. Animasi ronde (RONDE X/Y) ~1.5s
2. Sistem random target untuk ronde ini (3.00-10.00). Sama untuk P1 & P2 di ronde itu.
3. P1 turn — "GILIRAN [NAMA P1]" + target ("STOP DI 08.45") + 5 detik countdown
4. Stopwatch jalan, P1 klik tombol / SPASI untuk stop
5. Hasil P1 — akurasi + label (PERFECT/AMAZING/dst) + efek peak kalau 100% (~2.5s)
6. P2 turn (target sama)
7. Ronde berikutnya
8. Setelah ronde terakhir → result match (winner +200, loser +50, atau seri +150/+150)

---

## 7. Input klik — **FINAL**

- **Touchpad / mouse**: cursor harus mengarah ke tombol stop untuk register klik (onclick di element tombol, bukan global)
- **Spacebar**: bebas, klik di mana aja (keydown listener global)
- **Penting**: tombol click handler harus INSTANT (capture timestamp di first line). Efek visual press tombol biasa OK, TAPI **dilarang efek berat** (heavy animation / transition) yang bisa cause delay sebelum timestamp di-capture.

---

## 8. Main Lagi flow — **FINAL** (BERBEDA dari bom.html)

Karena project publik / pameran → **MAIN LAGI HARUS INPUT NAMA LAGI** dari awal. Bukan skip register. Tombol "MAIN LAGI" → balik ke screen register kosong.

---

## 9. Modal Mode Lainnya di index.html — **FINAL**

- Card 3 → **PERFECT TIMER** unlocked, thumbnail `perfecttimer.png`, navigate ke `perfecttimer.html`
- Card 4 → **TEBAK GAMBAR** tetap locked, thumbnail `thumbnailtebakgambar.png`

---

## 10. Edge case Seri — **FINAL**

Seri match (total wins sama) → **+150 / +150** ke LB. Result screen tampilin "SERI!" dengan 2 card pemain side-by-side.

---

## 11. Difficulty selector NORMAL / HARD — **FINAL** [iterasi setelah implementasi awal]

Setelah pilih ronde → screen baru pilih difficulty:
- **NORMAL**: target detik **BULAT** (.00). Contoh: `04.00`, `07.00`, `09.00`. Generator pakai `Math.floor(Math.random() * 8) + 3` → 3-10 detik bulat.
- **HARD**: target dengan sentidetik **RANDOM**. Contoh: `04.26`, `07.83`, `09.51`. Generator pakai random 3.00-10.00 step 0.01 (sama logic sebelumnya).

Flow baru: register → pilih ronde → **pilih difficulty (NORMAL/HARD)** → VS animation → ...

Difficulty ditampilkan di label VS screen ("PERFECT TIMER · 3 RONDE · HARD") dan round intro.

---

Semua confirmed. Mulai implementasi.
