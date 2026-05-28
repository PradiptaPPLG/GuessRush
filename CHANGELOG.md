# Changelog - Guess Rush

## [2026-05-28] - Welcome Subtitle: Speed up Glitch Transition (3000ms → 1200ms) [AI / arahan Pradipta]
### Changed
- **`SUBTITLE_GLITCH_MS`**: `3000` → `1200`. Glitch transition kerasa snappy (~1.2s) bukan kelamaan (~3s). Arahan Pradipta: "glitch nya kelamaan, aga cepet glitch nya".
- **`frameMs` scramble loop**: `55ms` → `35ms` (~18fps → ~28fps). Scramble char swap lebih cepat supaya tetep chaotic walau window di-shorten ke 1.2s.
- **Total siklus per phrase**: was `4500 + 3000 = 7500ms`, sekarang `4500 + 1200 = 5700ms`.

### Files
- `index.html`: JS Block 12 — `SUBTITLE_GLITCH_MS` const + `frameMs` local di `_glitchTransitionSubtitle`.
- `CHANGELOG.md`: entri ini.


## [2026-05-28] - Welcome Subtitle: Random Phrase Cycler + Glitch Transition [AI / arahan Pradipta]
### Changed
- **Subtitle welcome screen** (was: static "ARENA TAKLUKAN ALAT TEKNIK SMK") → random phrase cycler dengan 6 cyberpunk phrases:
  1. "RUN THE CODE: RUN YOUR BRAIN"
  2. "DECRYPT THE SYNTAX, WIN THE GAME!"
  3. "PUSH YOUR LOGIC TO THE MAXIMUM"
  4. "FAST THINKING, ZERO ERROR"
  5. "MASTER THE LOGIC, CONQUER THE CODE"
  6. "WHERE LOGIC MEETS SPEED"
- **Siklus**: tampil 4.5 detik tenang → 3 detik glitch transition → next random phrase (tidak repeat berurutan).

### Added
- **CSS Section 5B** baru (di antara section 5 INTRO dan section 6 BUTTONS) — `.welcome-subtitle` + `.glitch-active` variant. Komponen efek glitch:
  - **RGB split text-shadow**: cyan offset -2px + magenta offset +2px → typical chromatic aberration look.
  - **Position jitter** (`@keyframes subtitle-jitter`, 0.12s steps(1) infinite): translate(-2px to 2px, -1px to 1px) random 7 keyframes.
  - **Pseudo ::before clip-path tear (top band)**: red layer dengan `clip-path: inset(0 0 70% 0)` shifting horizontally → simulasi "scan line tear" atas.
  - **Pseudo ::after clip-path tear (bottom band)**: cyan layer dengan `clip-path: inset(70% 0 0 0)` → tear bawah. Background card-bg supaya overlay nutup base text di area tear.
  - Pseudo pakai `attr(data-text)` — di-keep in sync oleh JS tiap frame scramble.
- **JS Block 12 (Welcome Subtitle Cycler)**:
  - `SUBTITLE_PHRASES[]` array 6 phrases.
  - `SUBTITLE_GLITCH_MS = 3000`, `SUBTITLE_VISIBLE_MS = 4500`.
  - `_pickNextSubtitlePhrase()`: random non-repeat (do-while loop sampai dapat index beda dari current).
  - `_glitchTransitionSubtitle(newText)`: scramble loop ~18fps (frameMs 55). Progressive lock dari kiri — character index < `floor(newText.length * t)` di-lock ke final char, sisanya random dari `SUBTITLE_SCRAMBLE_CHARS` (`!<>-_/\\[]{}=+*^?#&%@$01`). Spaces / punctuation (`,:!`) selalu skip scramble untuk readability. Saat t=1, lock final + remove `.glitch-active` class.
  - `_startSubtitleCycle()`: pertama tunggu 4.5s (initial visible phase), lalu trigger transition pertama, lalu setInterval setiap `visible + glitch = 7500ms`.
  - **Guard**: cycle function cek `#screen-welcome.active` — kalau user udah navigate ke screen lain, skip animation (ngga waste compute).

### Files
- `index.html`:
  - CSS section 5B baru (sebelum section 6).
  - HTML: `<p>` subtitle ditambahin `id="welcome-subtitle"` + `class="welcome-subtitle"`, initial text diganti ke phrase #0.
  - JS block 12 baru (sebelum `</script>` close), auto-kick via `_startSubtitleCycle()` di end-of-script.
- `CHANGELOG.md`: entri ini.

### Preserved (NOT removed per rules.md)
- Welcome screen layout, intro-title (GUESS RUSH banner), intro-underline, semua tombol (MASUK ARENA / MODE LAINNYA / PENGATURAN) — unchanged.
- Subtitle `<p>` margin / font-size / letter-spacing / font-weight inline style tetap, cuma color overridden saat glitch-active (cyan→white).


