# Hari ke-1: Pengenalan Konsep Mobile Development

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

* Memahami perbedaan mendasar antara pengembangan aplikasi *native*, *hybrid*, dan *cross-platform*.
* Menjelaskan posisi React Native dalam ekosistem pengembangan aplikasi *mobile*.
* Mengidentifikasi tantangan dan keuntungan utama dalam pengembangan aplikasi *mobile* dibandingkan dengan pengembangan *web*.

## 2. Materi Pembelajaran

### A. Definisi Mobile App Development

Mobile App Development adalah proses end-to-end untuk merancang, membangun, menguji, merilis, dan memelihara aplikasi yang berjalan di perangkat mobile seperti smartphone dan tablet. Aplikasi ini ditargetkan untuk platform spesifik (Android/iOS) dan memanfaatkan kemampuan perangkat keras seperti kamera, GPS, sensor gerak, penyimpanan lokal, notifikasi, hingga proses latar belakang.

* Fokus utama: pengalaman pengguna yang selaras dengan platform target, performa yang stabil pada perangkat yang beragam, serta kepatuhan terhadap kebijakan distribusi toko aplikasi(Play Store/App Store).
* Output teknis: paket rilis `APK/AAB` (Android) atau `IPA` (iOS), sertifikat penandatanganan (signing), konfigurasi izin (permissions), dan metadata rilis (ikon, deskripsi, kebijakan privasi).
* Pendekatan implementasi: native (Kotlin/Swift), hybrid (WebView), atau cross-platform native (React Native/Flutter) yang memanfaatkan komponen UI native.

### B. Perbedaan Web Development vs Mobile App Development

Terdapat berbagai konsep umum (UI, state, networking), pengembangan aplikasi web dan aplikasi mobile memiliki perbedaan mendasar pada target lingkungan eksekusi, distribusi, dan akses fitur perangkat.

| Aspek | Web Development | Mobile App Development |
| :--- | :--- | :--- |
| Target eksekusi | Browser (DOM) | Runtime native (Android/iOS, komponen UI native) |
| Distribusi | URL, server deploy | Store (Play/App Store), sideload/enterprise |
| Update | Server-side, cache invalidation | Client-side, versi aplikasi, review store |
| UI/UX | DOM, responsif, input mouse/keyboard | Gesture, haptic, pola navigasi native |
| Akses hardware | Terbatas, via API browser | Luas: kamera, GPS, sensor, background tasks |
| Lifecycle | Halaman/tab, focus/visibility | App lifecycle, foreground/background, AppState |
| Penyimpanan | Cookies, localStorage, IndexedDB | AsyncStorage/SQLite, Keychain/Keystore |
| Performa | Bergantung browser & DOM | Dekat native, optimasi bridge/jembatan |
| Keamanan | CORS, CSP, XSS, CSRF | Sandbox, signing, permission, data-at-rest |
| Testing | Unit/UI via browser (Jest, Playwright) | Unit, E2E di device/simulator (Jest, Detox) |

Implikasi praktis: mobile menuntut pengelolaan izin, penanganan jaringan yang fluktuatif, optimasi konsumsi baterai/memori, dan proses rilis yang tunduk pada kebijakan toko aplikasi.

### C. Tiga Pendekatan Utama dalam Mobile Development

Pengembangan aplikasi *mobile* secara umum terbagi menjadi tiga kategori utama. Berikut penjelasan detail masing-masing pendekatan, termasuk definisi mendalam, prinsip kerja, dan konteks penggunaan. Setelah itu, disajikan tabel perbandingan keuntungan dan kekurangan untuk memudahkan analisis.

#### 1. Native Development

Native Development melibatkan pembangunan aplikasi secara langsung menggunakan bahasa pemrograman dan alat pengembangan yang dirancang khusus untuk platform target. Untuk Android, ini berarti menggunakan Kotlin atau Java dengan Android Studio, di mana kode di-compile menjadi bytecode yang dijalankan oleh Dalvik/ART runtime. Untuk iOS, Swift atau Objective-C dengan Xcode menghasilkan binary yang dijalankan di dalam ekosistem Apple. Pendekatan ini menekankan integrasi penuh dengan API platform, seperti Core Animation untuk animasi halus di iOS atau Jetpack Compose untuk UI deklaratif di Android. Prosesnya mencakup siklus build terpisah per platform, dengan fokus pada optimalisasi low-level seperti manajemen thread native dan akses langsung ke kernel perangkat. Cocok untuk aplikasi yang memerlukan performa ekstrem, seperti game AAA atau app medis real-time, di mana latensi milidetik krusial.

