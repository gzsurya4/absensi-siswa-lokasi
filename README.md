# absensi-siswa-lokasi
absensi siswa nerin berbasis lokasi

# Petunjuk Penggunaan: Absensi Siswa SMAN 1 Ciwaringin (Firebase + Blogger)

Aplikasi berupa **satu file** (`absensi-sman1-ciwaringin.html`) dengan database **Firebase Firestore**. Tidak ada server sendiri dan tidak memakai Google Sheets lagi.

## 1. Buat project Firebase

1. Buka https://console.firebase.google.com lalu **Add project** (nama bebas, misalnya `absensi-sman1-ciwaringin`). Google Analytics boleh dimatikan.
2. **Build > Firestore Database > Create database**. Pilih lokasi `asia-southeast2 (Jakarta)` dan mode **Production**.
3. **Build > Authentication > Get started**, lalu di tab **Sign-in method** aktifkan dua metode:
   - **Email/Password** (untuk admin)
   - **Anonymous** (untuk sesi siswa dan wali kelas)
4. Di tab **Users**, klik **Add user**, isi email dan password admin (misalnya `admin@sman1ciwaringin.sch.id`). **Salin User UID** akun tersebut.
5. Kembali ke **Firestore Database > Start collection**:
   - Collection ID: `admins`
   - Document ID: **UID admin tadi**
   - Field: `nama` (string) = `Administrator`
   
   Tanpa dokumen ini, login admin akan ditolak (akun Auth saja tidak cukup).
6. **Authentication > Settings > Authorized domains**: tambahkan domain tempat aplikasi berjalan (misalnya `namablog.blogspot.com`, domain kustom Anda, atau `*.vercel.app` jika memakai Vercel).
7. **Firestore Database > Rules**: hapus isi lama, tempel seluruh isi file `firestore.rules`, lalu klik **Publish**.

## 2. Ambil konfigurasi Firebase

**Project settings (ikon roda gigi) > General > Your apps > ikon Web `</>`**. Daftarkan aplikasi web, lalu salin objek `firebaseConfig`.

Buka file `absensi-sman1-ciwaringin.html`, cari bagian paling atas `<script type="module">`, lalu ubah dua hal:

```javascript
const IS_PREVIEW = false;   // WAJIB false saat dipakai sungguhan
const firebaseConfig = { /* tempel config dari Firebase di sini */ };
```

Setelah `IS_PREVIEW = false`, semua data contoh, banner "Mode Preview", dan akun demo hilang total. Aplikasi hanya membaca dan menulis ke Firestore (tidak memakai localStorage).

## 3. Pasang di Blogger

**Pakai file `absensi-sman1-ciwaringin.blogger.html`** (versi satu baris). Blogger menyisipkan `<br />` di setiap baris baru pada isi laman/postingan, sehingga file biasa yang berbaris-baris bisa merusak JavaScript dan layar hanya menampilkan loading tanpa akhir. Versi `.blogger.html` tidak punya baris baru, jadi aman.

Di versi ini, konfigurasi tetap bisa diedit: cari teks `const IS_PREVIEW=true` lalu ubah menjadi `false`, dan cari `const firebaseConfig={` lalu isi nilainya (semuanya masih dalam satu baris).

1. Blogger > **Laman (Pages) > Laman baru** (postingan juga bisa).
2. Klik ikon pena (kiri atas editor) dan ubah ke **Tampilan HTML**.
3. Hapus isi bawaan, tempel **seluruh isi** file `.blogger.html`, lalu **Publikasikan**. Jangan berpindah ke tampilan **Compose** setelah menempel, karena itu bisa mengubah kodenya.
4. Buka alamat laman tersebut dengan **HTTPS**, langsung di peramban (bukan di dalam iframe atau widget).

Jika aplikasi gagal dimuat, kotak merah akan muncul di layar loading dengan pesan galatnya. Setelah 15 detik masih memuat, muncul pesan agar memeriksa koneksi dan `firebaseConfig`.