## [2026-05-28] - Perfect Timer: Difficulty Selector NORMAL / HARD [AI / arahan Pradipta]
### Added
- **Screen `#screen-difficulty`** (baru, di antara `#screen-rounds` dan `#screen-vs`). 2 cards horizontal:
  - **NORMAL** (green/cyan, icon 🟢): target detik **BULAT** (sentidetik = 00). Contoh `04.00 · 07.00 · 09.00`. Generator: `Math.floor(Math.random() * (TARGET_MAX - TARGET_MIN + 1)) + TARGET_MIN` → integer 3..10 inclusive.
  - **HARD** (red/danger, icon 🔴): target dengan sentidetik **RANDOM** (step 0.01). Contoh `04.26 · 07.83 · 09.51`. Generator: tetap pakai logic awal `Math.round((TARGET_MIN + Math.random()*range) * 100) / 100`.
- **CSS `.difficulty-row` + `.diff-card`** (variants `.normal` dan `.hard`): hover lift + colored border/box-shadow + selected state. Cards lebih tinggi dari round-card karena ada title + subtitle + example.
- **State `game.difficulty`**: `'normal'` (default) atau `'hard'`. Di-reset ke `'normal'` saat MAIN LAGI.
- **Functions baru** (JS Block 5):
  - `goToDifficulty()`: sfx + sync selected state + show screen-difficulty.
  - `selectDifficulty(diff)`: set `game.difficulty` + toggle selected class.
- **Label updates**:
  - VS screen subtitle: "PERFECT TIMER · 3 RONDE" → "PERFECT TIMER · 3 RONDE · **NORMAL/HARD**".
  - Match result screen subtitle: sama format.

### Changed
- **Flow navigation**: `register → rounds → MULAI PERTANDINGAN` jadi `register → rounds → LANJUT PILIH MODE → difficulty → MULAI PERTANDINGAN`. Tombol di screen-rounds berubah label dari "MULAI PERTANDINGAN" → "LANJUT PILIH MODE", trigger `goToDifficulty()` (bukan `startMatch()` lagi).
- **`startNextRound()` target generator**: branch berdasarkan `game.difficulty` — `'normal'` → integer detik, `'hard'` → step 0.01 (sama logic awal).
- **`playAgain()`**: reset `game.difficulty = 'normal'` + sync selected card class di screen-difficulty (selain reset rounds yang sudah ada).

### Files
- `perfecttimer.html`:
  - CSS section 7 (extended dengan `.difficulty-row` + `.diff-card` variants).
  - HTML: screen-difficulty baru ditambah setelah screen-rounds. Tombol di screen-rounds rubah label + onclick. Label `<span id="vs-diff-label">` di VS subtitle + `<span id="final-diff-label">` di match-result subtitle.
  - JS Block 2 (state `difficulty` field), Block 5 (`goToDifficulty`, `selectDifficulty`, modifikasi `startMatch` set label + `startNextRound` branching generator), Block 12 (`playAgain` reset difficulty).
- `question.md`: tambah Q11 — difficulty selector NORMAL/HARD.
- `CHANGELOG.md`: entri ini.

### Preserved (NOT removed per rules.md)
- Generator HARD (random step 0.01) tidak dihapus — dipindahkan ke branch `else` di `startNextRound`, masih jalan persis sama untuk HARD mode.
- Round picker (1/3/5), VS animation, semua screen lain — unchanged.


## [2026-05-28] - NEW GAME: Perfect Timer (Timing Accuracy Duel) [AI / arahan Pradipta]
### Added
- **`perfecttimer.html`** (file baru, standalone — sama pattern dengan `bom.html`). Game 1v1 ketepatan waktu klik dengan flow:
  1. **Register** 2 nama (autosearch dari `guessRushLB`, sama UI pattern dengan Sambung Kata + Kaboom).
  2. **Pilih jumlah ronde**: 1, 3, atau 5 (ganjil — minim seri). Default 3.
  3. **VS animation** ("A vs B" cinematic slide-in + big VS pop).
  4. **Per ronde**:
     - Round intro: "RONDE X / Y" + target reveal (random 3.00-10.00 detik, format SS.HS).
     - P1 countdown 5 detik (5→4→3→2→1→GO!) dengan SFX beep.
     - P1 stopwatch jalan dari 00.00 naik (interval 10ms = 1 sentidetik per tick). Pemain klik tombol BIG STOP atau tekan SPASI untuk capture.
     - Auto-stop di 15.00 detik (akurasi 0%) kalau pemain telat klik.
     - P2 turn (countdown + stopwatch, **target sama** dengan P1 di ronde itu).
     - Round result screen: 2 card (P1 & P2) tampilin time clicked + selisih + akurasi % + label (PERFECT/AMAZING/GREAT/GOOD/MISS) + pemenang ronde.
  5. **Match result**:
     - Pemenang match = **best-of-rounds** (siapa menang lebih banyak ronde). Bukan total akurasi.
     - Round-by-round summary pills di bawah (R1: NAMA, R2: NAMA, dst).
     - Winner +200, loser +50. Seri (rare, total wins sama) → +150 / +150.
  6. **Leaderboard view** (same simplified throne battle pattern dari bom.html): top 30, podium gold/silver/bronze, count-up dual animasi winner+loser, scroll-to-winner.
  7. **MAIN LAGI** → balik ke screen register dengan **input nama dikosongkan** (project publik / pameran — tiap pair stranger harus input ulang, bukan prefill seperti bom.html).
  8. **MENYERAH** → lawan menang sisa ronde semua, langsung end-match.