#### 2. Hybrid Development

Hybrid Development menggabungkan teknologi web standar (HTML, CSS, JavaScript) ke dalam wadah aplikasi native melalui komponen WebView, yang pada dasarnya adalah browser embedded di dalam app. Framework seperti Apache Cordova atau Ionic memungkinkan satu set kode web di-bundle menjadi aplikasi yang dapat diinstal, dengan plugin JavaScript untuk memanggil fungsi native (misalnya, mengakses kamera melalui bridge sederhana). Prinsip kerjanya mirip situs web progresif (PWA), tapi dibungkus agar terlihat seperti app native—kode JS dieksekusi di engine seperti V8 (Android) atau WebKit (iOS), dan interaksi dengan hardware bergantung pada plugin pihak ketiga. Ini ideal untuk aplikasi konten-berat seperti berita atau dashboard admin, di mana kecepatan prototipe lebih penting daripada performa grafis intensif, meskipun sering kali mengorbankan kelancaran gesture native.

#### 3. Cross-Platform Native Development (React Native)

Cross-Platform Native Development memungkinkan pembuatan aplikasi menggunakan satu basis kode tunggal (umumnya JavaScript/TypeScript dengan framework seperti React Native atau Dart dengan Flutter), yang kemudian dirender menjadi komponen UI native asli untuk setiap platform tanpa bergantung pada WebView. Di React Native, misalnya, kode JSX diterjemahkan melalui bridge ke elemen native seperti UIView (iOS) atau View (Android), dengan engine layout seperti Yoga untuk konsistensi Flexbox cross-platform. Prinsipnya adalah "learn once, write anywhere"—kode logika bisnis dibagikan, sementara UI dan akses API disesuaikan secara otomatis atau manual via modul platform-spesifik. Ini melibatkan proses kompilasi di mana JavaScript di-bundle dan dikomunikasikan secara asinkronus ke thread native, menghasilkan aplikasi yang terasa otentik meski dari satu sumber. Pendekatan ini populer untuk startup atau app sosial seperti Instagram, di mana skalabilitas tim developer dan waktu rilis cepat menjadi prioritas, dengan trade-off pada kompleksitas debugging bridge.

| Pendekatan                  | Keuntungan                                                                 | Kekurangan                                                                 |
|-----------------------------|----------------------------------------------------------------------------|----------------------------------------------------------------------------|
| **Native Development**     | - Performa maksimal karena akses langsung ke hardware dan API tanpa overhead.<br>- Pengalaman pengguna paling otentik, selaras dengan panduan desain platform (e.g., Material Design).<br>- Kemampuan debugging mendalam dan dukungan komunitas platform resmi yang kuat. | - Memerlukan dua codebase terpisah, meningkatkan waktu dan biaya pengembangan.<br>- Pemeliharaan sulit untuk fitur cross-platform, karena perubahan harus direplikasi manual.<br>- Kurva belajar tinggi untuk developer baru, memerlukan spesialisasi per platform. |
| **Hybrid Development**     | - Satu codebase untuk multi-platform, mempercepat pengembangan dan prototyping.<br>- Mudah bagi developer web untuk bertransisi, dengan ekosistem JS yang kaya.<br>- Biaya rendah untuk maintenance, karena update sentralisasi. | - Performa lebih rendah karena ketergantungan WebView, menyebabkan lag pada interaksi kompleks.<br>- Pengalaman pengguna kurang native, terasa seperti situs web di app shell.<br>- Akses hardware terbatas dan tidak konsisten, bergantung pada plugin yang mungkin outdated. |
| **Cross-Platform Native**  | - Satu codebase menghasilkan UI native, mendekati performa native dengan biaya rendah.<br>- Skalabilitas tinggi untuk tim, dengan komunitas besar dan library siap pakai.<br>- Fleksibilitas untuk kustomisasi platform-spesifik tanpa rewrite total. | - Overhead bridge dapat menimbulkan bottleneck pada operasi intensif (e.g., animasi berat).<br>- Debugging lebih kompleks karena melibatkan dua layer (JS dan native).<br>- Potensi inkonsistensi UI jika tidak dikelola dengan baik, meski lebih baik dari hybrid. |

