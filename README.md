# 🎓 CampusFinder — Lost & Found Kampus

Aplikasi mobile untuk melaporkan & mencari barang hilang/ditemukan di area kampus.
Dibangun dengan **React Native (Expo + expo-router + TypeScript)** dan **Firebase
(Firestore Database + Storage)**.

## ✨ Fitur

- Beranda katalog visual: grid 2 kolom, toggle **Barang Ditemukan / Barang Dicari**,
  pencarian kata kunci, filter lokasi gedung (chips), data realtime dari Firestore.
- Form lapor barang: pilih jenis laporan, ambil foto (kamera/galeri),
  kategori, lokasi, tanggal, deskripsi, nomor WhatsApp, dan
  **pertanyaan verifikasi** anti-klaim-palsu.
- Halaman detail: foto penuh, info lengkap, tombol **Hubungi via WhatsApp**,
  dan aksi **Tandai Selesai**.

## 📁 Struktur Folder

```
campus-finder/
├── .env                    # konfigurasi Firebase (isi milikmu!)
├── .env.example            # template konfigurasi
├── firestore.rules         # aturan keamanan Firestore (siap deploy)
├── app.json                # konfigurasi proyek Expo
└── src/
    ├── app/                # LAYER UI (file-based routing oleh expo-router)
    │   ├── _layout.tsx     # navigasi Stack + pembungkus aplikasi
    │   ├── index.tsx       # Halaman Beranda (grid + filter + search)
    │   ├── report.tsx      # Form Lapor Barang (+ upload foto)
    │   └── item/[id].tsx   # Halaman Detail Barang (+ tombol WhatsApp)
    ├── components/         # komponen UI yang dipakai ulang
    │   ├── ItemCard.tsx    # kartu barang untuk grid
    │   └── Segmented.tsx   # toggle/tab ala segmented control
    ├── services/           # LAYER DATA (satu-satunya tempat memanggil Firebase)
    │   ├── firebase.ts     # inisialisasi app, Firestore, Storage
    │   └── items.ts        # subscribe, tambah, upload foto, update status
    ├── types/
    │   └── item.ts         # skema data Item (peta koleksi `items`)
    └── constants/
        └── app.ts          # palet warna + helper format tanggal
```

Prinsip yang dipakai: **UI (app/ + components/)** tidak pernah menyentuh
Firebase langsung — semua lewat **service layer (services/)**. Ini membuat
kode mudah diuji dan diganti backend bila diperlukan.

## 🗄️ Skema Database (koleksi `items`)

| Field | Tipe | Keterangan |
|---|---|---|
| id | string | id dokumen (otomatis Firestore) |
| title | string | nama barang |
| type | `'lost'` \| `'found'` | jenis laporan |
| category | string | kategori barang |
| imageUrl | string | URL foto di Firebase Storage |
| location | string | lokasi spesifik |
| date | Timestamp | tanggal kejadian |
| description | string | ciri-ciri khusus |
| contactNumber | string | nomor WhatsApp (format 62xxx) |
| verificationQuestion | string? | pertanyaan verifikasi (opsional) |
| status | `'active'` \| `'resolved'` | status laporan |
| createdAt | Timestamp | waktu laporan dibuat |

## 🚀 Langkah Menjalankan dari Nol

### 1. Siapkan Firebase (±10 menit, sekali saja)

1. Buka https://console.firebase.google.com → **Add project** (nama bebas, mis. `kampus-finder`).
2. Di halaman proyek, klik ikon **Firestore Database** → **Create database** →
   pilih lokasi terdekat → **Start in test mode**.
3. Di halaman proyek, klik ikon **Storage** → **Get started** → **Next** (test mode).
4. Kembali ke **Project Settings** (ikon gear) → tab **General** →
   bagian **Your apps** → klik ikon `</>` (Web) → daftarkan app
   (nickname: `campus-finder-web`) → salir 6 nilai config (apiKey, dst).
5. Tempelkan 6 nilai itu ke file **`.env`** di root project (ganti placeholder).

### 2. Install dependencies & jalankan

```bash
cd campus-finder
npm install
npx expo start
```

### 3. Jalankan di HP (paling mudah untuk pemula)

1. Install aplikasi **Expo Go** dari Play Store / App Store di HP kamu.
2. Pastikan HP dan laptop **satu jaringan WiFi**.
3. Buka terminal, jalankan `npx expo start`, lalu **scan QR code** dengan Expo Go.
4. Aplikasi langsung terbuka di HP (hot-reload: kode berubah → layar ikut berubah).

### 4. Jalankan di Emulator

- **Android:** `npx expo start --android` (butuh Android Studio + emulator aktif).
- **iOS (khusus macOS):** `npx expo start --ios`.

> Catatan kamera: di emulator, "Kamera" memakai kamera virtual; di HP asli
> jangan lupa izinkan akses Kamera & Galeri saat diminta.

### 5. (Opsional) Deploy aturan keamanan Firestore

Buka Firebase Console → Firestore → **Rules** → tempel isi `firestore.rules` → **Publish**.
Tanpa ini, database tetap jalan dalam mode test mode (terbuka 30 hari sejak dibuat).

## 🧪 Cara Mencoba Alurnya

1. Buka Beranda → tekan **＋ Lapor Barang**.
2. Pilih **Saya Menemukan** → ambil foto → isi data → **Kirim Laporan**.
3. Kembali ke Beranda — kartu baru muncul **tanpa refresh** (realtime).
4. Buka kartunya → tekan **Hubungi Pelapor via WhatsApp**.
5. Setelah barang kembali: **Tandai Selesai** → badge berubah hijau.

## 📦 Distribusi APK (agar semua orang bisa pakai)

APK = aplikasi Android jadi (`.apk`) yang tinggal di-install teman kamu,
tanpa perlu Expo Go atau satu WiFi dengan kamu.

```bash
cd campus-finder
npx eas login              # sekali saja, pakai akun expo.dev (gratis)
npx eas build -p android --profile preview
```

1. Tunggu build cloud (±10–20 menit), dapat link download APK.
2. Kirim link / file APK ke teman → mereka buka → Install (izinkan
   "Install dari sumber tidak dikenal" untuk Android).

> **Penting:** nilai di `.env` (Firebase config) dibakar ke dalam APK
> saat build. Jadi **isi `.env` dulu, baru build** — kalau placeholder,
> aplikasi terbuka tapi tidak ada data.

## ⚠️ Catatan Teknis

- Query Firestore memakai `orderBy('createdAt')` lalu filtering jenis/pencarian
  dilakukan di sisi client — agar tidak perlu membuat composite index (ramah pemula).
  Untuk data besar (ribuan laporan), pindahkan filter ke query `where()` + buat index.
- `EXPO_PUBLIC_*` memang aman dibundle karena API key Firebase hanya "pintu gerbang";
  keamanan sesungguhnya ditentukan `firestore.rules` dan aturan Storage.
- Nomor WhatsApp pelapor disimpan apa adanya; aplikasi otomatis menambahkan
  prefix `62` saat membuka `wa.me`.