- **Konstanta**: `TARGET_MIN = 3.0`, `TARGET_MAX = 10.0`, `STOPWATCH_MAX = 15.0`, `TICK_MS = 10`.
- **Formula akurasi**: `max(0, 100 - (|target - clicked| / target * 100))`, 2 desimal. PERFECT bila selisih ≤ 0.05s (5 sentidetik).
- **Visual effects unique untuk Perfect Timer**:
  - Round picker cards (1/3/5) dengan hover lift + selected gold border.
  - VS stage cinematic intro (P1 slide kiri + P2 slide kanan + big "VS" pop).
  - Round intro animasi (banner fade + num pop + target reveal dengan letter-spacing collapse).
  - Stopwatch big display monospace 4.6rem dengan danger flash kalau ≥ 13s.
  - Big STOP button (radius 22px, max-width 320px, :active translate 6px) — variant warna P1 (merah) / P2 (biru).
  - **PEAK EFFECT** (100% accuracy / selisih ≤ 0.05s): full-viewport gold flash + shockwave ring (120vmax expand) + banner slam "⭐ PERFECT! ⭐" + 2 gelombang confetti + chord SFX (C-E-G-C-E rising).
  - Accuracy label color escalation: PERFECT (gold) → AMAZING (cyan ≥95) → GREAT (green ≥85) → GOOD (yellow ≥70) → MISS (red).
  - Round result cards (P1/P2 side-by-side) dengan warna border per-pemain.
  - Round summary pills (R1: NAMA P1 / SERI / NAMA P2).
- **Input handling** (sesuai arahan Pradipta — "dilarang efek berat yang dapat menyebabkan delay"):
  - Stopwatch tick pakai `performance.now()` di first line setiap event handler → timestamp capture sebelum ada operasi visual.
  - Tombol STOP onclick: hard guard + capture elapsed di first 2 lines, baru update UI.
  - :active button transition pendek (0.05s ease-out) — efek tekan kerasa tanpa block JS.
  - Spacebar listener global dengan `document.addEventListener('keydown')` — preventDefault + stopTimer langsung.
  - Onclick di tombol = touchpad/mouse only (cursor wajib di tombol). Klik di area lain layar tidak register.
- **SFX library**: `click`, `flip`, `tick`, `beep` (countdown), `go` (GO!), `stop`, `peak` (5-note chord rising), `amazing`, `great`, `good`, `miss`, `win`.

### Changed
- **`index.html`** Modal MODE LAINNYA view 1 (game grid):
  - Card 3 (was SCRAMBLE locked, emoji 🎲) → **PERFECT TIMER** unlocked, thumbnail `assets/perfecttimer.png`, onclick → `openPerfectTimer()` navigate ke `perfecttimer.html`.
  - Card 4 (was SPEED QUIZ locked, emoji ⚡) → **TEBAK GAMBAR** (tetap LOCKED), thumbnail `assets/thumbnailtebakgambar.png`, no onclick (sesuai pattern locked cards).
  - Subtitle "1 ARENA BUKA · 3 SEGERA" → "**3 ARENA BUKA · 1 SEGERA**".
- **`openPerfectTimer()`** JS function baru di index.html: SFX flip + `window.location.href = 'perfecttimer.html'`. Same pattern dengan `openKaboom`.

### Game Logic Details (sesuai jawaban Pradipta di question.md)
- **Format SS.HS**: detik (00-30) titik sentidetik (00-99). Selalu 2 digit kiri . 2 digit kanan. Helper `fmtTime(sec)`.
- **Range target**: 3.00-10.00 detik random per ronde, step 0.01. P1 & P2 dapat target SAMA di ronde yang sama, tapi BEDA antar ronde.
- **Stopwatch arah**: NAIK dari 00.00 → max 15.00. Lewat 15s = auto-stop dengan akurasi 0%.
- **Pemenang ronde**: pemain dengan selisih `|target - clicked|` lebih kecil. Selisih sama persis (toleransi 0.0001s) → ronde tie (0 wins untuk dua-duanya).
- **Pemenang match**: best-of-rounds (`game.p1Wins` vs `game.p2Wins`). Sama = SERI match.
- **Surrender**: current player kalah, lawan dianggap menang semua ronde sisa (sampai `totalRounds`).
- **Main Lagi**: full state reset + clear input + balik ke screen-register. JANGAN prefill nama (project publik).

