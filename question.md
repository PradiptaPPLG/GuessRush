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

---
---

# Question — Just Half It! [2026-05-28]

Game baru: pemain harus membelah objek (PNG dari `assets/halfcut/`) **tepat di 50%**. Teknik: 1 PNG dirender 2x dengan `clip-path: inset(0 50% 0 0)` (kiri) & `inset(0 0 0 50%)` (kanan), terus dianimasi pisah saat cut. Cut position dinamis sesuai akurasi user. Asset pool 11 PNG: bola, burger, coklat, cupcake, eskrim, kue, nanas, pepsi, pizza, pretzel, semangka.

Tujuan section ini: kunci semua detail mekanik sebelum mulai ngoding. Tunggu jawaban Pradipta dulu.

---

## A. KONTROL PISAU (CUT INPUT) — paling kritis, pilih 1

User bilang **"klik drag"**. Ada 3 interpretasi:

**Opsi A1 — Click-drag freehand**
Pisau (🔪 atau .png) parkir di bawah arena. Pemain **mousedown** di pisau, **drag** ke atas objek, **mouseup** = posisi pisau saat lepas = cut position. Touch: pointerdown → pointermove → pointerup.

**Opsi A2 — Cursor follows mouse, click to cut**
Pisau **terus mengikuti** posisi cursor di atas arena (tidak perlu drag). Pemain hover ke posisi yang dia mau, klik 1x = potong. Lebih mirip "aim & shoot".

**Opsi A3 — Slider + tombol POTONG**
Ada slider horizontal di bawah objek. Pemain geser slider untuk pilih posisi, lalu klik tombol "POTONG!" untuk confirm. Lebih precise tapi kurang "feel" kayak benerin motong.

**→ Pilih mana?** (Saya rekomendasikan **A2** — paling smooth, dukung touch & mouse, sesuai vibe game arcade. Tapi user explicit bilang "klik drag" → mungkin A1 lebih dia maksud?)

---

## B. KNIFE VISUAL

Folder `halfcut` ngga ada `knife.png`. Pilihan:

- **B1**: Pakai emoji 🔪 (cepet, no asset, ukuran ~3rem).
- **B2**: CSS-drawn knife (rectangle blade + handle gradient — looks more polished).
- **B3**: Pradipta akan add `knife.png` ke `assets/halfcut/` nanti — beri tau dulu kalau perlu.

**→ Pilih yang mana?**

---

## C. AKURASI FORMULA

Cut position di-capture sebagai persentase dari width objek (0%-100%, sweet spot = 50%).

**Formula draft**: `accuracy = max(0, 100 - |50 - cutPercent| * 2)`

Contoh:
- Cut di 50% → 100% ✅
- Cut di 48% atau 52% → 96%
- Cut di 40% atau 60% → 80%
- Cut di 30% atau 70% → 60%
- Cut di 0% atau 100% → 0%

**→ OK pakai formula ini?** Atau mau lebih curam (misal `* 3` → cut di 40% jadi 70%, lebih sulit dapet score tinggi)? Atau mau pakai formula non-linear (ease curve)?

---

## D. LABEL TIERS (per ronde, ditampilkan setelah cut)

Draft tier (mirip PerfectTimer punya PERFECT/AMAZING/dst):

| Akurasi  | Label             | Warna             | Efek          |
|----------|-------------------|-------------------|---------------|
| 100%     | **PERFECT HALF!** | gold + glow       | confetti peak |
| 95–99%   | AMAZING SLICE     | gold              | glow          |
| 85–94%   | CLEAN CUT         | cyan              | normal        |
| 70–84%   | OK CUT            | white             | normal        |
| 50–69%   | WONKY             | orange            | normal        |
| <50%     | BUTCHERED         | red               | shake         |

**→ OK pakai ini?** Mau ganti nama/threshold/warna?

---

## E. MODE DIFFICULTY (NORMAL / HARD)

PerfectTimer punya NORMAL (target bulat) & HARD (target random sentidetik). Apakah Just Half It juga butuh? Opsi mode HARD:

**E1**: Objek **bergerak slide kiri-kanan** perlahan (~2 detik per cycle). Pemain harus timing klik selain aim.

**E2**: Objek **berotasi pelan** (~360°/8 detik). Cut tetap vertikal (left/right halves), tapi pemain harus baca rotasi.

**E3**: Objek **ukurannya lebih kecil** (50% width) — sweet spot lebih sempit secara visual, tapi proporsi tetep 50%.