### D. Posisi React Native

React Native (RN) adalah *framework* yang memungkinkan Anda membangun aplikasi *mobile* *cross-platform* dengan filosofi dan sintaks yang sama seperti ReactJS.

| Aspek | ReactJS (Web) | React Native (Mobile) |
| :--- | :--- | :--- |
| **Target** | Website(Chrome, Firefox, Safari, dll) | OS Mobile (Android/IOS) |
| **Sintaks Dasar** | Sama (JSX) | Sama (JSX) |
| **Styling** | CSS | JavaScript Objects (mirip Flexbox dan CSS *subset*) |
| **Komponen** | `<div>`, `<span>`, `<p>`, dll. | `<View>`, `<Text>`, `<Image>`, dll. |

**Inti dari React Native:** RN tidak menghasilkan kode *web* yang berjalan di *browser* tersembunyi. Sebaliknya, kode JavaScript Anda berkomunikasi dengan *runtime* *native* melalui sebuah *bridge* untuk membuat dan memanipulasi elemen UI *native* yang sesungguhnya. Inilah yang membuat aplikasi RN terasa seperti aplikasi *native* sejati.

### E. Tahapan dalam Mobile App Development

Berikut tahapan umum yang direkomendasikan dalam siklus hidup aplikasi mobile:

1) Discovery & Requirement

* Mengidentifikasi masalah bisnis utama dan menentukan target platform (Android/iOS) berdasarkan audiens dan pasar.
* Mendefinisikan use-case inti, alur pengguna, serta kebutuhan fungsional seperti dukungan offline atau online, termasuk analisis risiko awal dan prioritas fitur.

2) Perancangan Arsitektur & Teknologi

* Menentukan pendekatan pengembangan (native, hybrid, atau cross-platform) serta struktur keseluruhan modul aplikasi.
* Merancang strategi manajemen state, pola navigasi (seperti stack atau tab-based), serta mekanisme penanganan error dan logging untuk memastikan skalabilitas dan keandalan.

3) Desain UI/UX

* Merancang antarmuka yang adaptif terhadap variasi ukuran layar, resolusi, dan pola interaksi pengguna seperti gesture.
* Mengembangkan sistem desain terpadu, termasuk komponen yang dapat digunakan ulang dan prinsip aksesibilitas untuk mendukung beragam pengguna.

4. Setup Lingkungan & Proyek

* Menyiapkan infrastruktur pengembangan dasar, termasuk lingkungan runtime dan konfigurasi dependensi inti.
* Menginisialisasi struktur proyek, termasuk pengaturan transpiler, bundler, dan integrasi modul dasar seperti navigasi dan penyimpanan.

5) Implementasi Fitur

* Membangun layar dan komponen inti menggunakan elemen UI dasar.
* Mengintegrasikan fungsionalitas seperti komunikasi jaringan, penyimpanan data lokal, dan pengelolaan izin perangkat sesuai dengan kebutuhan fitur.
* Melakukan optimasi awal untuk performa, seperti pengelolaan memori dan pemuatan data secara efisien.

6) Pengujian

* Melakukan pengujian unit untuk memverifikasi fungsi individu dan pengujian snapshot untuk kestabilan UI.
* Melaksanakan pengujian end-to-end di lingkungan simulasi dan perangkat nyata, serta pengujian manual untuk skenario penggunaan beragam.

7) Build, Signing, dan Release

* Menyiapkan paket rilis yang aman melalui proses signing dan enkripsi.
* Mengonfigurasi metadata aplikasi dan memvalidasi kepatuhan terhadap standar keamanan serta regulasi platform.

8) Distribusi & Operasional

* Memublikasikan aplikasi ke saluran distribusi resmi dan memantau proses persetujuan serta umpan balik awal.
* Mengimplementasikan sistem pemantauan seperti pelaporan crash dan analitik pengguna untuk mendukung operasional harian.

9. Pemeliharaan & Iterasi

* Menganalisis dan memperbaiki isu pasca-rilis, termasuk kompatibilitas dengan pembaruan sistem operasi.
* Merencanakan iterasi berkelanjutan berdasarkan data penggunaan dan umpan balik untuk peningkatan fitur serta performa.