### Files
- `perfecttimer.html` (NEW): self-contained ~1100 baris (CSS sections 1-17 + HTML 9 screens + JS 12 blocks).
- `index.html`: HTML modal-modes view-1 (card 3 & 4 replaced dengan thumbnail images), JS Block 4 (`openPerfectTimer` function ditambah setelah `openKaboom`).
- `question.md`: rekap pertanyaan + jawaban final Pradipta (10 nomor).
- `CHANGELOG.md`: entri ini.

### Preserved (NOT removed per rules.md)
- Locked card pattern (lock icon corner + COMING SOON badge + no hover effects) — masih dipakai di card 4 (Tebak Gambar).
- Emoji icon system (`.game-icon` class) di CSS — masih ada untuk backward-compat, cuma sekarang tidak di-pakai di view-1 (semua card pakai thumbnail image). Future locked games bisa pakai emoji lagi kalau belum punya artwork.


## [2026-05-28] - NEW GAME: Kaching atau Kaboom! (5×5 Bomb Battle) [AI / arahan Pradipta]
### Added
- **`bom.html`** (file baru, standalone — bisa dihapus tanpa break index.html). Game 1v1 dengan flow:
  1. Register 2 nama (autosearch dari `guessRushLB` localStorage, sama mekanisme dengan Sambung Kata).
  2. Transition 5 detik (P2 berbalik badan, countdown big gold).
  3. P1 taro bom (1-5 bom bebas, flip animation tanah ↔ bom saat klik). Tombol SELESAI active dari 1 bom keatas.
  4. Transition 5 detik (P1 berbalik badan) → P2 taro bom (1-5 bom).
  5. Transition 5 detik (SIAP BERTANDING!) → mulai guessing phase.
  6. Guessing: 5×5 grid, alternating turn, 10 detik pick timer per turn.
     - Current player's own bombs **VISIBLE + LOCKED** (overlay "MILIKMU" badge, dim opacity) supaya ngga keklik sendiri.
     - Pick bom lawan → flip reveal bom.png + **EXPLOSION** (full-screen radial flash + screen shake + "💥 BOOM! 💥" text + boom SFX) + -1 nyawa.
     - Pick safe cell → flip reveal koin.png ATAU uang.png (random 50/50) + burst emoji float (🪙/💵) + sparkle ✨ + chime SFX.
     - Pick timer expire → auto-pick random non-own-bomb non-revealed cell.
  7. Game end conditions:
     - Nyawa pemain habis (≤0) → lawan menang.
     - Semua 25 kotak ke-reveal sebelum ada yang mati → cek sisa nyawa; sama = SERI, beda = pemain dengan nyawa terbanyak menang.
     - Surrender tombol MENYERAH → lawan auto-menang.
  8. Result handling:
     - **Win/loss**: result screen dengan winner (+200 gold card) + loser (+50). LB di-update langsung.
     - **SERI**: screen sendiri dengan 2 pilihan — MAIN LAGI (rematch, skip register, NO point update) atau AKHIRI (+150 each → masuk LB).
  9. Leaderboard view (in-file simplified throne battle): top 30, podium gold/silver/bronze, count-up dual animation winner+loser, scroll-to-winner. Sumber data: `guessRushLB` (unified dengan Guess Rush + Sambung Kata).
- **State `chain.isDraw`**: flag untuk routing SERI vs normal end. Set di `checkEndOrNext` saat semua cell revealed + lives sama.
- **CSS sections**: orbs (red+cyan untuk match bomb theme), grid 5×5, cell variants (bomb-placed/own-bomb/show-bomb/show-coin/show-money), cell-flip keyframes, pick-timer bar (3 state: normal/warning/danger), explosion-overlay + screen-shake, burst-emoji + sparkle, transition countdown, lives-bar dengan heart system, result/SERI result rows, simplified leaderboard styles.
- **Visual assets**: `assets/tanah.png` (unselect), `assets/bom.png` (bomb), `assets/koin.png` (coin reward), `assets/uang.png` (money reward), `assets/thumnailkachingataukaboom.png` (game card thumbnail), `assets/thumbnailsambungkata.png` (Sambung Kata thumbnail).

### Changed
- **`index.html`** Modal MODE LAINNYA view 1 (game grid):
  - Card 1: SAMBUNG KATA — emoji 🔤 → `<img>` thumbnail (`assets/thumbnailsambungkata.png`), Roblox-style.
  - Card 2: TEBAK GAMBAR (locked) → **KACHING ATAU KABOOM!** (unlocked, thumbnail image `assets/thumnailkachingataukaboom.png`, onclick → `openKaboom()` navigate to `bom.html`).
  - Card 3 & 4: SCRAMBLE + SPEED QUIZ (tetap locked, emoji).