Catatan:
- Kamera dan GPS hanya berfungsi di HTTPS dan halaman utama (top-level). Karena itu pakai **Laman**, bukan widget HTML/JS di sidebar.
- Aplikasi menutupi tampilan template Blogger secara penuh, jadi template tidak perlu diedit.
- Alternatif tanpa Blogger: unggah file yang sama sebagai `index.html` ke GitHub Pages atau Vercel.

## 4. Pengaturan awal oleh admin (urutan penting)

Masuk lewat tab **Admin** dengan email dan password dari langkah 1, lalu buka menu **Pengaturan**:

0. **Mode Absen** (tab pertama): pilih **Absen di Sekolah (KBM)** atau **Absen di Rumah (PJJ)**. Saat PJJ aktif, validasi lokasi dimatikan dan siswa cukup selfie wajah dari rumah. Dashboard siswa menampilkan mode yang sedang aktif. Perubahan berlaku seketika.
1. **Jadwal**: atur jam buka/tutup absen masuk dan pulang, hari sekolah, tanggal libur, dan password awal akun baru. Klik **Simpan**.
2. **Lokasi Sekolah**: berdirilah di sekolah, klik **Gunakan lokasi saya sekarang**, atur radius (disarankan 150 m) dan akurasi GPS maksimal (disarankan 100 m), lalu **Simpan**.
3. **Tampilan**: ubah judul dan subjudul halaman login, nama dan subjudul di sidebar, serta unggah logo. Logo SMAN 1 Ciwaringin sudah terpasang sebagai bawaan.

> **Penting:** siswa **tidak bisa absen** sebelum admin menyimpan Jadwal & Lokasi minimal satu kali. Aturan keamanan membaca dokumen ini sebagai dasar validasi.

Lalu isi data master:
- **Kelas**: buat kelas dan tetapkan wali kelas.
- **Pengguna**: tambah wali kelas dan siswa, atau **Impor** banyak siswa sekaligus dengan format satu baris per siswa: `NIS, Nama, Kelas, NoHP`.
- Username memakai huruf kecil, angka, dan tanda strip (3-30 karakter). Password awal sesuai pengaturan, dan siswa dapat menggantinya di menu **Profil**.

## 5. Fitur yang diminta

| Permintaan | Cara kerjanya |
|---|---|
| Validasi lokasi sekolah | GPS akurasi tinggi, jarak dihitung live di layar absen. Tombol ambil foto terkunci bila di luar radius atau akurasi buruk. Security Rules menolak ulang koordinat di luar radius. |
| Waktu terkunci | Waktu absen = `serverTimestamp` Firestore dan wajib sama dengan `request.time`. Layar dan watermark foto memakai jam server. Mengubah jam HP tidak berpengaruh. |
| Verifikasi izin oleh admin | Menu **Izin** di admin menampilkan pengajuan semua kelas (filter kelas dan status). Wali kelas tetap hanya untuk kelasnya. |
| Pengaturan lokasi, judul login, logo, nama sidebar | Menu **Pengaturan** (admin), 3 tab: Jadwal, Lokasi Sekolah, Tampilan. |
| Mode absen Sekolah / PJJ | Pengaturan > **Mode Absen**. Mode PJJ mematikan cek lokasi (aplikasi dan Security Rules). Banner mode tampil di beranda siswa, wali kelas, dan admin. |
| Aturan Alpha | Siswa yang tidak absen masuk sampai batas tutup absen masuk otomatis berstatus **Alpha** (dihitung otomatis, tanpa penjadwalan) dan **tidak bisa absen pulang**. Koreksi tetap bisa dilakukan wali kelas atau admin. |
| Laporan harian tidak hadir | Dashboard admin > **Laporan & PDF**, atau menu Laporan > tab **Tidak Hadir Harian** (Alpha, Izin, Sakit). Tombol **Cetak PDF** dan CSV. Wali kelas: tombol Cetak PDF di halaman Kehadiran. |
| PDF kehadiran per kelas | Menu Laporan > tab **Rekap Per Kelas**: pilih periode dan kelas, klik **Cetak PDF** (satu halaman per kelas + tanda tangan wali kelas). Wali kelas: Rekap Bulanan > Cetak PDF. Pada dialog cetak pilih **Simpan sebagai PDF**. |
| Kalender siswa | Beranda siswa: hijau = hadir/terlambat, kuning = sakit/izin, merah = alpha, netral = lainnya. Ketuk tanggal untuk detail; bisa mundur 6 bulan. |
| Kata motivasi | Muncul di kartu atas beranda siswa dan berganti setiap kali siswa membuka beranda. |
| Foto profil | Menu **Profil** (semua peran). Foto dikompres otomatis (JPEG kecil) dan tampil di header dan beranda. |