## 3. Contoh Implementasi

Karena hari ini adalah pengenalan konsep, tidak ada implementasi kode yang spesifik. Namun, berikut adalah perbandingan kode konseptual untuk menunjukkan perbedaan antara ReactJS dan React Native:

### Contoh Konseptual: Menampilkan Teks "Halo Dunia"

#### 1. ReactJS (Web)

```jsx
// App.js (ReactJS)
import React from 'react';

function App() {
  return (
    <div className="container">
      <p style={{ fontSize: '24px', color: 'blue' }}>
        Halo Dunia!
      </p>
    </div>
  );
}

export default App;
```

*Keterangan:* Menggunakan elemen HTML standar (`div`, `p`) dan *styling* CSS.

#### 2. React Native (Mobile)

```jsx
// App.js (React Native)
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

const App = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.greeting}>
        Halo Dunia!
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
  },
  greeting: {
    fontSize: 24, // Tidak perlu 'px'
    color: 'blue', // Tidak perlu tanda kutip
  },
});

export default App;
```

*Keterangan:* Menggunakan komponen Core React Native (`View`, `Text`) dan *styling* berbasis JavaScript Object (menggunakan `StyleSheet`).

## 4. Ringkasan Materi

* Mobile App Development mencakup proses ujung ke ujung untuk membangun aplikasi yang berjalan di Android/iOS dengan memanfaatkan komponen UI native dan fitur perangkat.
* Perbedaan utama dengan web: target runtime, cara distribusi, akses hardware, lifecycle, penyimpanan, performa, keamanan, strategi update, dan praktik testing.
* Tahapan pengembangan meliputi: discovery, arsitektur, desain, setup, implementasi, pengujian, build/signing/release, distribusi/operasional, serta pemeliharaan dan iterasi berkelanjutan.
* Tiga pendekatan implementasi: native (maksimal performa, dua codebase), hybrid (cepat, berbasis WebView), dan cross-platform native seperti React Native (satu codebase, UI native).
* React Native memposisikan diri sebagai solusi cross-platform yang menggunakan JSX dan komponen core RN, dengan jembatan untuk berkomunikasi ke API native sehingga menghasilkan pengalaman pengguna mendekati native.

## 5. Referensi

[1] React Native Docs 0.80: Core Components and Native Components. `https://reactnative.dev/docs/0.80/intro-react-native-components`


## 6. Evaluasi Harian

1. **Jelaskan definisi Mobile App Development sesuai pemahaman anda beserta fokus utama dan output teknisnya!**

2. **Bandingkan perbedaan mendasar antara Web Development dan Mobile App Development dalam aspek target eksekusi, distribusi, dan akses hardware. Berikan contoh implikasi praktis dari perbedaan tersebut dalam pengembangan aplikasi sehari-hari.**

3. **Uraikan tahapan Discovery & Requirement dalam siklus hidup aplikasi mobile. Bagaimana tahap ini memengaruhi keputusan target platform (Android/iOS) dan kebutuhan offline/online?**

4. **Deskripsikan tahapan Perancangan Arsitektur & Teknologi dalam Mobile App Development, khususnya dalam konteks React Native sesuai pemahaman anda. Mengapa pemilihan strategi state management dan navigasi menjadi krusial di tahap ini?**

5. **Jelaskan perbedaan antara pendekatan Native Development dan Hybrid Development dalam pengembangan aplikasi mobile. Sertakan keuntungan serta kekurangan masing-masing, dan berikan contoh framework yang relevan selain dari yang telah disampaikan di materi.**

6. **Apa yang dimaksud dengan Cross-Platform Native Development? Bandingkan keuntungan dan kekurangannya dengan pendekatan native.**

7. **Posisikan React Native dalam ekosistem pengembangan aplikasi mobile. Bagaimana React Native berbeda dari ReactJS dalam hal target, sintaks dasar, dan styling?**

8. **Analisis tantangan utama dalam pengembangan aplikasi mobile dibandingkan dengan web. Bagaimana pendekatan cross-platform seperti React Native mengatasi tantangan ini?**

9. **Uraikan tahapan Pengujian dan Build, Signing, serta Release dalam Mobile App Development menggunakan React Native!**

10. **Berdasarkan penjelasan diatas, jelaskan kenapa React native menjadi pilihan dalam development application mobile saat ini?**

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  
