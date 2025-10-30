# Hari ke-2: Pengenalan React Native & Setup Environment

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

* Menjelaskan arsitektur dasar React Native (JavaScript Thread, Native Modules, Bridge).
* Memahami peran React Native CLI dan perbedaannya dengan Expo.
* Mengidentifikasi dan menyiapkan prasyarat lingkungan pengembangan (Node.js, JDK, Android Studio/Xcode) untuk React Native CLI versi 0.80.

## 2. Materi Pembelajaran

### A. Arsitektur React Native (New Architecture)

Sejak RN 0.76, arsitektur baru diaktifkan secara default dan dilanjutkan di RN 0.80. Arsitektur ini mengganti *Bridge* lama dengan antarmuka yang lebih cepat dan selaras dengan fitur React 18 untuk pengalaman UI yang lebih responsif dan konsisten [4].

#### 1) Gambaran Umum

* Pemisahan dunia JavaScript dan Native tetap ada, tetapi cara berkomunikasi berubah dari saluran pesan ter-serialisasi (*Bridge*) menjadi antarmuka memori langsung.
* Dukungan terhadap fitur React 18 seperti concurrent rendering, Suspense untuk data fetching, Transitions, dan automatic batching.

#### 2) Komponen Kunci Arsitektur Baru

* JavaScript Interface (JSI):
  * Menghapus *Bridge* asinkron yang lama dan memungkinkan JavaScript memegang referensi ke objek C++ secara langsung.
  * Meminimalkan biaya serialisasi saat memanggil fungsi native, cocok untuk beban data tinggi (mis. pemrosesan kamera real-time).
* Fabric Renderer:
  * Renderer UI baru yang selaras dengan model layout React serta mendukung scheduling yang lebih baik.
  * Memberikan akses layout yang sinkron sehingga menghindari *visual jumps* saat mengukur dan menata ulang tampilan.
* TurboModules:
  * Sistem modul native generasi baru dengan inisialisasi dan pemanggilan yang lebih efisien melalui JSI.
  * Mengurangi overhead dan meningkatkan performa pemanggilan API native.
* Codegen:
  * Menghasilkan binding tipe-aman antara JavaScript dan Native (Android/iOS) dari spesifikasi antarmuka.
  * Mengurangi kesalahan integrasi dan menjaga konsistensi antar platform.

#### 3) Fitur Konkuren & Layout Sinkron

* Concurrent Renderer & Automatic Batching:
  * Mengurangi *re-render* yang tidak perlu dan meningkatkan responsivitas.
  * Transitions memungkinkan prioritas pembaruan UI, sehingga interaksi penting tetap lancar.
* Synchronous Layout & Effects:
  * Mengakses informasi layout secara sinkron untuk merender komponen adaptif tanpa keadaan visual perantara.
  * Meminimalkan flicker ketika menyesuaikan ukuran/posisi berdasarkan pengukuran.

#### 4) Implikasi Praktis untuk Proyek

* App baru RN 0.80 sudah memakai New Architecture secara default. Beberapa perpustakaan mungkin perlu versi terbaru agar kompatibel dengan Fabric/TurboModules.
* Kode Anda bisa memerlukan penyesuaian untuk memanfaatkan fitur konkuren dan layout sinkron secara penuh.
* Jika menghadapi hambatan, Anda dapat mengacu dokumentasi resmi untuk opsi *opt-out* sementara; namun disarankan tetap menggunakan arsitektur baru [4].

### B. React Native CLI vs. Expo

Terdapat dua cara utama untuk memulai proyek React Native:

| Fitur | React Native CLI | Expo |
| :--- | :--- | :--- |
| **Kontrol** | Penuh atas *native code* (Android/iOS) | Terbatas, disembunyikan oleh *framework* |
| **Akses API** | Akses langsung ke semua API *native* | Terbatas pada API yang disediakan oleh Expo SDK |
| **Ukuran Aplikasi** | Lebih kecil, hanya menyertakan *native module* yang dibutuhkan | Lebih besar, menyertakan seluruh Expo SDK |
| **Kompleksitas Setup** | Lebih kompleks (membutuhkan JDK, Android Studio, Xcode) | Sangat mudah, hanya butuh Node.js |
| **Versi 0.80** | New Architecture default di proyek CLI [4] | Mendukung New Architecture di SDK modern |

Karena modul ini menggunakan **React Native CLI**, kita akan fokus pada *Bare Workflow* yang memberikan kontrol penuh.

### C. Prasyarat Lingkungan Pengembangan (React Native CLI 0.80)

Untuk React Native 0.80, prasyarat utama untuk pengembangan Android adalah:

