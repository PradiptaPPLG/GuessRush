# Question — Kaching atau Kaboom! [2026-05-28]

Beberapa hal kecil yang ambigu di spec, saya assume default + tetap implement. Bisa dikoreksi kapanpun.

## 1. Jumlah bom: EXACTLY 5 atau UP TO 5?

Spec: "max bom buat kedua pemain itu = 5".

**Default sekarang**: EXACTLY 5 — tombol SELESAI di-disable sampai pemain naro tepat 5 bom. Reasoning: kalau optional, pemain bisa cheat dengan naro 0 bom dan menjebak lawan menebak random.

Alternatif: optional 1-5 bom (lebih lenient).

---

## 2. Edge case — semua 25 kotak ke-reveal tapi keduanya masih hidup

Possible scenario: banyak coin/uang, sedikit hit bom yang bukan punya sendiri. Total nyawa 3+3=6, total bom max 10 (atau lebih sedikit kalau duplikat). Jadi mungkin saja kalau players "lucky" pick semua coin dulu.

**Default sekarang**: Kalau semua kotak ke-reveal sebelum ada yang mati → menang adalah pemain dengan **sisa nyawa terbanyak**. Kalau sama → DRAW (kedua dapat +100 instead of +200/+50).

Alternatif: lanjut pick ulang (ngga mungkin karena kotak habis), atau auto-win P1 / random.

---

## 3. Cell dengan bom sendiri (tanpa bom lawan) — saat pick

Spec: "bom yang ditetapin pemain 1 ketika giliran pemain 1 maka tampilin aja bomnya dimana aja".

**Default sekarang**: Reveal bom.png + flip animation + SFX "place" (subtle, bukan boom) + **tidak ada damage**, lanjut ke giliran lawan. Logika: pemain naro bom-nya sendiri, sadar resikonya, jadi visual indicator aja.

---

## 4. Bomb placement — bisa undo (klik ulang)?

**Default sekarang**: YA — bisa klik kotak yang sudah ada bom untuk un-place. Counter "sisa bom" update real-time. Reasoning: pemain perlu fleksibilitas mengatur posisi sebelum SELESAI.

---

## 5. Turn-around timer — antara pick P1 dan pick P2

**Default sekarang**: TIDAK ADA timer 5-detik antara pick P1 → pick P2. Setelah pick + animasi reveal (~1s), langsung ke giliran lawan. Reasoning: timer 5-detik di spec hanya untuk transisi bomb placement → guessing phase (P2 berbalik sekali sebelum P1 pick pertama). Selama guessing, board public (revealed cells stay revealed, kedua pemain bisa lihat).

Alternatif: timer 3-5 detik antar pick untuk dramatic effect / give breathing room.

---

## 6. View Leaderboard dari result screen

**Default sekarang**: bom.html render leaderboard sendiri (top 30 + highlight winner gold + count-up animation untuk P1 dan P2). Setelah LB selesai animasi, tombol KEMBALI KE MENU → navigate ke index.html (welcome).

Alternatif: sessionStorage flag → index.html trigger LB display dengan throne battle yang sudah ada di sana (sharing animation code).

---

Kalau semua default OK, ngga perlu reply — saya tinggal lanjut polish. Kalau ada yang mau diubah, sebutin nomornya aja.