- **CSS `.game-thumb`** baru: 1:1 aspect ratio + object-fit cover + border-radius 10px + shadow. Variant `.unlocked .game-thumb` (cyan border) dan `.locked .game-thumb` (grayscale brightness 0.5).
- **`openKaboom()`** JS function baru di index.html: SFX flip + `window.location.href = 'bom.html'`.

### Game Logic Details (sesuai jawaban Pradipta di question.md)
- **Bomb count 1-5**: Bebas per pemain, validasi tombol SELESAI active dari 1 bom.
- **Place undo**: Klik kotak bom → flip animation, kembali jadi tanah (same UX as place).
- **Own bomb saat pick**: Di-block, ngga bisa klik. Visible sebagai placeholder "MILIKMU" badge.
- **Auto-pick on timeout**: Sistem pilih random cell yang BUKAN bom sendiri (supaya pemain ngga ke-bom sendiri tanpa salahnya).
- **MAIN LAGI**: Skip register, restart match dengan nama sama (baik dari result win/loss, LB, atau SERI screen).
- **Unified LB**: All games (Guess Rush, Sambung Kata, Kaching atau Kaboom) share `localStorage.guessRushLB`.

### Files
- `bom.html` (NEW): self-contained game ~900 baris (CSS + HTML + JS).
- `index.html`: CSS section 11B (`.game-thumb`), HTML modal view 1 (card 1 & 2 pakai thumbnail), JS Block 4 (`openKaboom` function).
- `assets/`: 6 PNG baru (tanah, bom, koin, uang, thumnailkachingataukaboom, thumbnailsambungkata).
- `question.md` (NEW): rekap pertanyaan + default + jawaban Pradipta.
- `CHANGELOG.md`: entri ini.


## [2026-05-28] - MODE LAINNYA: Layout Horizontal Row (Card Sejajar Kepinggir) [AI / arahan Pradipta]
### Changed
- **`.game-grid`**: `flex-direction: column → row`. 4 card sekarang berjajar HORIZONTAL (sejajar ke samping), bukan vertikal ke bawah. Gap dikecilkan 10px → 8px supaya muat di modal 420px.
- **`.game-card`**: layout internal di-flip — `flex-direction: row → column`, `align-items: center`, `justify-content: space-between`, `text-align: center`. Konten sekarang stack: ICON (atas) → NAMA (tengah) → BADGE (bawah). `flex: 1; min-width: 0` supaya 4 card share equal width (~90-95px tiap card).
- **Entrance animation**: `translateX(-20px) → 0` (slide dari kiri) diganti `translateY(18px) scale(0.92) → 0` (pop dari bawah dengan elastic scale). Lebih cocok untuk row layout — cards "naik" ke posisi.
- **Hover unlocked**: `translateX(4px)` (slide kanan) → `translateY(-6px) scale(1.04)` (lift atas + scale). Standard "card pop" feel.
- **Font sizing diperkecil**: h3 `0.85rem → 0.65rem` (letter-spacing 1.2 → 0.5), badge `0.55rem → 0.48rem` padding `4px 10px → 3px 7px`, icon `2rem → 1.9rem`. Word-break: break-word ditambah ke h3 supaya nama panjang ("SAMBUNG KATA") wrap clean.
- **`.game-card p`** (description): `display: none`. Terlalu sempit di card ~90px lebar untuk text body — info description dihilangkan, tinggal icon + nama + badge.
- **Border-radius**: 16px → 14px (proporsional dengan card lebih kecil).

### Preserved (NOT removed per rules.md)
- `.game-text` wrapper di HTML masih ada (cuma sekarang container untuk h3 saja, p di-hide via CSS).
- View 2 (Casual/Chaos), tombol KEMBALI di view 1 & 2, semua locked state styling — unchanged.

### Files
- `index.html`: CSS section 11B (`.game-grid` direction, `.game-card` layout + animation + sizing, `.game-card p` hide).
- `CHANGELOG.md`: entri ini.


