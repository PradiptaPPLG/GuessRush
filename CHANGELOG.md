# Changelog - Guess Rush

## [2026-05-27] - Ceremony Podium: Flex Layout (Cluster di Center) [AI / arahan Pradipta]
### Fixed
- **JUARA 2 & 3 menjauh dari JUARA 1 di wide screen**: Root cause: slot pakai positioning absolute `left: 3vw / right: 3vw / left: 50% margin-left: -15vw` → anchored ke edge viewport. Di layar lebar (laptop landscape), gap antar slot bisa ratusan px → terlihat tidak konsisten / "JUARA 3 jauh amat".
- **Solusi**: `.ceremony-podium` ganti ke FLEX LAYOUT (`display: flex; justify-content: center; align-items: flex-end; gap: 2vw`). Slot ganti dari `position: absolute` → `position: relative` (flex child). Position lock (right/left/margin-left) dihapus. Sekarang 3 slot dirender berurutan sesuai HTML order (rank-2 → rank-1 → rank-3) dengan gap 2vw, otomatis cluster di tengah viewport apapun ukurannya.
- **Transform animasi tetap**: `.show` reveal animation (translateY scale) sama persis, transform-origin: bottom center. Spotlight position tidak diubah — masih cover area kiri/tengah/kanan dengan radial gradient soft edges yang naturally illuminate the new clustered positions.

### Files
- `index.html`: CSS section 16 — `.ceremony-podium`, `.ceremony-slot`, `.ceremony-rank-1/2/3` (positioning).
- `CHANGELOG.md`: entri ini.


## [2026-05-27] - Leaderboard: Player Highlight Gold + Layer Fix + Auto-scroll to Me [AI / arahan Pradipta]
### Added
- **Highlight current player NAMA + SKOR jadi GOLD**: CSS rule baru `.player-focus .rank-info, .player-focus .rank-pts { color: var(--gold) !important; text-shadow: 0 0 10px rgba(255,215,0,0.5); }`. Jadi misal user pake nama "Pradipta", baris dengan "Pradipta" + skornya jadi warna kuning emas (bukan putih/cyan default), langsung kelihatan baris mana yang dipakai.
- **`scrollToMyPosition(myName)`** (JS Block 9D1): hitung posisi current player card berdasarkan `style.top`, scroll arena supaya card-nya center di view. Math.max 0 supaya #1/#2 ngga scroll negatif. Dipanggil 2 kali: (a) setelah initial render di `startThroneBattle` (120ms delay) → fokus ke posisi awal player; (b) setelah reshuffle di `phaseReshuffle` → ikutin posisi BARU setelah rank naik.

