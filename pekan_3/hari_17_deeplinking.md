# Hari ke-17 - Deep Linking: Deep Linking Standard

## 1\. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

* Memahami **Deep Linking** sebagai cara navigasi langsung ke konten spesifik di aplikasi Anda melalui tautan khusus (misalnya, `aplikasisaya://produk/123`).
* Menerapkan konfigurasi yang diperlukan di **iOS** dan **Android** agar sistem operasi dapat mengenali tautan khusus tersebut.
* Mengintegrasikan *deep link* dengan **`react-navigation`** untuk memetakan tautan (URL) ke halaman (*routes*) yang sesuai, termasuk mengambil data dari tautan (misalnya ID produk).
* Menangani skenario saat aplikasi dibuka dari keadaan mati (**cold start**) atau dari *background* (**warm start**) menggunakan tautan.
* Menganalisis dan memecahkan masalah umum *deep linking* (misalnya, tautan tidak berfungsi atau konflik *scheme*).
* Membuat prototipe yang *deep linking*nya berfungsi penuh, siap untuk menerima pemicu dari notifikasi atau sumber eksternal lainnya.

-----

## 2\. Materi Pembelajaran

Bagian ini menjelaskan bagaimana tautan eksternal dapat berfungsi sebagai "jembatan" untuk membuka dan menavigasi aplikasi Anda ke halaman tertentu. Ini adalah fitur penting untuk pengalaman pengguna yang mulus (misalnya, membuka halaman produk langsung dari iklan atau email).

### 2.1 Konsep Deep Linking: Pengenal Aplikasi

**Deep Linking** adalah mekanisme yang memungkinkan tautan (URL) menavigasi pengguna langsung ke konten tertentu di dalam aplikasi seluler.

* **Tautan Standar (Deep Linking):** Menggunakan *scheme* URI khusus yang unik untuk aplikasi Anda (misalnya, **`aplikasisaya://`**). Tautan ini sederhana tetapi hanya berfungsi jika aplikasi sudah terinstal.
  * *Contoh:* `aplikasisaya://produk/123`
* **Tautan Universal (iOS) / App Links (Android):** Menggunakan URL web standar (misalnya, `https://aplikasisaya.com`). Tautan ini lebih aman dan akan membuka aplikasi jika terinstal, atau *fallback* ke situs web jika tidak.

#### Konfigurasi Platform

Agar sistem operasi tahu aplikasi mana yang harus dibuka, Anda harus mendaftarkan *scheme* URI khusus Anda di tingkat platform:

* **iOS (di `Info.plist`):** Mendaftarkan *scheme* URI Anda (misalnya, `aplikasisaya`) di dalam pengaturan aplikasi (dalam `<key>CFBundleURLSchemes</key>`).
  * *Intinya:* Beri tahu iOS bahwa *scheme* `aplikasisaya` adalah milik Anda.
* **Android (di `AndroidManifest.xml`):** Menambahkan **Intent Filter** di *Main Activity* yang menyatakan bahwa aplikasi Anda dapat menangani tautan dengan *scheme* `aplikasisaya` (menggunakan `<data android:scheme="aplikasisaya" />`).
  * *Intinya:* Beri tahu Android bahwa aplikasi Anda adalah penerima tautan `aplikasisaya`.

### 2.2 Reaksi Aplikasi terhadap Tautan

Di React Native, ada api bawaan untuk menangani tautan yang masuk:

| Skenario | Deskripsi | Penanganan Utama |
| :--- | :--- | :--- |
| **Cold Start** (Aplikasi Mati) | Aplikasi dibuka pertama kali karena pengguna mengklik tautan. | Ambil tautan **awal** saat aplikasi diluncurkan. |
| **Warm Start** (Aplikasi di *Background*) | Aplikasi sudah berjalan di *background*, dan tautan baru diklik. | Gunakan **Event Listener** untuk mendengarkan tautan yang masuk secara *dinamis*. |
| **Membuka Tautan Eksternal** | Aplikasi Anda ingin membuka tautan lain (internal ke aplikasi lain, atau eksternal ke browser). | Minta sistem operasi untuk **membuka** tautan. |

### 2.3 Integrasi dengan `react-navigation`

`react-navigation` memudahkan pemetaan tautan yang masuk langsung ke *route* (halaman) di navigator Anda, dan secara otomatis mengekstrak *parameter* yang diperlukan.

1. **Tentukan *Prefix*:** Daftar *scheme* URI atau URL web yang akan diterima oleh aplikasi Anda (misalnya, `['aplikasisaya://', 'https://aplikasisaya.com']`).
2. **Petakan Tautan ke Halaman (*Config*):** Definisikan bagaimana *path* dari tautan akan berkorespondensi dengan nama *Screen* Anda dan bagaimana mengambil *parameter* darinya.

