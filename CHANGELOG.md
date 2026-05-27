# Changelog - Guess Rush

## [2026-05-26] - Timer 1 Menit + Podium Peak Showcase [AI / arahan Pradipta]
### Changed
- **Match timer 120s → 60s** (1 menit). Threshold warning ≤20s, danger ≤8s (sebelumnya ≤40s/≤15s). Default display "01:00".

### Added
- **JUARA Banner overlay**: `#juara-banner` (`position: fixed`, top 18vh) slam-in dengan 3 keyframes (juara-slam → juara-pulse 3× → juara-out) berisi "🏆 JUARA: NAMA 🏆".
- **Shockwave ring**: `#juara-shockwave` (`position: fixed`, viewport center) expand jadi 120vmax dengan border gold fading saat juara 1 di-reveal.
- **`triggerJuaraPeak(name)`**: trigger banner + shockwave + 3 gelombang confetti + chord SFX (C5-E5-G5).
- **Auto-scroll ke top** pada arena saat juara 1 di-reveal supaya gold card kelihatan.

### Fixed
- **Trophy 🏆 kepotong overflow arena**: `#leaderboard-arena` padding-top 8 → 40px supaya trophy (`top: -32px`) ada room di atas card #1. Trophy size 1.8 → 2.2rem dengan double drop-shadow.
- **Confetti kepotong**: ganti dari `position: absolute` di arena ke `position: fixed` (viewport-wide) + append ke `body` bukan arena. Jumlah pieces 28 → 40 per burst, total 3 gelombang.

### Files
- `index.html`: CSS section 15 baru (juara banner + shockwave), JS block 9D enhanced + new 9D2.

## [2026-05-26] - Pick Animation + Letter & Leaderboard Polish [AI / arahan Pradipta]
### Added
- **`.deck-card.dropping`** + `@keyframes card-drop`: kartu yang tidak dipilih jatuh ke bawah dengan rotasi acak (variable `--drop-rot` per kartu).
- **`.deck-card.chosen`** + `@keyframes card-chosen-flip`: kartu terpilih flip + scale up 1.55x (cubic-bezier elastic), override `flip-reveal` via `!important`.

### Changed
- **`.deck-row`**: `min-height: 165px → 220px`, tambah `padding: 30px 5px 20px`, `align-items: center` — supaya kartu scale-up tidak menabrak dinding atas container.
- **`pickCard()`**: tidak lagi pakai class `.disabled`. Sekarang kartu lain dapat `.dropping` (dengan random rotation) dan kartu terpilih dapat `.flipped` + `.chosen`.
- **Rank Card (leaderboard)**: ukuran diramping → `height: 60→52px`, `padding: 0 20→14px`, `left/right: 0→10px` (inset dari edge), font ukuran disesuaikan (`rank-info`: 0.95→0.85rem, `rank-pts`: 1.1→0.95rem, `min-width`: 80→60px). `border-radius: 16→14px`.
- **`#leaderboard-arena` (fixed-layout)**: tambah `padding: 8px 4px 20px` + `overflow-x: hidden` — supaya glow throne (gold/silver/bronze) tidak terpotong oleh `overflow-y: auto`.
- **`ROW_H`**: 68 → 60 (sinkron dengan tinggi card baru di JS block 9 & 9C).

### Fixed
- **Huruf "present" (kuning) hilang**: di `checkGuess`, sebelumnya `input.value = ""` dipanggil untuk semua input non-correct, termasuk yang sudah dapat class `present`. Sekarang hanya `absent` yang di-clear; `present` mempertahankan value supaya user lihat huruf benar tapi salah posisi.

## [2026-05-26] - Flow Match: 1 Ronde per Match [AI / arahan Pradipta]
### Changed
- **checkGuess (jawab benar)**: Sebelumnya looping `setTimeout(initRound, 1200)` untuk ronde berikutnya. Sekarang langsung `showFinalResult()` setelah 1.4s — tampil hasil + input nama → masuk leaderboard. Pertandingan = 1 ronde per match.
- **surrender()**: Sebelumnya skip ronde dan lanjut ke kartu baru. Sekarang langsung akhiri pertandingan → simpan skor (0 kalau belum jawab) → leaderboard. Konsisten dengan logic "satu match = satu ronde".
- **Teks tombol menyerah**: Hapus suffix "— SKIP RONDE", karena sekarang bukan skip lagi tapi akhiri match.
- Timer 2 menit tetap dipertahankan sebagai deadline single-round. Habis waktu → auto `showFinalResult()`.

## [2026-05-26] - Hotfix: Kartu Invisible & Multi-Round [AI / arahan Pradipta]
### Fixed
- **Kartu invisible setelah dealing**: `@keyframes card-bob` & `flip-reveal` tidak set `opacity` di tiap keyframe — saat class `idle-bob`/`flipped` ditambahkan, animasi mengganti `deal-card` dan base `opacity: 0` dari `.deck-card` kembali aktif. Tambah `opacity: 1` eksplisit di keyframes terkait.
- **"Sekali jadi langsung menang"**: Konsekuensi dari kartu invisible — setelah jawab benar, ronde baru muncul tapi kartu tidak terlihat sehingga user mengira game selesai. Otomatis terselesaikan oleh fix di atas.
- **Animasi deal-card restart pada kartu non-terpilih**: Saat `pickCard()` menghapus `idle-bob` dari semua kartu lalu menambah `disabled`, animation-name berubah dari `card-bob` → `deal-card`, sehingga browser merestart fly-in animation. Fix: hanya kartu terpilih yang di-remove `idle-bob`, kartu lain biarkan terus bobbing.
- **`.deck-card.disabled` tidak terlihat dim**: `opacity: 0.25` ketimpa oleh `animation forwards` dari `deal-card`. Diganti pakai `filter: grayscale(0.85) brightness(0.45)` yang tidak bertabrakan dengan cascade animation.