## 6. Uji coba sebelum dipakai massal

1. Login admin, simpan Jadwal dan Lokasi.
2. Buat 1 kelas dan 1 siswa uji. Login sebagai siswa di HP.
3. Absen dari dalam sekolah, harus berhasil. Absen dari luar radius, harus ditolak.
4. Ubah jam HP ke waktu lain, lalu absen. Jam yang tercatat tetap jam server.
5. Ajukan izin sebagai siswa, lalu setujui dari akun admin.
6. Di Firebase Console > Firestore > **Rules > Rules Playground**, jalankan simulasi untuk memastikan aturan sesuai.

## 6b. Setelah memperbarui aplikasi (revisi 12 Nov)

1. **Publish ulang `firestore.rules`** (Firestore > Rules > Publish). Aturan baru mengenali mode PJJ dan menolak absen pulang tanpa absen masuk.
2. Tempel ulang file `.blogger.html` yang baru ke laman Blogger (atau unggah ulang `index.html` ke Vercel), lalu pertahankan `IS_PREVIEW=false` dan `firebaseConfig`.
3. Buka Pengaturan > Mode Absen dan simpan sekali agar field `modeAbsen` tersimpan.
4. Catatan: PDF dibuat lewat dialog cetak peramban (tanpa library tambahan). Dokumen absensi lama yang hanya berisi absen pulang (hasil uji coba) tetap terbaca.

## 7. Batasan yang perlu diketahui

- **Security Rules belum diuji di emulator** pada saat pembuatan (emulator tidak dapat diunduh di lingkungan pembuatan). Rules ditulis dan ditinjau manual, jadi lakukan uji pada langkah 6 sebelum dipakai resmi. Bila ada penolakan yang tidak wajar, tanyakan kembali dengan menyertakan pesan galat dari Rules Playground.
- **GPS palsu**: aplikasi web tidak bisa memblokir 100% aplikasi mock location. Sebagai mitigasi, absen yang mencurigakan (akurasi tidak wajar, koordinat bulat) ditandai di laporan, dan server tetap menolak koordinat di luar radius.
- **Password**: tanpa backend, pencocokan password terjadi lewat bukti hash (PBKDF2) yang diperiksa Security Rules. Salt dapat dibaca per akun, tetapi hash tidak pernah bisa dibaca. Tidak ada pembatasan percobaan login di sisi server, jadi gunakan password awal yang kuat dan minta siswa menggantinya.
- **Data lama dari Google Sheets** (riwayat absensi dan foto di Drive) **tidak ikut dimigrasikan**. Daftar siswa dapat diimpor ulang lewat menu Impor (CSV: NIS, Nama, Kelas, NoHP). Simpan Spreadsheet lama sebagai arsip.
- **Kuota gratis (Spark)**: 20.000 penulisan dan 50.000 pembacaan per hari. Satu siswa memakai kira-kira 8-10 penulisan per hari, jadi cukup untuk sekitar 2.000 siswa. Foto disimpan base64 di Firestore (tanpa Firebase Storage).
- **Deteksi wajah** memakai library face-api dari CDN (atau `FaceDetector` bawaan peramban bila tersedia). Siswa perlu koneksi internet saat absen.

## 8. Menjalankan lokal / deploy ke GitHub dan Vercel

Karena satu file, cukup simpan sebagai `index.html`:
- **GitHub Pages**: commit `index.html`, lalu Settings > Pages > Deploy from branch.
- **Vercel**: impor repo, tanpa build command, output directory root.

Jangan lupa menambahkan domain hasil deploy ke **Authorized domains** (langkah 1.6).