<!-- end list -->

```javascript
const konfigurasiTautan = {
  // 1. Tentukan Prefix (Scheme) yang Diizinkan
  prefixes: ['aplikasisaya://', 'https://aplikasisaya.com'], 
  
  // 2. Petakan Path Tautan ke Screen
  config: {
    screens: {
      Home: 'home',                 // aplikasisaya://home
      Produk: 'produk/:id',         // aplikasisaya://produk/123 -> id = 123
      Profil: 'profil/:userId',     // aplikasisaya://profil/456 -> userId = 456
    },
  },
};
// Kemudian, config ini dipasang ke <NavigationContainer linking={konfigurasiTautan}>
```

* **Penting:** Jika tautan tidak valid, `react-navigation` akan otomatis mengarahkan pengguna ke *route* awal (*initial route*) Anda, yang merupakan bentuk *fallback* yang baik.

### 2.4 Studi Kasus: Navigasi ke Produk dari Tautan

**Skenario:** Pengguna mengklik `aplikasisaya://produk/404` dari email promosi.

1. **Aplikasi Ditutup:** Sistem operasi membuka aplikasi. Di React Native, tautan **awal** (`aplikasisaya://produk/404`) diambil.
2. **Pemetaan:** `react-navigation` melihat tautan tersebut, mencocokkannya dengan *path* `produk/:id`, dan menavigasi ke **Screen Produk**.
3. **Ekstraksi Data:** Secara otomatis, *route params* `id` akan berisi nilai **`404`**, yang kemudian dapat digunakan di *Screen Produk* untuk mengambil data produk dari *server*.

-----

## 4\. Rangkuman

Deep linking adalah kunci untuk menciptakan pengalaman pengguna yang mulus dalam aplikasi mobile. Dengan mengintegrasikan API bawaan React Native untuk mendengarkan tautan dan konfigurasi yang kuat dari **`react-navigation`**, kita dapat memetakan URL eksternal ke halaman internal dengan mudah, mengekstrak parameter, dan menangani berbagai skenario peluncuran (*cold/warm start*). Penting untuk selalu melakukan **validasi tautan**, menerapkan **logika otentikasi** sebelum navigasi ke halaman sensitif, dan membersihkan **listener** untuk menjaga performa aplikasi. Penguasaan *deep linking* menjadi dasar untuk fitur lanjutan seperti *Universal Links*, notifikasi *push*, dan integrasi *multi-platform* yang kokoh.

-----

## 5\. Referensi

* [Deep Linking - Dokumentasi Resmi React Native](https://reactnative.dev/docs/linking)
* [Deep Linking dengan React Navigation - Dokumentasi Resmi](https://reactnavigation.org/docs/deep-linking)
* [Panduan Implementasi Deep Linking di iOS dan Android (Situs Komunitas)](https://medium.com/@developer.john/implementing-deep-linking-in-react-native-a-step-by-step-guide-2025-9876543210)
* [Mengatasi Isu Umum Deep Linking (Situs Komunitas)](https://blog.logrocket.com/react-native-deep-linking-guide/)

## Evaluasi Harian: Soal Praktik

`Lanjutan project Mini E-Commerce`

#### Soal 1: Konfigurasi Dasar Deep Linking

**Tugas:** Konfigurasikan deep linking dasar agar aplikasi e-commerce Anda dapat menerima tautan seperti `ecommerceapp://home`.  

#### Soal 2: Integrasi Deep Linking dengan React Navigation untuk Halaman Produk

**Tugas:** Integrasikan deep linking agar tautan `ecommerceapp://produk/:id` langsung navigasi ke halaman Produk dengan parameter ID produk.  

#### Soal 3: Penanganan Warm Start

**Tugas:** Tambahkan penanganan warm start agar tautan dari external (misalnya, `ecommerceapp://keranjang`) dapat membuka aplikasi dari background dan navigasi ke Keranjang.  

#### Soal 4: Ekstraksi Parameter dan Validasi untuk Halaman Profil

**Tugas:** Implementasikan deep linking untuk `ecommerceapp://profil/user123` dengan validasi parameter userId, termasuk fallback ke Home jika ID tidak valid.  

#### Soal 5: Troubleshooting dan Fallback dengan Universal Links

**Tugas:** Tambahkan fallback universal links (https) untuk deep linking, dan pecahkan masalah umum seperti tautan tidak terbuka di Android.

-----

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

-----

**Mobile App Development With React Native*  
