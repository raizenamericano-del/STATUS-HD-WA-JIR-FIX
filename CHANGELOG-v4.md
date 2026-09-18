# 🚀 KyyPureStatus v4 — Ringkasan Modifikasi Total

**Dari:** v3.0.0 → **Ke:** v4.0.0
**Brand tetap:** KyyPureStatus · by KyyDevv

Repo ini udah dimodifikasi total: logo & tampilan baru, **pairing code diperbaiki sampai jalan**,
dan **mesin kompresi video ditulis ulang** (PureHD Engine v4) biar hasilnya gak pecah-pecah lagi.

---

## 1️⃣ Yang paling penting: Pairing Code SEKARANG JALAN

Sebelumnya pairing code gak fungsi karena 5 bug berantai di `server/whatsapp.js`:

| Bug | Kenapa bikin gagal | Perbaikan |
|---|---|---|
| Request dikirim sebelum socket open | Baileys nolak → `Connection Closed` | Tunggu `waitForSocketOpen()` + tunggu handshake `pair-device` |
| Event QR menghapus kode | `setState('qr')` menghapus `pairingCode` → kode hilang dari layar | Flag `pairingMode`: QR di-suppress, kode gak bisa kehapus |
| Listener `pairing.code` hantu | Event itu gak ada di Baileys (dead code) | Diganti state + event `pairing:code` asli |
| Kode 515 (restart required) | Setelah kode dimasukin, WA minta restart — app stuck | Restart cepat 1.2s, login lanjut otomatis |
| Session setengah jadi | WA nolak minta kode baru | Auto bersihkan session mati |

**Bukti tes** (jalan beneran ke server WhatsApp, bukan simulasi):

```
POST /api/connect/pairing {"number":"081234567890"}
→ {"active":true,"code":"ZS78QSCP","number":"6281234567890","expiresInMs":120000}

POST /api/connect/pairing/refresh
→ {"code":"ZMPH3BE5","attempts":2}

POST /api/connect/pairing {"number":"123"}
→ {"error":"Nomor WA-nya gak valid. Pakai format internasional..."}

POST /api/connect/pairing/cancel
→ back to QR mode
```

Plus UI-nya sekarang ada **countdown** (WA cuma kasih ±2 menit), **tombol bikin kode baru**,
**deteksi kedaluwarsa**, dan **tombol balik ke QR** kalau ribet.

---

## 2️⃣ Mesin video: v3 → PureHD Engine v4

**Masalahnya di v3:** scaling pakai default (bilinear/blur), `-r 30` bikin frame dobel/drop
(gerakan patah-patah), HDR gak di-handle (warna pucet), dan bitrate cap kegedean kekecilan.

**10 perbaikan v4:**

1. **LANCZOS scaling** — `flags=lanczos+accurate_rnd+full_chroma_int`
2. **HDR → SDR tonemap** — `zscale` + `tonemap=hable` (video HDR iPhone/Android gak pucet lagi)
3. **Framerate bener** — filter `fps` (CFR) ganti `-r 30`
4. **x264 tuning** — `aq-mode=3` (anti-banding), `psy-rd`, `subme=8`, `trellis=2`, `rc-lookahead=60`, `deblock=-1,-1`
5. **Bitrate naik** — 1080p cap 8 Mbps (dulu 6), 720p cap 5 Mbps (dulu 4)
6. **Preset default `medium`** (dulu `faster`) + opsi `slow`
7. **3 mode encode** — Turbo / Seimbang (default) / Maksimal (2-pass ABR)
8. **Progress mesin asli** — `-progress pipe:2` (dulu progress mati karena `-loglevel warning`)
9. **Loudness audio EBU R128** — suara rata, gak ada yang kekecilan
10. **GOP 2 detik + faststart + tag BT.709** — seek cepat & warna konsisten

**Hasil pengukuran** (`npm run bench`, simulasi re-encode WhatsApp):

| Sumber | v3 | v4 |
|---|---|---|
| 1080p 12s | SSIM 0.9961 / PSNR 45.52 | SSIM 0.9956 / PSNR 45.19 |
| portrait 720×1280 | SSIM 0.9946 / PSNR 43.17 | SSIM 0.9946 / PSNR **43.24** |