## [2026-05-27] - MODE LAINNYA: Polish Game Gallery Layout [AI / arahan Pradipta]
### Changed
- **`.game-grid`**: grid 2×2 → vertical list (`display: flex; flex-direction: column; gap: 10px`). 4 game card berjejer ke bawah, bukan 2-kolom.
- **`.game-card`**: layout horizontal dalam tiap card — `[icon (2rem) | text(h3+p) flex:1 | badge]`. Padding 14px 16px. Entrance animation `translateY scale` → `translateX(-20px) → 0` (slide-in dari kiri). Hover (unlocked only): `translateY(-6px) scale(1.04)` → `translateX(4px)` (slide right, lebih cocok list).
- **`.game-card .game-text`**: container baru wrap h3 + p, `flex: 1; min-width: 0` supaya text grow + truncate.
- **`.game-card.locked`** hover effects DIHAPUS: `locked-shake` di-`:hover` removed, `cursor` `not-allowed → default`. Coming soon cards sekarang STATIC — ngga ada animasi atau cursor change saat di-hover. (arahan Pradipta: "jangan ada hover buat coming soon")
- **Locked cards onclick** DIHAPUS: ngga lagi trigger popup "SEGERA HADIR!" karena ngga ada interaction sama sekali.
- **`showComingSoon(name)`** function DIHAPUS dari JS (ngga ada caller setelah onclick dihapus).
- **HTML card structure**: tambah `<div class="game-text">` wrap h3 + p untuk flex layout. Lock icon dipindah dari `font-size 1rem top:10px right:12px` → `font-size 0.75rem top:6px right:8px` (lebih subtle di pojok).

### Added
- **Tombol KEMBALI di view 1**: `<button class="modes-back-btn" onclick="closeModesModal()">← KEMBALI</button>` di top view-games. Tutup modal MODE LAINNYA → balik ke welcome screen. (arahan Pradipta: "kasih tombol back buat balik ke main menu")
- **`closeModesModal()`** (JS Block 4): tutup modal-modes dengan `display='none'` + SFX flip.

### Preserved (NOT removed per rules.md)
- `@keyframes locked-shake` definition di CSS — masih ada untuk backward-compat, cuma ngga di-apply via `:hover` lagi.
- View 2 (`#modes-view-chain`) + tombol KEMBALI di view 2 (back ke game grid) tetap unchanged.

### Files
- `index.html`: CSS section 11B (`.game-grid`, `.game-card` + variants), HTML modal-modes view 1 (back btn + card structure), JS Block 4 (`showComingSoon` dihapus, `closeModesModal` ditambah).
- `CHANGELOG.md`: entri ini.


## [2026-05-27] - MODE LAINNYA: Game Gallery (4 Cards) + Settings Modal + Combo Refactor [AI / arahan Pradipta]
### Added
- **CSS Section 11B — MORE MODES Game Grid + View Transitions**: Multi-view system di modal `#modal-modes`. `.modes-view` default hidden, `.modes-view.active` trigger `modes-view-in` (translateX + scale + blur fade) saat tampil. Grid 2×2 untuk `.game-card` dengan staggered entrance (`game-card-in` keyframes + animation-delay 0.10s/0.20s/0.30s/0.40s per nth-child).
- **`.game-card.unlocked`**: cyan border + glow + hover lift (translateY -6px scale 1.04) + shimmer sweep diagonal (`::before` pseudo dengan linear-gradient yang translateX dari -120% → 120% saat hover).
- **`.game-card.locked`**: saturate(0.3) brightness(0.7) + grayscale icon + lock icon (🔒) top-right + COMING SOON badge dengan `soon-pulse` keyframes (opacity 0.6 ↔ 1 + box-shadow red) + `locked-shake` keyframes saat hover (shake horizontal 5×).
- **HTML Modal multi-view**:
  - View 1 `#modes-view-games` (default): 4 game cards — SAMBUNG KATA (unlocked, 🔤, cyan) + TEBAK GAMBAR (locked, 🖼️) + SCRAMBLE (locked, 🎲) + SPEED QUIZ (locked, ⚡).
  - View 2 `#modes-view-chain`: tombol KEMBALI + dua card lama (Casual 30s / Chaos 10s). NOT REMOVED — dipreservasi sesuai rules.md.
- **`showGameGrid()`** + **`showChainSubModes()`** + **`showComingSoon(name)`** (JS Block 4): switch view via class toggle + `offsetWidth` reflow trick supaya entrance animation cards replay tiap switch. `showComingSoon` trigger popup orange "[NAME] - SEGERA HADIR!".
- **Modal Settings `#modal-settings`**: 1 opsi card (🏆 AKHIRI PERTANDINGAN gold) yang trigger `startCeremony()`. Diakses dari tombol ⚙️ PENGATURAN di welcome screen (btn-ghost kecil di bawah MODE LAINNYA).
- **`openSettings()`** + **`settingsEndMatch()`** (JS Block 8): buka modal-settings, lalu close + startCeremony saat card di-klik.
- **Combo system di Sambung Kata** (CSS 213+, JS 1467+): `chain.combo` counter saling-berbalas naik tiap submit valid, escalate 🔥 → 🔥🔥 → 🔥🔥🔥 (gray → orange ≥3 → red ≥8 → gold inferno ≥15) + floating "🔥 COMBO ×N" di atas current word + pitch SFX naik per combo. `chain.bestCombo` track maksimum, ditampilkan di result screen sebagai "🔥 COMBO TERTINGGI: N 🔥".
- **`spawnComboFloat`** + **`updateComboDisplay`** + **`bumpCombo`** (JS Block 7B2): animation helpers untuk combo UI.

