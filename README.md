# 0209_SewaTenda

**Aplikasi Penyewaan Perlengkapan Pendakian Berbasis Android**

[![Platform](https://img.shields.io/badge/Platform-Android-3DDC84?logo=android&logoColor=white)](#)
[![Bahasa](https://img.shields.io/badge/Bahasa-Kotlin-7F52FF?logo=kotlin&logoColor=white)](#)
[![Antarmuka](https://img.shields.io/badge/Antarmuka-Jetpack%20Compose-4285F4?logo=jetpackcompose&logoColor=white)](#)
[![Arsitektur](https://img.shields.io/badge/Arsitektur-MVVM-2F6B4F)](#)
[![SRS](https://img.shields.io/badge/SRS-v1.6-8B5E3C)](docs/SRS_SewaTenda_20240140209.pdf)
[![Status](https://img.shields.io/badge/Status-Dalam%20Pengembangan-B26B00)](#-log-progres-mingguan)

---

## 📖 Tentang Proyek

**SewaTenda** adalah aplikasi mobile untuk mengelola usaha penyewaan perlengkapan pendakian —
tenda, carrier, sleeping bag, matras, dan headlamp. Aplikasi ini digunakan oleh **pengelola (admin)**
untuk mencatat stok alat, data pelanggan, transaksi penyewaan, serta pengembalian beserta
**perhitungan denda keterlambatan secara otomatis**.

Proyek ini merupakan **Project Akhir mata kuliah Pengembangan Aplikasi Mobile (PAM) 2026**,
dibangun dengan **Jetpack Compose** dan arsitektur **MVVM**, terhubung ke **REST API** dengan
autentikasi **JWT** dan basis data **MySQL**.

| Keterangan | Isi |
|---|---|
| **Mata Kuliah** | Pengembangan Aplikasi Mobile (PAM) 2026 |
| **Jenis** | Tugas Akademik — Project Akhir |
| **Repositori** | `0209_SewaTenda` (4 digit NIM terakhir + nama project) |
| **Dokumen SRS** | [`docs/SRS_SewaTenda_20240140209.pdf`](docs/SRS_SewaTenda_20240140209.pdf) — v1.6, 31 halaman, Appendix A–E |

---

## ✨ Fitur Utama

| No | Modul | Deskripsi | Status |
|----|-------|-----------|--------|
| 1 | **Autentikasi** | Login dan register admin, Access Token (JWT, 15 menit) + Refresh Token (7 hari) | ⏳ |
| 2 | **Dasbor** | Ringkasan stok, transaksi aktif, keterlambatan, dan total denda | ⏳ |
| 3 | **Kategori Alat** | CRUDS kategori (tenda, carrier, sleeping bag, matras, headlamp) | ⏳ |
| 4 | **Unit Alat & Stok** | CRUDS unit alat, kondisi alat, harga sewa per hari, stok berkurang otomatis | ⏳ |
| 5 | **Pelanggan** | CRUDS data pelanggan beserta riwayat penyewaan | ⏳ |
| 6 | **Transaksi Penyewaan** | CRUDS transaksi, perhitungan biaya otomatis, struk PDF | ⏳ |
| 7 | **Pengembalian & Denda** | CRUDS pengembalian, denda keterlambatan otomatis, status pembayaran denda | ⏳ |
| 8 | **Laporan** | Rekap transaksi dan pendapatan per periode | ⏳ |
| 9 | **Profil** | Profil admin, logout, informasi aplikasi | ⏳ |
| 10 | **Penyimpanan Berkas** | Menyimpan dan membaca struk PDF di penyimpanan lokal perangkat | ⏳ |

> **Keterangan status:** ✅ selesai · 🔄 sedang dikerjakan · ⏳ belum dimulai

### Cakupan CRUDS

Setiap modul data wajib memiliki kelima operasi **CRUDS** (Create, Read, Update, Delete, Search)
sesuai pedoman Bagian D butir 6.

| Modul | Create | Read | Update | Delete | Search |
|-------|--------|------|--------|--------|--------|
| Kategori Alat | ✅ | ✅ | ✅ | ✅ | ✅ |
| Unit Alat & Stok | ✅ | ✅ | ✅ | ✅ | ✅ |
| Pelanggan | ✅ | ✅ | ✅ | ✅ | ✅ |
| Transaksi Penyewaan | ✅ | ✅ | ✅ | ✅ | ✅ |
| Pengembalian & Denda | ✅ | ✅ | ✅ | ✅ | ✅ |

*(Tanda ✅ menandakan kebutuhan sudah **dispesifikasikan** pada SRS §3.1–3.5, **didokumentasikan**
pada activity diagram SRS B.5–B.9, dan **dirancang** pada mockup 17 layar. Implementasi kode
menyusul.)*

---

## 🏗️ Arsitektur

Aplikasi menggunakan pola **MVVM** dengan pemisahan lapisan yang jelas. Tidak ada penyimpanan
lokal terstruktur (Room dilarang sesuai pedoman); satu-satunya input/output berkas adalah
**struk PDF**.

```mermaid
flowchart LR
    subgraph Android["📱 Android — Jetpack Compose"]
        V["Tampilan<br/>Layar Compose"] --> VM["ViewModel<br/>StateFlow"]
        VM --> R["Repositori"]
    end

    subgraph Jaringan["🌐 Jaringan"]
        RA["Retrofit + OkHttp"]
        AI["AuthInterceptor<br/>JWT bearer"]
        RA --- AI
    end

    subgraph Peladen["🖥️ Backend"]
        EX["REST API Express"]
        DB[("MySQL")]
        EX --> DB
    end

    R --> RA
    AI -->|"JSON / HTTPS"| EX
    VM -.->|"struk PDF"| FS["📄 Penyimpanan Lokal"]
```

### Alur Data

```
Antarmuka (Compose) → ViewModel (StateFlow<UiState>) → Repositori → Retrofit → REST API → MySQL
                          ↑                                                    │
                          └──────────── respons JSON ───────────────────────────┘
```

**Prinsip yang diterapkan:**

- **Satu sumber kebenaran** di `ViewModel` melalui `StateFlow` + `UiState`
  (Memuat / Berhasil / Gagal / Kosong)
- **Repositori** sebagai satu-satunya pintu akses data (tidak ada pemanggilan API dari lapisan UI)
- **AuthInterceptor** menyisipkan `Authorization: Bearer <access_token>` secara otomatis
- **Alur refresh token**: respons `401` → refresh → ulangi permintaan sekali → jika gagal, logout
- **Navigation Compose** untuk seluruh perpindahan layar (tanpa satu Activity per halaman)

---

## 🧰 Teknologi yang Digunakan

### Sisi Android

| Komponen | Teknologi |
|----------|-----------|
| Bahasa pemrograman | Kotlin |
| Antarmuka | Jetpack Compose + Material 3 |
| Navigasi | Navigation Compose |
| Manajemen status | ViewModel + StateFlow + Coroutines |
| Komunikasi jaringan | Retrofit 2 + OkHttp + Gson |
| Autentikasi | JWT (Access 15 menit, Refresh 7 hari) |
| Penyimpanan | Berkas lokal (struk PDF) — **tanpa Room** |
| Build | Gradle (Kotlin DSL) |

### Sisi Backend

| Komponen | Teknologi |
|----------|-----------|
| Runtime | Node.js + Express |
| Basis data | MySQL |
| Autentikasi | JWT (access + refresh token) |
| Format data | REST + JSON |

---

## 🗃️ Model Data

Lima entitas utama yang saling terhubung (rincian lengkap pada **SRS Appendix B** — ERD pada
B.2 dan activity diagram CRUDS pada B.5–B.9):

```mermaid
erDiagram
    KATEGORI_ALAT ||--o{ UNIT_ALAT : "memiliki"
    PELANGGAN    ||--o{ TRANSAKSI : "melakukan"
    UNIT_ALAT    ||--o{ TRANSAKSI : "disewa pada"
    TRANSAKSI    ||--o| PENGEMBALIAN : "menghasilkan"

    KATEGORI_ALAT {
        int id_kategori PK
        string nama_kategori
        string deskripsi
        string ikon
        bool aktif
    }
    UNIT_ALAT {
        int id_unit PK
        int id_kategori FK
        string nama_unit
        int stok
        string kondisi
        decimal harga_per_hari
    }
    PELANGGAN {
        int id_pelanggan PK
        string nama
        string no_telepon
        string alamat
        string catatan
    }
    TRANSAKSI {
        int id_transaksi PK
        int id_pelanggan FK
        int id_unit FK
        int jumlah
        date tanggal_sewa
        date tanggal_kembali
        decimal total_biaya
        string status
    }
    PENGEMBALIAN {
        int id_pengembalian PK
        int id_transaksi FK
        date tanggal_aktual
        int hari_terlambat
        decimal denda
        string status_denda
    }
```

**Aturan bisnis penting:**

- Stok berkurang saat transaksi dibuat, dan bertambah saat alat dikembalikan
- Denda = `hari_terlambat × (harga_per_hari × 20%)`, dihitung otomatis saat pengembalian dicatat
- Kategori atau pelanggan yang masih terhubung dengan data aktif tidak dapat dihapus
  (validasi referensial)

---

## 🔌 Ringkasan REST API

Kontrak lengkap (contoh permintaan/respons, kode status, dan pemetaan requirement) tersedia pada
**SRS Appendix D**. Ringkasannya sebagai berikut.

| Grup | Endpoint | Keterangan |
|------|----------|------------|
| **Autentikasi** | `POST /api/auth/register`<br/>`POST /api/auth/login`<br/>`POST /api/auth/refresh`<br/>`POST /api/auth/logout` | Registrasi, login, perpanjang token, logout |
| **Kategori** | `GET/POST /api/kategori`<br/>`GET/PUT/DELETE /api/kategori/:id` | CRUDS kategori alat (`?search=` untuk pencarian) |
| **Unit Alat** | `GET/POST /api/unit-alat`<br/>`GET/PUT/DELETE /api/unit-alat/:id` | CRUDS unit dan stok (`?search=`, `?kategori=`) |
| **Pelanggan** | `GET/POST /api/pelanggan`<br/>`GET/PUT/DELETE /api/pelanggan/:id` | CRUDS pelanggan (`?search=`) |
| **Transaksi** | `GET/POST /api/transaksi`<br/>`GET/PUT/DELETE /api/transaksi/:id` | CRUDS transaksi (`?search=`, `?status=`) |
| **Pengembalian** | `GET/POST /api/pengembalian`<br/>`GET/PUT/DELETE /api/pengembalian/:id` | CRUDS pengembalian dan denda |
| **Dasbor** | `GET /api/dashboard` | Ringkasan statistik |
| **Laporan** | `GET /api/laporan?dari=&sampai=` | Rekap per periode |

**Konvensi:** seluruh endpoint (kecuali autentikasi) memerlukan header
`Authorization: Bearer <access_token>`. Respons berhasil menggunakan kode `2xx` dengan isi JSON
`{ "success": true, "data": ... }`, sedangkan kegagalan menggunakan kode `4xx`/`5xx` dengan isi
`{ "success": false, "message": "..." }`.

---

## 📁 Struktur Repositori

```
0209_SewaTenda/
├── README.md
├── docs/
│   ├── SRS_SewaTenda_20240140209.pdf    # Software Requirements Specification v1.6 (31 halaman)
│   ├── UI_Preview.png                   # Mockup layar 01–10
│   └── UI_Preview_CRUDS.png             # Mockup layar 11–17 (form dan dialog)
│
├── app/                                 # 📱 Aplikasi Android
│   └── src/main/java/com/sewatenda/
│       ├── data/
│       │   ├── remote/                  # ApiService, AuthInterceptor, ApiClient
│       │   ├── repository/              # Implementasi repositori per modul
│       │   ├── model/                   # DTO permintaan dan respons
│       │   └── local/                   # FileStorageHelper (struk PDF)
│       ├── ui/
│       │   ├── navigation/              # NavGraph, Routes
│       │   ├── screens/                 # login, dasbor, kategori, unit, ...
│       │   ├── components/              # AppBar, SearchBar, EmptyState, dll.
│       │   └── theme/                   # Color, Type, Theme (hijau–coklat)
│       ├── viewmodel/                   # ViewModel + UiState per modul
│       └── util/                        # Formatter (Rupiah, tanggal), Session
│
└── backend/                             # 🖥️ REST API
    ├── src/
    │   ├── routes/                      # auth, kategori, unit, pelanggan, ...
    │   ├── controllers/                 # Logika tiap endpoint
    │   ├── models/                      # Query MySQL
    │   └── middlewares/                 # verifyToken, errorHandler
    ├── sql/
    │   └── schema.sql                   # 5 tabel + relasi + data awal
    ├── .env.example                     # Contoh konfigurasi
    └── package.json
```

---

## 🚀 Cara Menjalankan

### Prasyarat

- **Android Studio** Ladybug atau lebih baru (JDK 17)
- **Node.js** 18+ dan **MySQL** 8+
- Perangkat atau emulator Android dengan API 26+

### 1️⃣ Backend dan Basis Data

```bash
# Membuat basis data dan mengimpor skema
mysql -u root -p < backend/sql/schema.sql

# Menyiapkan konfigurasi
cd backend
cp .env.example .env        # sesuaikan DB_HOST, DB_USER, DB_PASS, JWT_SECRET

# Memasang dependensi dan menjalankan
npm install
npm run dev                 # default: http://localhost:3000
```

Uji cepat: `GET http://localhost:3000/api/health` → `{ "success": true }`

### 2️⃣ Aplikasi Android

1. Buka folder `app/` di Android Studio
2. Sesuaikan base URL pada `app/build.gradle.kts`:
   - Emulator → `http://10.0.2.2:3000/`
   - Perangkat fisik → `http://<IP-LAN-komputer>:3000/`
3. Lakukan sinkronisasi Gradle, lalu tekan **Run** ▶️
4. Login menggunakan akun awal dari `schema.sql`
   (default: `admin@sewatenda.id` / `admin123`)

---

## 📊 Log Progres Mingguan

Sesuai pedoman Bagian C, README ini diperbarui **setiap minggu** sebagai laporan progres.
Riwayat commit dijaga **bertahap** (bukan satu kali push di akhir).

| Minggu | Periode | Capaian | Status |
|--------|---------|---------|--------|
| 1 | 17 – 23 Nov 2026 | Analisis kebutuhan; **SRS v1.5** selesai (23 halaman, Appendix A–E, spesifikasi API, matriks kepatuhan) | ✅ |
| 2 | 24 – 30 Nov 2026 | Mockup antarmuka **17 layar** (10 layar utama + 5 form CRUDS + detail struk + dialog hapus); penyiapan repositori dan struktur folder; **revisi SRS v1.6** — activity diagram CRUDS Appendix B.5–B.9 (31 halaman) | 🔄 |
| 3 | 1 – 7 Des 2026 | Skema MySQL 5 tabel; backend autentikasi (JWT + refresh) dan modul Kategori | ⏳ |
| 4 | 8 – 14 Des 2026 | Backend modul Unit Alat, Pelanggan, Transaksi, dan Pengembalian | ⏳ |
| 5 | 15 – 21 Des 2026 | Kerangka Android: tema, navigasi, Retrofit, login, dan dasbor | ⏳ |
| 6 | 22 – 28 Des 2026 | Android 5 modul CRUDS (daftar, form, pencarian, hapus) | ⏳ |
| 7 | 29 Des 2026 – 4 Jan 2027 | Penyimpanan berkas struk PDF; validasi form; kondisi galat dan data kosong | ⏳ |
| 8 | 5 – 11 Jan 2027 | Pengujian menyeluruh, perbaikan bug, persiapan demo dan presentasi | ⏳ |

> Target tanggal dapat disesuaikan dengan jadwal presentasi.

---

## 🎨 Desain Antarmuka

Tema **hijau hutan dan coklat tanah** — merepresentasikan suasana alam pendakian.

| Warna | Kode Hex | Penggunaan |
|-------|----------|------------|
| Hijau utama | `#2F6B4F` | App bar, tombol primer, aksen aktif |
| Hijau gelap | `#1F4D38` | Gradasi, elemen sekunder |
| Coklat | `#8B5E3C` | Aksen denda, badge peringatan |
| Krem | `#F7F3EC` | Latar halaman |
| Tinta | `#2A2622` | Teks utama |

Ketujuh belas layar mockup mencakup: Splash, Login, Register, Dasbor, Kategori Alat,
Unit Alat & Stok, Pelanggan, Transaksi Penyewaan, Pengembalian & Denda, Profil,
5 form CRUDS, Detail Transaksi & Struk, serta Konfirmasi Hapus.

---

## 🗺️ Rencana Pengembangan

- [x] Analisis kebutuhan dan penyusunan SRS (v1.6)
- [x] Activity diagram CRUDS — Appendix B.5–B.9
- [x] Mockup antarmuka 17 layar
- [ ] Skema MySQL dan backend REST API
- [ ] Android: autentikasi JWT dan refresh token
- [ ] Android: dasbor dan 5 modul CRUDS
- [ ] Penyimpanan berkas struk PDF
- [ ] Pengujian dan demo presentasi

---

## 📄 Lisensi dan Catatan Akademik

Proyek ini dibuat untuk keperluan **akademik** (Project Akhir PAM 2026) dan tidak untuk
distribusi komersial. Dokumen SRS, desain antarmuka, dan kode sumber merupakan karya
mahasiswa penulis.

**Rujukan pedoman:** *Pedoman Project Akhir Pengembangan Aplikasi Mobile 2026* —
Bagian C (ketentuan repositori), Bagian D (persyaratan minimal fitur), Bagian F (penilaian).

---

## 👤 Penulis

| Keterangan | Isi |
|---|---|
| **Nama** | Andy Zaky Zein |
| **NIM** | 20240140209 |
| **Kelas** | C |
| **Program Studi** | Teknologi Informasi |
| **Mata Kuliah** | Pengembangan Aplikasi Mobile (PAM) 2026 |
| **Dosen Pengampu** | _\<diisi\>_ |
| **NIDN** | _\<diisi\>_ |

---

<p align="center">
  <sub>© 2026 — SewaTenda · Project Akhir Pengembangan Aplikasi Mobile</sub>
</p>