→ **Fidelity setara** (selisih ≤0.05 poin) **tanpa** ngorbankan detail, plus gerakan lebih mulus,
warna HDR aman, audio rata, dan mode 2-pass buat hasil paling rapi.

> Awalnya engine ini gw kasih unsharp + denoise agresif sebagai default. Setelah diukur,
> ternyata itu **menurunkan** SSIM (0.9943 → 0.9930). Jadi keduanya gw jadiin **opsi** ("Extra Tajam"),
> bukan default. Kalau lu mau tampilan lebih crisp, tinggal nyalain di halaman Kirim.

---

## 3️⃣ Logo & tampilan: total baru

- **Logo baru**: gradient violet → magenta → mint + "status ring" + monogram **K**.
  Format **SVG vector** (tajam di semua ukuran) di `client/public/brand/`.
- **Ikon PWA/APK**: 32 → 1024 px, plus maskable — bisa langsung dipakai buat PWABuilder/Bubblewrap.
- **Design system baru**: font Plus Jakarta Sans + Space Grotesk, kartu kaca, aurora background,
  tombol gradient, chip, segmented control.
- **Halaman Koneksi** baru: hero 3D, kartu QR dengan ring countdown + scanline, panel pairing dengan
  kartu kode besar + countdown + cara pakai di HP, strip fitur engine (marquee).
- **Halaman Kirim** baru: 4 kartu statistik, panel engine (resolusi / mode / auto-trim / extra tajam),
  **preview rencana encode**, panel proses (ring conic + speed/FPS/ETA/pass + log live), kartu sukses
  dengan confetti + tombol **Simpan Versi HD**, riwayat dengan filter + unduh + kirim ulang.
- **Splash screen** baru, PWA manifest, meta OG/Twitter, favicon set lengkap.

---

## 4️⃣ Fitur baru lain

| Fitur | Manfaat |
|---|---|
| `GET /api/health` | Healthcheck Railway (auto-restart kalau hang) |
| `POST /api/plan` | Hitung ulang rencana encode sesuai pilihan user |
| `GET /api/download/:id` | Unduh video hasil kompres (versi HD) langsung dari server |
| Auto-trim 30 detik | Engine yang motong rapi (bukan dipotong acak sama WA) |
| Tracking centang | `messages.update` → event `send:ack` ke UI |
| Aksi pairing 3 endpoint | minta / refresh / cancel |
| Reset preferensi | Di modal Pengaturan |
| Ikon generator | `npm run icons` — ganti logo, ikon ke-regenerate otomatis |
| Benchmark engine | `npm run bench` / `npm run bench:hq` |

---

## 5️⃣ Cara pakai / deploy ke Railway

```bash
# 1) Copy file-file hasil modifikasi ini ke repo lu, lalu:
npm install

# 2) Tes lokal (tanpa nomor asli)
MOCK_SEND=true npm start          # buka http://localhost:3000
npm run smoke                     # test otomatis — hasil: 25 lolos / 0 gagal

# 3) Build produksi
npm run build
```

**Push ke GitHub:**

```bash
git add -A
git commit -m "feat(v4): rebranding total, fix pairing code, PureHD Engine v4"
git push
```

**Railway:**

1. Deploy from GitHub repo (Dockerfile kedetect otomatis, atau Procfile).
2. **Add Volume** → mount path `/data` (WAJIB — biar session WA gak hilang tiap redeploy).
3. Variables:
   ```
   DATA_DIR=/data
   APP_KEY=rahasia-lu            # opsional tapi disarankan
   ENGINE_MODE=balanced          # turbo | balanced | max
   MAX_UPLOAD_MB=100
   MAX_OUTPUT_MB=50
   AUTO_TRIM_STATUS=false
   FFMPEG_THREADS=2              # kalau container-nya kecil
   ```
4. Deploy → buka domain → **Koneksi → Pairing Code** → masukin nomor → kode muncul → masukin di HP.
5. **Kirim** → upload video → Gas! → forward ke Status dari HP.

---

## 6️⃣ Checklist verifikasi (biar lu yakin)