### Changed
- **`openMoreModes`** (JS Block 4): tambah `showGameGrid()` di awal supaya modal selalu mulai dari view 1 tiap dibuka (ngga stuck di view 2 dari session sebelumnya).
- **Welcome screen** (HTML): tambah tombol ke-3 "⚙️ PENGATURAN" (btn-ghost kecil, font 0.7rem, padding 10px) di bawah MODE LAINNYA. Onclick → `openSettings()`.
- **Leaderboard tombol AKHIRI PERTANDINGAN** (HTML): ganti jadi **KEMBALI KE MENU** (`btn-ghost` cyan, onclick `backToMenuAfterCeremony`). Trigger ceremony dipindah ke modal Settings. Tombol di Sambung Kata game screen (`terminateMatch`) tetap.
- **`playAgain`** (JS Block 8): tambah branching berdasarkan `lastGameMode`. Kalau `'chain'` → `openChainRegister(chain.mode)` + prefill nama P1/P2 dari `chain.players`. Kalau `'guess'` → `initRound()` (perilaku lama). Sebelumnya selalu `initRound` → user dari chain mode keluar ke Guess Rush, bingung.
- **`lastGameMode`** state (JS Block 7): set ke `'guess'` di `initRound`, `'chain'` di `initWordChain`. Track mode terakhir untuk routing playAgain.
- **`chain` state**: tambah `combo: 0` + `bestCombo: 0` (reset di `initWordChain`).
- **`submitChain`**: increment combo + track best + spawn float + update display + bump animation tiap submit valid.
- **`endChainGame` → result screen**: tampilkan `chain.bestCombo` di `#chain-result-combo` (UI baru di screen-chain-result).
- **`chain-combo` UI** (HTML + CSS): dari bg + border + padding pill → minimalist text-only (`position: absolute; top: 100%`) anchored di bawah `chain-title` "MODE: 1V1". Ngga ngambil layout space → nama pemain ngga geser. Color escalation pure text-shadow.

### Removed
- **`endMatchFinal()`** (JS Block 8): function dihapus, dipindah ke `settingsEndMatch` (semantic baru: end match diakses via Settings, bukan tombol langsung di leaderboard).

### Preserved (NOT removed per rules.md)
- Card SAMBUNG KATA 1V1 - CASUAL & CHAOS (dengan badge 30 DETIK / 10 DETIK + deskripsi panjang) — dipindah ke view 2 `#modes-view-chain`, tetap utuh dengan onclick `openChainRegister`.
- `triggerJuaraPeak`, `#juara-banner`, `.trophy-emoji`, `.shake-climb`, `.rank-jump`, `.podium-celebrate` — semua CSS + JS function masih ada, untuk backward-compat.

### Files
- `index.html`:
  - CSS Section 11B baru (game-grid + game-card unlocked/locked + modes-view transitions + modes-back-btn).
  - CSS Section 10B: `.chain-combo` direstruktur (position: absolute, no bg/border).
  - HTML: `#modal-modes` direstruktur jadi 2 view, `#modal-settings` baru, welcome screen tambah tombol PENGATURAN, leaderboard tombol diganti.
  - JS Block 4: `openMoreModes` + 3 function navigasi baru.
  - JS Block 7B2 baru: combo helpers.
  - JS Block 8: `playAgain` branching + `openSettings`/`settingsEndMatch`.
- `CHANGELOG.md`: entri ini.


## [2026-05-27] - Sambung Kata: Count-Up Dual Animation di Leaderboard (Winner +200, Loser +50) [AI / arahan Pradipta]
### Added
- **`startThroneBattleChain(fullData, winnerName, loserName, oldWS, oldLS)`** (JS Block 9F): variant `startThroneBattle` khusus chain mode. Render leaderboard dengan OLD scores untuk winner & loser, lalu trigger `phaseCountUpDual` → reshuffle (LB sudah punya new scores) → podium.
- **`phaseCountUpDual(winnerName, wStart, wEnd, loserName, lStart, lEnd, done)`** (JS Block 9F2): animasi count-up DUAL — winner card animate dari `oldScore → oldScore+200` dengan floating "+200" gold, loser card animate dari `oldScore → oldScore+50` dengan floating "+50" green. Sync duration 1800ms supaya kedua animasi selesai bareng (ngga awkward kalau loser selesai duluan dengan delta lebih kecil).
- **`chain.lastLoser`** + **`chain.lastOldWinnerScore`** + **`chain.lastOldLoserScore`**: state baru di `endChainGame` untuk track OLD scores SEBELUM add points, supaya `chainViewLeaderboard` bisa animasi dari old → new.