1. **Node.js:** Versi 18.18 atau yang lebih baru.
2. **Java Development Kit (JDK):** Versi 17 direkomendasikan.
3. **Android Studio:** Diperlukan untuk Android SDK dan AVD (Android Virtual Device).
4. **React Native CLI:** Diinstal secara otomatis saat membuat proyek baru.
5. **Hermes Engine:** Mesin JavaScript default yang dioptimalkan untuk RN; pastikan aktif untuk performa lebih baik.

**Langkah-langkah Kunci Setup Android:**

1. **Instalasi Android Studio:** Unduh dan instal Android Studio.
2. **Instalasi Android SDK:**
    * React Native 0.80 membutuhkan **Android SDK Platform 35 (VanillaIceCream)**.
    * Pastikan **Android SDK Build-Tools 35.0.0** juga terinstal [3].
3. **Konfigurasi Variabel Lingkungan:**
    * Setel variabel lingkungan `ANDROID_HOME` ke lokasi SDK Anda (misalnya, `~/Android/Sdk`).
    * Tambahkan `platform-tools` dan `emulator` ke `PATH` Anda.

```bash
# Contoh konfigurasi di ~/.zshrc atau ~/.bashrc
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

4. **Watchman (Opsional, Direkomendasikan):** Alat oleh Facebook untuk memantau perubahan *filesystem* guna meningkatkan performa *bundling*.
5. **Emulator/Perangkat Fisik:** Uji di berbagai perangkat untuk memvalidasi performa dan kompatibilitas (DPI, versi Android, produsen).
6. **Dependensi Library:** Gunakan versi pustaka yang kompatibel dengan Fabric/TurboModules (lihat dokumentasi masing-masing library).

## 3. Contoh Implementasi

### A. Membuat Proyek Baru dengan React Native CLI 0.80

React Native CLI modern tidak lagi diinstal secara global, tetapi digunakan melalui `npx`.

```bash
# Ganti MyAwesomeApp dengan nama proyek Anda
# --version 0.80.0 untuk memastikan versi yang digunakan
npx @react-native-community/cli@latest init AwesomeProject --version 0.80
```

### B. Menjalankan Aplikasi di Emulator/Simulator

Setelah proyek dibuat, masuk ke direktori proyek dan jalankan perintah:

```bash
cd MyAwesomeApp

# Untuk Android
npx react-native run-android

# Untuk iOS (hanya di macOS)
npx react-native run-ios
```

Perintah ini akan:

1. Memulai Metro Bundler (server yang mengkompilasi kode JavaScript).
2. Membangun aplikasi *native* (menggunakan Gradle untuk Android atau Xcode untuk iOS).
3. Menginstal aplikasi ke emulator/simulator yang sedang berjalan.

## 4. Ringkasan Materi

* New Architecture menggantikan *Bridge* lama dengan JSI, serta menghadirkan Fabric renderer, TurboModules, dan Codegen untuk interop yang lebih cepat dan aman.

* Dukungan fitur React 18 (concurrent rendering, Suspense, Transitions, automatic batching) meningkatkan responsivitas dan konsistensi perilaku antara web dan mobile.
* RN 0.80 mengaktifkan New Architecture secara default; memastikan kompatibilitas pustaka sangat penting sebelum mengadopsi fitur baru secara agresif.
* CLI vs Expo: CLI memberi kontrol penuh terhadap kode native dan siklus rilis, sementara Expo memudahkan setup namun dengan batasan tertentu. Keduanya kini mendukung New Architecture di versi modern.
* Setup lingkungan mencakup Node 18+, JDK 17, Android SDK 35, Hermes aktif, serta pengujian di emulator/perangkat fisik.

## 5. Referensi

[1] React Native Docs 0.80: Architecture. `https://reactnative.dev/docs/0.80/architecture-overview`
[2] React Native Docs 0.80: Get Started with React Native. `https://reactnative.dev/docs/0.80/environment-setup`
[3] React Native Docs 0.80: Set Up Your Environment (Android). `https://reactnative.dev/docs/0.80/set-up-your-environment`
[4] React Native: About the New Architecture. `https://reactnative.dev/architecture/landing-page`

## 6. Evaluasi Harian

1. **Jelaskan apa perbedaan antara React Native CLI & Expo dalam mengembangkan aplikasi menggunakan React Native**

2. **Jelaskan kelebihan & kekurangan dari masing - masing pendekatan dalam mengembangkan aplikasi di React Native & kapan sebaiknya menggunakan salah 1 diantara keduanya?**

3. **Sebutkan keperluan-keperluan dan persyaratan untuk mengembangkan aplikasi menggunakan React Native dan kenapa semua itu diperlukan?**

4. **Jelaskan langkah-langkah untuk membuat dan menjalankan proyek React Native baru menggunakan CLI. Apa saja tantangan yang mungkin dihadapi dan bagaimana mengatasinya?**

5. **Jelaskan arsitektur dasar React Native sesuai dengan pemahaman anda**
