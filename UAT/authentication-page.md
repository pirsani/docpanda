# Rencana Uji Penerimaan Pengguna (UAT): Halaman Autentikasi

## Tujuan

Tujuan dari UAT ini adalah untuk memvalidasi fungsionalitas, kegunaan, dan kinerja Halaman Autentikasi agar sesuai dengan persyaratan yang ditentukan dan siap untuk digunakan.

---

## Cakupan

- **Komponen yang akan diuji:**
  - Fungsionalitas login
  - Pemulihan kata sandi
  - Validasi input
  - Penanganan kesalahan
  - Desain dan responsivitas antarmuka pengguna (UI)
  - Langkah-langkah keamanan

---

## Peran dan Tanggung Jawab

| Peran             | Tanggung Jawab                                      |
|-------------------|-----------------------------------------------------|
| Manajer UAT       | Mengawasi proses UAT dan memastikan pengujian selesai|
| Analis Uji        | Melaksanakan kasus uji dan mendokumentasikan hasil  |
| Pengguna Bisnis   | Memberikan umpan balik tentang kegunaan dan fungsi  |
| Pengembang        | Menyelesaikan masalah atau cacat yang ditemukan     |

---

## Lingkungan Pengujian

- **Platform:** Aplikasi web (misalnya, Chrome, Firefox, Safari)
- **Perangkat:** Desktop, tablet, ponsel
- **Data:** Akun pengujian, kredensial valid dan tidak valid

---

## Skenario dan Kasus Uji

### 1. Fungsionalitas Login

| ID Kasus Uji | Deskripsi                            | Langkah Pengujian                                                                           | Hasil yang Diharapkan                  |
|--------------|--------------------------------------|-------------------------------------------------------------------------------------------|----------------------------------------|
| TC-001       | Login dengan kredensial valid        | Masukkan nama pengguna dan kata sandi yang valid; klik "Sign In"                            | Pengguna berhasil masuk                |
| TC-002       | Login dengan kredensial tidak valid  | Masukkan nama pengguna atau kata sandi yang salah; klik "Sign In"                           | Pesan kesalahan ditampilkan            |
| TC-003       | Login dengan kolom kosong            | Biarkan kolom nama pengguna dan/atau kata sandi kosong; klik "Sign In"                      | Pesan kesalahan ditampilkan            |
| TC-004       | Sesi kedaluwarsa                     | Masuk, lalu diamkan selama 60 menit                                                       | Pengguna secara otomatis keluar        |

### 2. Pengujian Antarmuka Pengguna (UI)

| ID Kasus Uji | Deskripsi                            | Langkah Pengujian                                                                           | Hasil yang Diharapkan                  |
|--------------|--------------------------------------|-------------------------------------------------------------------------------------------|----------------------------------------|
| TC-005       | Uji responsivitas halaman            | Akses halaman pada berbagai peramban (desktop, tablet)                           | UI menyesuaikan dengan baik            |
| TC-006       | Uji keterbacaan pesan kesalahan      | Aktifkan kesalahan; perhatikan letak dan keterbacaan pesan kesalahan                       | Pesan kesalahan jelas dan terlihat     |

### 3. Pengujian Keamanan

| ID Kasus Uji | Deskripsi                            | Langkah Pengujian                                                                           | Hasil yang Diharapkan                  |
|--------------|--------------------------------------|-------------------------------------------------------------------------------------------|----------------------------------------|
| TC-007       | Uji pencegahan injeksi SQL           | Coba injeksi SQL pada kolom input                                                         | Input disanitasi; tidak ada eksekusi SQL|
| TC-008       | Uji pencegahan serangan brute-force  | Masukkan kredensial salah berulang kali                                                   | Akun terkunci setelah percobaan tertentu|

---

## Kriteria Penerimaan

1. Semua kasus uji harus berhasil.
2. Tidak ada cacat dengan tingkat kritis atau tinggi yang belum diselesaikan.
3. UI ramah pengguna dan responsif pada semua perangkat.
4. Langkah-langkah keamanan efektif mengurangi kerentanan umum.

---

## Persetujuan

| Nama               | Peran           | Tanggal       | Tanda Tangan     |
|--------------------|-----------------|---------------|-------------------|
|                    | Manajer UAT     |               |                   |
|                    | Pengguna Bisnis |               |                   |
|                    | Pengembang      |               |                   |