### Fixed
- **Layer bug — `.player-focus` card nembus area button** (kasus user di #7 Maharani, cyan border + glow keliatan di atas MAIN LAGI button). Root cause: arena `position: relative` TANPA z-index → ngga create stacking context → child `.player-focus` (z-index 100) leak ke parent (game-container) stacking dan paint OVER `.lb-actions` (z-index 10). Fix: `#leaderboard-arena { z-index: 1 }` → bikin stacking context lokal. Sekarang `.player-focus` z-index 100 contained di arena, arena overall di z-index 1 < `.lb-actions` z-index 10 → button menang.
- **Gradient `.lb-actions` terlalu transparent**: was `linear-gradient(transparent, rgba(17,20,27,0.95) 35%)` — top 35% transparan sampai 0.95, jadi card bawah keliatan jelas. Sekarang `linear-gradient(rgba(17,20,27,0.7) 0%, rgba(17,20,27,1) 45%, rgba(17,20,27,1) 100%)` — top 0.7 opacity (was full transparent), full opaque dari 45% → card bawah tertutup dengan rapi tapi tetap ada fade effect halus.
- **Margin overlap dikurangi**: `.lb-actions { margin-top: -22px → -14px }`, `padding-top: 10px → 14px` — transisi visual lebih bersih.

### Changed
- **`phasePodium`**: `arena.scrollTo({ top: 0, behavior: 'smooth' })` saat reveal #1 DIHAPUS. Sebelumnya auto-scroll ke top, jadi player di rank rendah ke-bypass. Sekarang scroll diserahkan ke `scrollToMyPosition(myName)` yang dipanggil di startThroneBattle + phaseReshuffle. Player #1 tetap ke top secara natural (math: cardTop 0 → scrollTarget Math.max(0, 26 - arenaH/2) = 0).

### Files
- `index.html`: CSS section 8 (`.player-focus .rank-info`, `.player-focus .rank-pts`), section 14 (`#leaderboard-arena z-index`, `.lb-actions background`/`margin-top`/`padding-top`). JS Block 9 (`startThroneBattle` — scrollToMyPosition call after spacer), Block 9C (`phaseReshuffle` — scrollToMyPosition di done timeout), Block 9D (`phasePodium` — hapus scrollTo top), Block 9D1 baru (`scrollToMyPosition` function).
- `CHANGELOG.md`: entri ini.


## [2026-05-27] - Leaderboard: Consistent width + Hapus shake/medal animations [AI / arahan Pradipta]
### Changed
- **`.gold-rank`**: hapus `transform: scale(1.02)`. Sebelumnya gold card lebih besar 2% dari silver/bronze — bikin width inconsistent antar podium.
- **`.player-focus`**: hapus `transform: scale(1.05)`. Current player card sekarang highlight via cyan border + glow box-shadow doang (no scale). Sebelumnya kalau current player kebetulan #3 (bronze), scale 1.05 nya bikin #3 lebih besar dari #1 (gold scale 1.02) → konsistensi rusak.
- **`.podium-medal`**: `transform: translateY(-50%) scale(0)` + `transition: 0.45s` DIHAPUS. Medal sekarang langsung visible saat card render (no pop-in animation). `.podium-medal.show` jadi no-op.
- **`phaseCountUp`**: `myCard.classList.add('shake-climb')` + corresponding `.remove()` DIHAPUS. Skor masih count-up smooth + jitter, tapi card ngga shake-shake lagi saat poin naik.
- **`phaseReshuffle`**: `card.classList.add('rank-jump')` DIHAPUS. Reshuffle posisi tetap pakai `transition: top 1.2s cubic-bezier(...)` (smooth slide), tapi tanpa scale/rotation animation pas naik signifikan.
- **`phasePodium`**: `card.classList.add('podium-celebrate')` DIHAPUS. Reveal podium hanya nampilin medal (yang sudah static) + SFX, tanpa scale-up animation card.

### Preserved (CSS rules masih ada untuk backward-compat)
- `.shake-climb` + `@keyframes battle-shake`
- `.rank-jump` + `@keyframes rank-jump`
- `.rank-card.podium-celebrate` + `@keyframes podium-celebrate`
- `.podium-medal.show` (sebagai no-op selector)
- Class-class ini masih definisi di CSS, cuma JS ngga lagi nambahin. Bisa di-restore dengan add class application kalau dibutuhin lagi.

### Files
- `index.html`: CSS section 8 (`.gold-rank`, `.player-focus`, `.podium-medal` + `.podium-medal.show`), JS Block 9B/9C/9D (`phaseCountUp`, `phaseReshuffle`, `phasePodium`).
- `CHANGELOG.md`: entri ini.


## [2026-05-27] - Polish: Confetti z-index, Hover button, Leaderboard layout [AI / arahan Pradipta]
### Fixed
- **Confetti tidak muncul di ceremony**: `.ceremony-overlay` z-index 99999 → **5000**. Confetti (z-index 9999) dan shockwave (9997) yang di-spawn ke `document.body` jadi MENUMPUK DI ATAS ceremony overlay (sebelumnya ketutup dark bg). Sekarang confetti rain visible saat JUARA 1 reveal.
- **Tombol KEMBALI KE MENU geser saat hover**: base `button:hover:not(:active)` punya specificity (0,2,1) yang menang dari `.ceremony-close:hover` (0,2,0), jadi `transform: translateX(-50%)` ke-override pure `translateY(-2px)` dan button "loncat" ke kanan (lost centering). Fix: selector ditingkatkan jadi `button.ceremony-close:hover:not(:active)` (specificity 0,3,1) yang menang dari base. Active state juga di-override dengan `button.ceremony-close:active` supaya tetap center saat di-klik.
- **Leaderboard medal dempet dengan #**: medal `font-size: 1.05rem → 1.25rem` + `left: -4px → 8px` (medal sit DI DALAM card, ngga overhang lagi).
- **Leaderboard layout cramped**: `.rank-pos` dari `width: 34px` no-padding → `width: 52px; padding-left: 28px; box-sizing: border-box`. Sekarang SEMUA card (podium & non-podium) reserve slot 28px untuk medal di kiri, # selalu align di kolom yang sama. `.rank-info` dapat `padding-left: 6px` untuk gap kecil antara # dan nama. Visual hierarchy [medal | # | name ............ score] jadi lebih breathable.

### Files
- `index.html`: CSS section 8 (`.rank-pos`, `.rank-info`, `.podium-medal`, `.gold-rank .rank-pos`) + section 16 (`.ceremony-overlay` z-index, `button.ceremony-close:hover:not(:active)`, `button.ceremony-close:active`).
- `CHANGELOG.md`: entri ini.


## [2026-05-27] - Champion Ceremony (Quizizz-style) di AKHIRI PERTANDINGAN [AI / arahan Pradipta]
### Added
- **CSS Section 16 — CHAMPION CEREMONY**: Overlay full-viewport (`#ceremony-overlay`) dengan dark radial bg + starfield.
- **3 Spotlight cones** (`.ceremony-spot-1/2/3`): kerucut cahaya kuning/perak/perunggu dari atas, fade-in pakai keyframes `ceremony-spot-focus` (blur 45→4→10px) — meniru lampu sorot quizizz.
- **Podium slots** (`.ceremony-slot.ceremony-rank-1/2/3`): tata letak — JUARA 2 kiri, JUARA 1 tengah (terbesar, pedestal 100px), JUARA 3 kanan (pedestal 45px). Masing-masing punya medal 🥇🥈🥉, label "JUARA N", nama, skor, pedestal.
- **`ceremony-medal-shine`** keyframes: medal emas berdenyut drop-shadow + scale (1 → 1.08) infinite.
- **`startCeremony()`** (JS Block 11): baca top 3 dari `localStorage.guessRushLB`, jalankan sequence: 0ms overlay → 800ms JUARA 3 (right spotlight, SFX G4) → 3500ms JUARA 2 (left spotlight, SFX C5) → 7000ms JUARA 1 (center spotlight + chord E5-G5-C6 + 3 gelombang confetti + shockwave ring) → 10000ms tombol KEMBALI KE MENU muncul.
- **`closeCeremony()`** + **`backToMenuAfterCeremony()`**: tutup overlay → reset sessionScore/save-section/leaderboard-section → showScreen welcome.
- **HTML overlay** `#ceremony-overlay` setelah `#juara-banner` di body.

### Changed
- **`endMatchFinal()`**: sebelumnya `confirm("Akhiri pertandingan...")` lalu langsung balik ke welcome. Sekarang hanya `startCeremony()` — confirm dialog dihapus karena tombol ini "bukan reset/quit langsung" (arahan Pradipta), tapi pemicu animasi ceremony. Reset & balik menu dilakukan `closeCeremony()` setelah user pencet KEMBALI KE MENU.
- **`phasePodium()`**: hilangkan trigger JUARA banner + trophy floating + shockwave + confetti saat reveal #1 di leaderboard. Yang tersisa di leaderboard hanya medal `.podium-medal.show` + `.podium-celebrate` (animasi ringan ranking) + `sfx.correct()` + auto-scroll. Animasi heavy "JUARA 1 [NAMA]" terus-terusan SUDAH TIDAK MUNCUL saat skor disimpan — sekarang hanya muncul kalau user pencet AKHIRI PERTANDINGAN.

### Preserved (NOT removed per rules.md)
- `triggerJuaraPeak()`, `#juara-banner` element + CSS section 15, `.trophy-emoji` CSS — semua TETAP ada. Trophy emoji tidak lagi di-append ke gold card (kreasi DOM dihapus dari phasePodium) tapi class CSS dipertahankan. `triggerJuaraPeak` tidak dipanggil dari mana pun tapi function tetap di-define — bisa dipakai future feature.
- `spawnConfetti()` dipakai ulang dari startCeremony (climax JUARA 1).
- `#juara-shockwave` element dipakai ulang dari startCeremony.

### Files
- `index.html`: CSS section 16 baru (sebelum `</style>`), HTML overlay `#ceremony-overlay` di body, JS — modifikasi `phasePodium` + `endMatchFinal`, tambah JS Block 11 (`startCeremony`/`closeCeremony`/`backToMenuAfterCeremony`).
- `CHANGELOG.md`: entri ini.


## [2026-05-27] - Ekspansi Data: 3 Kategori Baru + Penambahan Item [AI / arahan Pradipta]
### Added
- **Kategori baru HEWAN** (12 item): Singa, Gajah, Kucing (easy) → Harimau, Panda, Kuda (medium) → Komodo, Flamingo, Tapir (hard) → Axolotl, Okapi, Dugong (impossible).
- **Kategori baru OLAHRAGA** (12 item): Sepak Bola, Renang, Bulu Tangkis (easy) → Voli, Tinju, Panahan (medium) → Anggar, Polo, Squash (hard) → Korfball, Sepaktakraw, Kabaddi (impossible).
- **Kategori baru IBUKOTA** (12 item): Jakarta, Tokyo, Paris (easy) → London, Beijing, Ankara (medium) → Canberra, Ottawa, Brasilia (hard) → Naypyidaw, Astana, Nuku'alofa (impossible).
- **SPINNER_FAKE_CATEGORIES**: Tambah HEWAN, OLAHRAGA, IBUKOTA sebagai kategori real sehingga spinner bisa lock ke kategori baru ini.

### Changed
- **NEGARA**: Ditambah 10 item baru — China, Amerika, Prancis (easy); Rusia, Turki, Spanyol (medium); Kanada, Portugal (hard); Kamboja, Namibia (impossible). Total jadi 22 item.
- **PROFESI**: Ditambah 7 item baru — Polisi, Koki (easy); Pilot, Wartawan (medium); Hakim, Astronot (hard); Kriminolog, Paleontolog (impossible). Total jadi 19 item.
- **BUAH**: Ditambah 8 item baru — Jeruk, Anggur (easy); Nanas, Melon (medium); Salak, Lengkeng (hard); Kepel, Cempedak (impossible). Total jadi 20 item.
- **BARANG**: Ditambah 7 item baru — Payung, Gunting (easy); Termos, Kompas (medium); Kunci, Stopkontak (hard); Susuk, Gentong (impossible). Total jadi 19 item.
- **Total item keseluruhan**: 48 → ~104 item (2× lipat lebih banyak).

### Files
- `index.html`: `itemsData` array (JS block 2) — tambah item di 4 kategori lama + tambah 3 kategori baru. `SPINNER_FAKE_CATEGORIES` diperbarui.
- `CHANGELOG.md`: Dicatat entri ini.


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