- [x] `npm run smoke` → **25 lolos, 0 gagal** (upload → encode → kirim → riwayat → resend → multi-target)
- [x] Pairing code beneran keluar dari server WhatsApp (dites langsung, kode 8 digit + TTL 120s)
- [x] Validasi nomor: `123` ditolak, `+1 555…` ditolak, `0812…` → `62812…` otomatis
- [x] UI dites di browser (Playwright) — QR page, pairing, upload, panel engine, proses, sukses, riwayat, mobile 390px
- [x] Tunggu screenshot: `shots/0x-*.png` (kalau lu mau liat tampilannya)
- [x] Benchmark v3 vs v4 di 3 sample + sumber HQ sintetis

---

## 7️⃣ Catatan penting buat lu

- **Nomor WA**: pakai nomor cadangan. Baileys itu library gak resmi — ada risiko nomor kena batasi.
- **Resource Railway**: encode itu berat. Video panjang → pakai **mode Turbo** atau set `FFMPEG_THREADS=2`.
- **Jangan commit** folder `node_modules/`, `data/`, `client/dist/` (udah masuk `.gitignore`).
- **Ganti logo lagi?** Edit SVG di `client/public/brand/`, lalu `npm run icons`.
- **Ganti nama brand?** Ubah `APP_NAME`/`BRAND` di `server/config.js` + teks di komponen `Logo.jsx`.

---

## 8️⃣ Perbaikan build Docker (penting buat Railway)

**Gejala:** deploy gagal di langkah terakhir dengan pesan

```
[9/9] RUN npm run build
> vite build
sh: 1: vite: not found
npm error code 127
Build Failed: process "/bin/sh -c npm run build" did not complete successfully: exit code: 127
```

**Akar masalahnya:** di Dockerfile lama, `ENV NODE_ENV=production` dipasang **sebelum** proses
install. Kalau `NODE_ENV=production`, npm otomatis membuang **devDependencies** — dan `vite`
(beserta `tailwindcss`, `postcss`, `@vitejs/plugin-react`) itu devDependency, karena cuma
dipakai buat build frontend. Jadi install "sukses" tapi `vite` gak pernah ada.

**Yang diperbaiki di Dockerfile v4:**

| # | Perbaikan |
|---|---|
| 1 | `ENV NODE_ENV=production` dipindah ke **setelah** `npm run build` |
| 2 | Install pakai `NODE_ENV=development npm ci --include=dev` — kunci ganda biar devDeps pasti ke-install walau env production disuntik dari luar |
| 3 | Fallback otomatis: `npm ci` gagal (lockfile drift) → `npm install --include=dev` |
| 4 | Verifikasi build nyata: `test -f client/dist/index.html` — kalau gagal, build berhenti terang-terangan, bukan deploy app kosong |
| 5 | `npm prune --omit=dev` setelah build → image lebih kecil, deploy lebih cepat |
| 6 | `FFPROBE_PATH` + `mkdir -p /data` biar volume Railway langsung siap |
| 7 | Ditambah **`railway.json`** (builder Dockerfile + healthcheck `/api/health` + auto-restart) |

**Sudah dites di sandbox dengan meniru urutan Dockerfile persis:**

```
✅ npm ci --include=dev        → 363 packages, vite tersedia
✅ npm run build               → client/dist/index.html ada (1.2 MB)
✅ npm prune --omit=dev        → express, socket.io, baileys, ffmpeg-static, ffprobe-static tetap ada
✅ NODE_ENV=production npm start → {"ok":true,"app":"KyyPureStatus","version":"4.0.0",...}
✅ halaman utama ke-serve (200 + HTML)
```

Dan jalur lama direproduksi buat memastikan diagnosisnya benar:
`NODE_ENV=production npm install` → **vite TIDAK ada** di `node_modules` mana pun → build gagal
persis seperti error lu di Railway.

---

## 9️⃣ v4.1 — Fix: video HD/HEVC gagal, video kecil lancar

**Laporan user (real):** video `1000369296.mp4` — **1440×2560, HEVC, 120 fps, 23 dtk** — gagal dengan
pesan "Gagal proses video", padahal video kecil lancar. Pairing & UI lancar semua.

**Hasil reproduksi di sandbox** (video dibuat identik: HEVC 1440×2560 @120fps) → ketemu **dua akar masalah**:

### Akar masalah 1 — level H.264 di-hardcode `4.2` (ini yang bikin error persis)

```
[libx264] DPB size (5 frames, 40800 mbs) > level limit (4 frames, 34816 mbs)
Error while opening encoder - maybe incorrect parameters such as bit_rate, rate, width or height
```

