# Sistem Manajemen Pondok — Setup (Tahap 1: Database + Backend + Login)

Tahap ini mencakup: struktur database, backend API (`kode.gs`), dan halaman login
multi-role (`login.html`). Modul dashboard, halaman santri, absen, dan keuangan
menyusul di tahap berikutnya.

## 1. Buat Google Sheet (1 menit)

1. Buka [sheets.google.com](https://sheets.google.com) → buat spreadsheet baru, kosong.
2. Beri nama terserah, misal **"Database Pondok"**.

## 2. Pasang backend (2 menit)

1. Di spreadsheet tadi: **Extensions → Apps Script**.
2. Hapus semua isi editor default (`Code.gs`), tempel seluruh isi file **`kode.gs`** yang sudah dibuatkan.
3. Klik **Save** (ikon disket / Ctrl+S).
4. Di toolbar atas, ada dropdown pilihan fungsi (biasanya bertuliskan `doGet`) → ganti ke **`setupSheets`**.
5. Klik **Run** ▶️. Kalau muncul minta izin (authorize), klik **Review permissions** → pilih akun Google kamu → **Advanced** → **Go to (nama project) (unsafe)** → **Allow**. Ini normal karena script belum diverifikasi Google (punya kamu sendiri, aman).
6. Setelah selesai jalan, cek spreadsheet-nya — akan otomatis muncul 4 sheet: `DataSantri`, `RiwayatSetoran`, `Users`, `TransaksiKeuangan`, lengkap dengan header kolom. Juga otomatis dibuatkan 1 akun percobaan:
   - Username: `admin`
   - PIN: `123456`
   - Role: `SuperAdmin`

   **Ganti PIN ini nanti setelah setup selesai**, langsung edit di sheet `Users`.

## 3. Deploy sebagai Web App (1 menit)

1. Masih di editor Apps Script: klik **Deploy → New deployment**.
2. Klik ikon gerigi ⚙️ di samping "Select type" → pilih **Web app**.
3. Isi:
   - **Execute as**: `Me`
   - **Who has access**: `Anyone`
4. Klik **Deploy**. Izinkan akses lagi kalau diminta.
5. Copy **Web app URL** yang muncul (formatnya `https://script.google.com/macros/s/xxxxx/exec`).

## 4. Sambungkan ke `login.html`

Buka file `login.html`, cari baris ini di bagian atas `<script>`:

```js
const API_URL = 'GANTI_DENGAN_URL_DEPLOYMENT_APPS_SCRIPT';
```

Ganti dengan URL hasil deploy tadi. Simpan file.

## 5. (Opsional) Aktifkan kirim WA / OTP via Fonnte

Fitur OTP WhatsApp untuk role **SuperAdmin** dan **Admin** butuh akun Fonnte:

1. Daftar di [fonnte.com](https://fonnte.com), hubungkan nomor WA device kamu.
2. Copy **token device** dari dashboard Fonnte.
3. Buka `kode.gs`, cari baris:
   ```js
   const FONNTE_TOKEN = 'GANTI_DENGAN_TOKEN_FONNTE_KAMU';
   ```
   Ganti dengan token kamu. Simpan, lalu **Deploy → Manage deployments → Edit (ikon pensil) → Deploy** lagi supaya perubahan aktif.
4. Isi kolom `no_wa` di sheet `Users` untuk akun SuperAdmin/Admin (format `62812xxxxxxx`, tanpa tanda `+`).

Kalau `FONNTE_TOKEN` belum diisi, login untuk role SuperAdmin/Admin akan gagal
saat mengirim OTP — sementara pakai role `Ustadz`, `Wali`, atau `Santri` dulu untuk testing (tidak butuh OTP).

## 6. Coba login

Buka `login.html` di browser (bisa langsung dari file lokal untuk testing, atau
host di GitHub Pages seperti `kertas_pantau.html`). Pilih role **Super Admin**,
masuk dengan `admin` / `123456`.

---

### Struktur kolom database (referensi)

| Sheet | Kolom |
|---|---|
| `DataSantri` | id, nama, kelas, halaqoh, no_wali, target, tercapai, target_kitab, tercapai_kitab, target_tahsin, tercapai_tahsin, poin, status, foto_url |
| `RiwayatSetoran` | id_santri, tanggal, jenis, jumlah, nilai, catatan, bukti_url, ustadz |
| `Users` | username, pin, role, halaqoh, no_wa |
| `TransaksiKeuangan` | tanggal, jenis, keterangan, jumlah, bukti |

`target`/`tercapai` = progres Qur'an. `target_kitab`/`tercapai_kitab` dan
`target_tahsin`/`tercapai_tahsin` dihitung otomatis dari `jenis` setoran yang
mengandung kata "Kitab" / "Tahsin" (lihat form input di `santri.html`).

> **Kalau sheet `DataSantri` sudah dibuat sebelum tahap 3 ini**, tambahkan manual
> 4 kolom baru (`target_kitab`, `tercapai_kitab`, `target_tahsin`, `tercapai_tahsin`)
> di antara kolom `tercapai` dan `poin` — atau paling gampang, hapus sheet lama lalu
> jalankan ulang `setupSheets()` (perhatian: ini akan menghapus data yang sudah ada).

`jenis` di `RiwayatSetoran` bisa berisi: `Setor Quran`, `Setor Kitab`, `Setor Tahsin`,
`Murajaah`, `Telat`, `Bolos`, `Absen`. Semua yang diawali kata "Setor" dihitung +10 poin.

### Rumus poin (otomatis, dihitung ulang tiap ada setoran baru)
- Setor (semua jenis): **+10**
- Murajaah: **+5**
- Telat: **−5**
- Bolos: **−20**

### Tahap selanjutnya
Berikutnya: `keuangan.html` (keuangan + QRIS), notifikasi WA otomatis, leaderboard,
AI asisten, dan modul keamanan/admin.

### Catatan Tahap 4 — Absen (`absen.html`)
- Absen manual oleh Ustadz/Admin/SuperAdmin (Wali & Santri tidak bisa akses halaman ini).
- Idempotent: menyimpan absen untuk santri+tanggal yang sama akan **memperbarui**
  baris yang sudah ada di `RiwayatSetoran`, bukan bikin duplikat.
- Rekap bisa difilter per rentang tanggal & halaqoh, plus ekspor PDF.