### Changed
- **`endChainGame`**: track `oldWinnerScore` & `oldLoserScore` sebelum LB di-update. Simpan ke `chain.lastOld*Score` untuk diambil sama `chainViewLeaderboard`.
- **`chainViewLeaderboard`**: was panggil `startThroneBattle(lb, winner, score, score)` (delta 0 → no animation), sekarang panggil `startThroneBattleChain(lb, winnerName, loserName, oldWS, oldLS)` → keduanya pemain count-up bersamaan. Winner masih dapat `.player-focus` (highlight gold) + auto-scroll.

### Files
- `index.html`: JS Block 7C (`endChainGame` + `chainViewLeaderboard` modifikasi), JS Block 9F baru (`startThroneBattleChain`), Block 9F2 baru (`phaseCountUpDual`).
- `CHANGELOG.md`: entri ini.


## [2026-05-27] - Sambung Kata: Register 2-Player + Turn Color (P1 Merah / P2 Biru) + Auto-LB [AI / arahan Pradipta]
### Added
- **Screen `#screen-chain-register`**: register Pemain 1 + Pemain 2 dengan label color-coded (P1 merah, P2 biru), VS divider di tengah, autosuggest dropdown dari leaderboard (`handleChainNameInput` mirip `handleNameInput` di flow Guess Rush). Trigger via klik mode card di modal MODE LAINNYA (was: langsung initWordChain).
- **`openChainRegister(mode)`** + **`handleChainNameInput(val, playerNum)`** + **`selectChainName(name, playerNum)`** + **`startChainGame()`** + **`cancelChainRegister()`** — JS Block 7A.
- **Validasi register**: kedua nama wajib diisi, nama P1 & P2 ngga boleh sama (case-insensitive).
- **Turn-based color theme** (CSS Section 10B): saat giliran P1 → `#game-container.turn-p1` & `#screen-word-chain.turn-p1` (border + box-shadow + chain-word-big color + suffix-highlight color + chain-input border + p-tag.player-1.active + typing-status semuanya merah). Saat ganti ke P2 → swap ke biru. Semua pakai `transition: 0.6s` → smooth color mix saat turn change.
- **`applyChainTurnTheme()`** (JS Block 7B): dipanggil di `initWordChain` (set initial P1) dan setiap akhir `submitChain` (set ke giliran berikutnya).
- **`endChainGame(loserIdx)`** (JS Block 7C): winner = `1 - loserIdx`, accumulate ke LB (winner +200, loser +50). Kalau nama belum ada di LB → tambah entry baru; kalau sudah ada → tambah point ke existing. Simpan winner ke `chain.lastWinner` lalu show `#screen-chain-result`.
- **Screen `#screen-chain-result`**: card winner (gradient gold + glow) + card loser (subtle), tampilin nama + "+200 POIN" / "+50 POIN". Tombol LIHAT LEADERBOARD + KELUAR KE MENU.
- **`chainViewLeaderboard()`** + **`backToWelcomeFromChain()`**: leaderboard view pakai existing `startThroneBattle` dengan start==end (skip count-up animation), winner di-focus via player-focus class + auto-scroll.

### Changed
- **`initWordChain(mode, p1Name, p2Name)`**: tambah parameter nama. Set `chain.players = [p1Name, p2Name]`. Default fallback "Player 1"/"Player 2" kalau dipanggil tanpa nama (backward compat).
- **`chain` state**: tambah `players: ['Player 1','Player 2']` + `mode: 'casual'` + `lastWinner` (set saat endChainGame).
- **`renderChainPlayers`**: tag pakai nama dari `chain.players` + class `.player-1` / `.player-2` untuk color theming.
- **`updateChainUI`**: placeholder + status pakai nama actual (was hardcode "PLAYER N").
- **`submitChain`**: history pakai nama actual (was hardcode "P1"/"P2"), call `applyChainTurnTheme()` setelah switch turn.
- **`startChainTimer` time-out**: was `alert + location.reload`, sekarang panggil `endChainGame(chain.turn)` — current turn = loser karena ngga submit dalam waktu.
- **`terminateMatch`** (tombol AKHIRI PERTANDINGAN di chain screen): was `showFinalResult` (flow Guess Rush, ngga relevan), sekarang panggil `endChainGame(chain.turn)` — current turn dianggap menyerah → other wins.
- **Modal MODE LAINNYA card onclick**: was `initWordChain('casual'/'chaos')`, sekarang `openChainRegister('casual'/'chaos')`.

### Files
- `index.html`:
  - CSS Section 10B baru (register screen + turn-based color theme P1 merah/P2 biru + chain-result).
  - HTML: `#screen-chain-register` + `#screen-chain-result` baru, modal-modes onclick diubah.
  - JS Block 7 di-restruktur: 7 base, 7A (register flow), 7B (game with turn theme), 7C (end-game + LB update + result view).
  - JS Block 8: `terminateMatch` redirect ke `endChainGame`.
- `CHANGELOG.md`: entri ini.


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