**E4**: **Single mode aja, tanpa difficulty selector**. Skip screen difficulty.

**E5**: NORMAL = static. HARD = objek slide kiri-kanan + ada wind line (visual deception) yang offset dari true center.

**→ Pilih mode (atau "tanpa difficulty")?**

---

## F. RONDE & FLOW

Sesuai PerfectTimer:
- F1. Ronde selector **1 / 3 / 5** — confirm?
- F2. **Same object** untuk P1 & P2 di ronde yang sama (fair) — confirm?
- F3. Object **shuffle no-repeat** dalam 1 match (sampai pool habis baru reset) — confirm?
- F4. Best-of-rounds untuk match winner — confirm?
- F5. LB: winner **+200**, loser **+50**, draw **+150/+150** — confirm? *(sama PerfectTimer)*
- F6. Per ronde: Round intro 1.5s → "GILIRAN [P1]" + 3s countdown → P1 cut → result P1 (2.5s) → P2 turn → result P2 → ronde berikutnya — confirm?

---

## G. TIME LIMIT PER CUT

PerfectTimer max 15s sebelum auto-stop. Just Half It mau:

- **G1**: Tidak ada time limit — pemain bebas aim selama mau.
- **G2**: Max 10 detik untuk klik, kalau lewat auto-miss (akurasi 0%).
- **G3**: Max 5 detik. Bikin pressure.

**→ Pilih mana?**

---

## H. CUT ANIMATION & VISUAL

- H1. Dua bagian PNG **slide pisah** (translateX ±80px) selama ~600ms — confirm?
- H2. Overlay garis gradient gelap di cut edge (mitigasi "no cross-section") — confirm? Garis ini ukuran/tebal berapa? Default: 4px width, gradient `linear-gradient(90deg, rgba(0,0,0,0.7), transparent)`.
- H3. Setelah pisah, halves **fade out** lalu menghilang sebelum show result? Atau halves **stay on screen** sebagai bukti dimana pemain motong?
- H4. **Slow-mo effect** untuk cut 100% (knife freeze 0.3s sebelum animasi slice)?
- H5. **Ghost line** ditengah objek setelah cut (vertical dashed line di 50%) sebagai feedback "ini posisi seharusnya"?

---

## I. THUMBNAIL DI index.html

User bilang: **"tetapkan thumbnailjusthalfit menggantikan ke speed quiz"**.

Catatan: Slot 4 di `index.html` saat ini sudah BUKAN "speed quiz" lagi — sebelumnya udah diganti jadi **TEBAK GAMBAR** (locked, `thumbnailtebakgambar.png`). Jadi yang dimaksud user adalah:

- **Slot 4 (locked tebak gambar)** → **JUST HALF IT!** (unlocked, `thumbnailjusthalfit.png`, navigate ke `justhalfit.html`).
- Card 4 status: **UNLOCKED** (klikabel, PLAY badge).
- Setelah ini: total 4 unlocked (Sambung Kata, Kaching/Kaboom, Perfect Timer, Just Half It!) — header text "3 ARENA BUKA · 1 SEGERA" perlu diganti **"4 ARENA BUKA"** atau tetep ada slot 5 placeholder?

**→ Confirm replace tebak gambar → just half it. Lalu mau ada placeholder card 5 (locked, tebak gambar geser ke situ) atau hilangin tebak gambar dulu?**

---

## J. FILE & TECHNICAL

- J1. File baru `justhalfit.html` standalone (sama pattern bom.html / perfecttimer.html) — confirm?
- J2. Share `localStorage.guessRushLB` — confirm?
- J3. Light mode bootstrap (IIFE block 0) + CSS `body.light` override — confirm sama style PerfectTimer?
- J4. Tombol "MAIN LAGI" di result match → balik ke register screen kosong (re-input nama) — confirm? *(public exhibition rule)*
- J5. CHANGELOG.md entry wajib — confirm? *(rules.md mandatory)*

---

## K. EDGE CASES

- K1. Pemain klik di **luar area objek** (misal di luar bounding box) — abaikan input, atau treat sebagai cut di 0%/100% (akurasi 0%)?
- K2. Kalau ada time limit dan time habis → akurasi 0%, "TIME OUT!" label — confirm?
- K3. Kalau P1 & P2 di ronde sama dapet **akurasi sama persis** (tie ronde) → 0 wins untuk dua-duanya (sama PerfectTimer) — confirm?

---

Tunggu jawaban Pradipta. Setelah confirmed, mulai implementasi.