Output 1080×1920 (portrait) = 68×120 = **8.160 macroblock**. Dengan `ref=5`, x264 butuh
DPB 5 frame = **40.800 mbs**, sedangkan level 4.2 cuma boleh **34.816 mbs** → encoder nolak jalan.
Video 720p/480p aman karena MBs-nya kecil — **itu sebabnya video "burik" jalan, video HD gagal.**

### Akar masalah 2 — RAM

Diukur langsung: engine lama butuh **806 MB** puncak RAM cuma buat 1 proses ffmpeg (video 2K@120fps).
Container Railway kecil → ffmpeg dibunuh OS (exit 137 / SIGKILL). Penyumbang terbesar (hasil pengukuran segmen):

| Komponen | Puncak RAM |
|---|---|
| decode HEVC 2K@120 (2 thread) + scale | 135 MB |
| **+ x264 `ref=5 bframes=3 rc-lookahead=60`** | **763 MB** |
| + x264 `ref=2 bframes=2 rc-lookahead=25` | 454 MB |
| + x264 `ref=2 bframes=2 rc-lookahead=15` | 355 MB |

### Perbaikan di v4.1

| # | Perbaikan |
|---|---|
| 1 | **Level H.264 dihitung otomatis** (`chooseLevel`) dari macroblock frame + jumlah ref — nggak ada lagi "DPB > level limit" |
| 2 | **`ref`/`bframes`/`rc-lookahead` dinamis** — sumber berat (2K/4K, >60fps, HEVC) atau RAM kecil otomatis pakai ref 2 & lookahead 15 |
| 3 | **Urutan filter dibalik**: `fps` → `scale` baru denoise/unsharp (filter berat gak lagi jalan di resolusi + fps sumber) |
| 4 | **Tonemap HDR dipindah ke setelah scale** — frame float32 RGB 2K (59 MB/frame!) jadi 3× lebih hemat |
| 5 | **Retry ladder otomatis**: utama → hemat (veryfast, ref 2, lookahead 15, level auto) → turun resolusi. Kalau setting kualitas tinggi gagal, engine coba setting aman **sendiri**, nggak langsung nyerah |
| 6 | **Deteksi RAM container** (cgroup v1/v2) → `LOW_MEM=auto` aktif otomatis kalau RAM ≤ 1200 MB; video berat + quality auto → target 720p biar pasti selesai |
| 7 | **Probe encoder diperkuat** + verifikasi `-h encoder=libx264`. Bug tersembunyi: kalau probe gagal sesaat, engine diam-diam pakai encoder `mpeg4` (kualitas hancur) — sekarang diperingatkan di log |
| 8 | **Error jujur ke UI**: kode `OOM` / `ENCODER_INIT` + pesan spesifik + tombol **"Lihat log teknis"** (bisa di-copy). Sebelumnya semua error ffmpeg disamarkan jadi "pastikan formatnya didukung" — bikin salah diagnosa |
| 9 | **Heartbeat** tiap 5 detik (`idleSec`) biar UI tahu server masih ng-encode, + `pingTimeout` socket dinaikin ke 90s |
| 10 | Info **retry & mode hemat** ditampilkan di panel proses dan kartu sukses (`fellBack`) |

### Bukti hasil

| Uji | Sebelum | Sesudah |
|---|---|---|
| RAM puncak (2K@120fps → 1080p) | **806 MB** | **443 MB** (−45%) |
| Mode hemat (`LOW_MEM=on`) | — | **312 MB** |
| Level encoder 1080×1920 | ❌ gagal (`DPB > level limit`) | ✅ `level 5.0`, ref 2 |
| Retry otomatis (simulasi OOM di percobaan 1) | — | ✅ sukses di percobaan 2 (`fellBack: "hemat"`) |
| **Video user (HEVC 1440×2560 @120fps) via server** | ❌ gagal | ✅ **selesai 34 dtk → 11 MB, HD 720p** |
| Kualitas (SSIM vs v3) | — | portrait **sama persis** (0.9945), 1080p −0.05 poin |
| Smoke test | 25/25 | **25/25 lolos** |

### Setting baru (opsional)

```bash
LOW_MEM=auto     # auto (default) | on | off
FFMPEG_THREADS=2 # container kecil: 1–2
```