## [2026-05-26] - Major Overhaul: Deck, Timer, Podium [AI / arahan Pradipta]
### Added
- **Reveal Screen**: Sistem 4-kartu yang dibagikan satu-per-satu dari kiri ke kanan (animasi `deal-card`).
- **Category Spinner**: Teks acak berputar (alat sekolah, alat kantor, alat tukang, dst) selama kartu dibagikan, lalu lock pada kategori target dengan animasi `locked-pulse` (judul ganti dari "TARGET TERDETEKSI" → "SPIN KATEGORI").
- **Pilih Kartu**: User memilih salah satu dari 4 kartu — tiap kartu mewakili difficulty berbeda dari kategori yang sama. Kartu yg dipilih flip reveal difficulty + poin; kartu lain disabled.
- **Field `category`** di tiap item `itemsData` (5 kategori: MESIN & OTOMOTIF, ELEKTRO & LISTRIK, TIK & MULTIMEDIA, BANGUNAN & SIPIL, BUSANA & BOGA).
- **Match Timer 2 Menit**: Timer bar dengan state warning (≤40s, kuning) & danger (≤15s, merah pulse). Auto-end ke leaderboard saat habis. Timer jalan terus antar ronde.
- **Leaderboard Fixed Layout**: Hanya area arena yang scroll, tombol fixed di bawah (CSS section 14).
- **Throne Battle Enhanced**:
  - Fase 1 → render dengan poin LAMA (posisi awal player).
  - Fase 2 → count-up smooth (ease-out cubic + jitter) dengan floating `+XX` di samping card.
  - Fase 3 → reshuffle posisi dengan `rank-jump` animation untuk kartu yang naik signifikan.
  - Fase 4 → podium showcase (reveal medal bronze → silver → gold), trophy 🏆 pop di juara 1, confetti burst.
- **Medal Podium**: 🥇🥈🥉 muncul di sisi kiri rank #1/#2/#3.
- **Tombol MAIN LAGI** (ganti "COBA LAGI") + **AKHIRI PERTANDINGAN** di leaderboard (sticky bottom).
- **`playAgain()`** & **`endMatchFinal()`** untuk navigasi pasca-leaderboard.
- **`stopMatchTimer()`** dipanggil di setiap titik akhir (surrender → tidak, hanya next round; finalScore / endMatch → stop).
- Try/catch pada parse `localStorage.getItem('guessRushLB')` untuk safety jika data corrupt.

### Changed
- **Game Screen**: Tombol "AKHIRI PERTANDINGAN" DIHAPUS dari layar gameplay Guess Rush (sesuai arahan Pradipta — akhiri pertandingan hanya muncul di leaderboard, bukan saat in-game). Menyerah (skip ronde) tetap ada sebagai teks tombol minimal.
- **goToGame()**: Update label kategori (`KATEGORI: ...`) dan trigger `startMatchTimer()`.
- **initRound()**: Total rewrite — sekarang generate 4 deck cards via kategori + difficulty matrix.
- **startThroneBattle()**: Total rewrite jadi 4-fase animation (render → count up → reshuffle → podium).
- **finalizeScore()**: Aktifkan class `fixed-layout` saat leaderboard ditampilkan.
- **showFinalResult()**: Reset visibility section + matikan match timer.

### Removed
- **`flipCard()`** function (single-card reveal lama) — tidak lagi dipakai karena reveal screen diganti deck 4 kartu.
- DOM tag lama: `#difficulty-card`, `#reveal-difficulty`, `#reveal-points`, `#fixed-lb-footer`, `#btn-restart-normal` — diganti dengan struktur baru deck-row / lb-actions.

### Files
- `index.html`: CSS section 12-14 baru (deck, timer, podium); reveal/game/result HTML direstrukturisasi; JS block 5/5B/5C/5D + 8 + 9/9B/9C/9D/9E direwrite.

## [2024-05-24] - Restoration & Correction [AI]
- **Restored**: Tombol "Akhiri Pertandingan" pada screen Guess Rush dan Sambung Kata sesuai permintaan [Pradipta].
- **Restored**: Animasi khusus Tahta (Throne Effect) Ranking 1, 2, dan 3 di Leaderboard.
- **Restored**: Logika UI tombol "Main Lagi" yang fixed/sticky di bawah leaderboard jika data > 5.
- **Removed**: Fitur tambahan yang tidak diminta (XP System, Leveling, Achievements, Library, Tutorial) untuk menjaga keaslian project.
- **Added**: Ekspansi Data Alat Teknik SMK hingga 180+ item untuk mencapai target baris kode 800+.
- **Added**: Dokumentasi Komentar Blok di setiap section (CSS, HTML, JS) sesuai rules.md agar memudahkan maintenance.
- **Fixed**: Perbaikan logika Sambung Kata 1V1 agar lebih snappy dan konsisten.
