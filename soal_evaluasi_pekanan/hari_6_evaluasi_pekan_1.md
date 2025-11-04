# Hari ke-6: Evaluasi Pekan 1

## 1. Durasi

**Waktu pengerjaan:** 1 hari (maksimal 8 jam efektif)  
**Batas pengumpulan:** sesuai instruksi mentor  

---

## 2. Tujuan Evaluasi

- Mengevaluasi pemahaman konsep dasar dari pekan 1.
- Menguji kemampuan menyusun struktur UI dengan Core Components (`View`, `Text`, `Image`, `TextInput`).
- Membuktikan pemahaman daftar data dasar (menampilkan daftar produk dengan gambar dari internet).
- Menilai pengelolaan state sederhana untuk menambah produk baru.
- Menilai kemampuan membuat modal berisi form (input nama, harga, URL gambar, deskripsi) dan menutup/buka modal dengan benar.
- Menilai dasar validasi input (wajib/format) dan umpan balik UX sederhana.

## 3. Deskripsi Project

- Judul proyek: Mini E-Commerce.
- Deskripsi singkat: Bangun satu halaman yang menampilkan daftar produk dengan gambar, nama, harga, dan deskripsi singkat. Sediakan tombol untuk menambah produk melalui sebuah modal berisi form. Gambar produk diambil dari sumber online (URL gambar).
- Fitur wajib:
  - Daftar produk: tampilkan minimal 5 produk awal (boleh data statis awal) dengan gambar (URL), nama, harga, deskripsi.
  - Tambah produk: sebuah tombol yang membuka modal berisi form dengan bidang minimal: Nama Produk (wajib), Harga (wajib, angka), URL Gambar (wajib, valid), Deskripsi (opsional). Setelah submit, produk baru muncul di daftar.
  - Modal: dapat dibuka/ditutup dengan jelas; submit hanya jika valid; ada umpan balik dasar saat data tidak valid.
  - Gambar dari online: gunakan URL gambar (misal CDN atau placeholder) dan pastikan tampil konsisten.
  - Penyimpanan data: cukup di memori aplikasi (state) tidak perlu backend.
- Batasan/ruang lingkup:
  - Tidak perlu navigasi multi‑halaman.
  - Tidak perlu integrasi backend atau upload file.
  - Styling sederhana, utamakan keterbacaan dan fungsi.

## 4. Kriteria Penilaian

- Kelengkapan fitur (40%):
  - Daftar produk tampil dengan gambar, nama, harga, deskripsi.
  - Tombol tambah produk dan modal berfungsi.
- Fungsi tambah produk (25%):
  - Form di modal memiliki input yang diminta.
  - Produk baru muncul di daftar setelah submit.
- Validasi & UX (20%):
  - Validasi wajib/format dasar (angka untuk harga, URL gambar).
  - Umpan balik jelas untuk input tidak valid; modal tertutup dengan benar setelah submit.
- Dokumentasi & kesiapan run (15%):
  - Petunjuk menjalankan proyek jelas.
  - Informasi versi RN/lingkungan disertakan.

## 6. Tips Memulai

- Memulai dengan membaca Basmallah dan berdoa
- Rencanakan data: tentukan struktur objek produk (id, nama, harga, urlGambar, deskripsi) dan siapkan beberapa data awal.
- Kerangka UI terlebih dahulu: susun `<View>` untuk header, daftar, dan tombol tambah; fokuskan pada alur utama sebelum memperindah tampilan.
- Modal & form: pastikan alur buka/tutup jelas, dan validasi dasar berjalan (kosong/format angka/URL).
- Gambar online: uji beberapa URL gambar agar tampil konsisten; siapkan placeholder jika URL gagal dimuat.
- Uji kasus utama: tambah produk valid, input tidak valid, tutup modal tanpa submit, dan cek produk muncul sesuai harapan.
- Jaga sederhana: styling minimal, tekankan fungsi dan keterbacaan.
