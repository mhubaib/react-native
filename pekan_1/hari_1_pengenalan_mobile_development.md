# Hari ke-1: Pengenalan Konsep Mobile Development

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

* Memahami perbedaan mendasar antara pengembangan aplikasi *native*, *hybrid*, dan *cross-platform*.
* Menjelaskan posisi React Native dalam ekosistem pengembangan aplikasi *mobile*.
* Mengidentifikasi tantangan dan keuntungan utama dalam pengembangan aplikasi *mobile* dibandingkan dengan pengembangan *web*.

## 2. Materi Pembelajaran

### A. Definisi Mobile App Development

Mobile App Development adalah proses end-to-end untuk merancang, membangun, menguji, merilis, dan memelihara aplikasi yang berjalan di perangkat mobile seperti smartphone dan tablet. Aplikasi ini ditargetkan untuk platform spesifik (Android/iOS) dan memanfaatkan kemampuan perangkat seperti kamera, GPS, sensor gerak, penyimpanan lokal, notifikasi, hingga proses latar belakang.

* Fokus utama: pengalaman pengguna yang selaras dengan platform, performa yang stabil pada perangkat beragam, serta kepatuhan terhadap kebijakan distribusi (Play Store/App Store).
* Output teknis: paket rilis `APK/AAB` (Android) atau `IPA` (iOS), sertifikat penandatanganan (signing), konfigurasi izin (permissions), dan metadata rilis (ikon, deskripsi, kebijakan privasi).
* Pendekatan implementasi: native (Kotlin/Swift), hybrid (WebView), atau cross-platform native (React Native/Flutter) yang memanfaatkan komponen UI native.

### B. Perbedaan Web Development vs Mobile App Development

Terdapat berbagai konsep umum (UI, state, networking), pengembangan aplikasi web dan aplikasi mobile memiliki perbedaan mendasar pada target lingkungan eksekusi, distribusi, dan akses fitur perangkat.

| Aspek | Web Development | Mobile App Development |
| :--- | :--- | :--- |
| Target eksekusi | Browser (DOM, CSS) | Runtime native (Android/iOS, komponen UI native) |
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

### C. Tahapan dalam Mobile App Development

Berikut tahapan umum yang direkomendasikan dalam siklus hidup aplikasi mobile. Contoh tools dan praktik menyesuaikan ekosistem React Native (RN):

1) Discovery & Requirement

* Klarifikasi masalah bisnis, dan target platform (Android/iOS).

* Definisikan use-case utama, alur pengguna, dan kebutuhan offline/online.

2) Perancangan Arsitektur & Teknologi

* Pilih pendekatan (RN vs native), struktur modul, dan strategi state management.

* Tentukan jenis navigasi (Stack, Bottom Tabs, Drawer, material top tabs), arsitektur navigasi, dan strategi error handling/logging.

3) Desain UI/UX

* Rancang antarmuka adaptif terhadap variasi layar, DPI, dan gesture.

* Siapkan design system, komponen reusable, dan alur aksesibilitas.

4) Setup Lingkungan & Proyek

* Siapkan Node.js, paket manajer (`npm`/`yarn`), Android Studio, Xcode (macOS untuk build iOS).

* Inisialisasi proyek RN, konfigurasi `babel`, `metro`, dan dependensi inti (navigasi, HTTP client, storage).

5) Implementasi Fitur

* Bangun layar inti menggunakan Core Components RN (`View`, `Text`, `Image`, `ScrollView`).

* Integrasikan networking (REST/GraphQL), penyimpanan lokal (AsyncStorage/SQLite), dan permission (kamera, lokasi) sesuai kebutuhan.
* Optimalkan performa: memoization, lazy loading, list virtualization.

6) Pengujian

* Unit test (Jest) untuk util dan komponen, snapshot untuk UI stabil.

* E2E (Detox) di emulator/simulator, uji manual di perangkat fisik beragam.

7) Build, Signing, dan Release

* Android: keystore, `gradlew assembleRelease`, proguard/obfuscation bila perlu.

* iOS: provisioning profile, `Archive` via Xcode, App Transport Security.
* Siapkan metadata rilis, kebijakan privasi, dan materi store (ikon, screenshot).

8) Distribusi & Operasional

* Publikasikan ke Play Console/App Store Connect, pantau review dan kepatuhan.

* Pasang observability: crash reporting (Sentry), analytics, dan remote config.

9. Pemeliharaan & Iterasi

* Tindaklanjuti bug, kompatibilitas OS baru, dan optimasi performa.

* Automasi CI/CD (fastlane, GitHub Actions) untuk build dan distribusi berkelanjutan.

### D. Tiga Pendekatan Utama dalam Mobile Development

Pengembangan aplikasi *mobile* secara umum terbagi menjadi tiga kategori utama:

#### 1. Native Development

* **Definisi:** Membangun aplikasi menggunakan bahasa pemrograman dan *tool* yang spesifik untuk *platform* target (misalnya, Kotlin/Java dengan Android Studio untuk Android, dan Swift/Objective-C dengan Xcode untuk iOS).
* **Keuntungan:** Performa maksimal, akses penuh ke semua fitur perangkat keras (*hardware*) dan API *platform*, serta pengalaman pengguna (*user experience*) yang paling otentik.
* **Kekurangan:** Membutuhkan dua basis kode (*codebase*) yang terpisah, yang berarti waktu pengembangan dan biaya pemeliharaan yang lebih tinggi.

#### 2. Hybrid Development

* **Definisi:** Membangun aplikasi menggunakan teknologi *web* (HTML, CSS, JavaScript) dan membungkusnya dalam kontainer *native* (WebView). Contoh *framework* populer adalah Apache Cordova atau Ionic.
* **Keuntungan:** Satu basis kode untuk berbagai *platform*, pengembangan cepat, dan familiar bagi pengembang *web*.
* **Kekurangan:** Performa yang sering kali lebih rendah dari *native*, pengalaman pengguna yang kurang otentik, dan akses terbatas ke API *native* (tergantung pada *plugin*).

#### 3. Cross-Platform Native Development (React Native)

* **Definisi:** Membangun aplikasi menggunakan satu bahasa (JavaScript/TypeScript) dan *framework* tunggal, tetapi aplikasi yang dihasilkan *dirender* ke komponen UI *native* dari *platform* target (bukan WebView). Contoh utama adalah React Native dan Flutter.
* **Keuntungan:** Satu basis kode yang menghasilkan aplikasi dengan performa mendekati *native* dan pengalaman pengguna yang otentik, karena menggunakan komponen UI *native* [1].
* **Kekurangan:** Memerlukan jembatan (*bridge*) untuk berkomunikasi antara kode JavaScript dan API *native*, yang dapat menjadi hambatan performa di kasus tertentu.

### E. Posisi React Native

React Native (RN) adalah *framework* yang memungkinkan Anda membangun aplikasi *mobile* *cross-platform* dengan filosofi dan sintaks yang sama seperti ReactJS.

| Aspek | ReactJS (Web) | React Native (Mobile) |
| :--- | :--- | :--- |
| **Target** | DOM (Document Object Model) | Komponen UI Native (View, Text, Image, dll.) |
| **Sintaks Dasar** | Sama (JSX) | Sama (JSX) |
| **Styling** | CSS | JavaScript Objects (mirip Flexbox dan CSS *subset*) |
| **Komponen** | `<div>`, `<span>`, `<p>`, dll. | `<View>`, `<Text>`, `<Image>`, dll. |

**Inti dari React Native:** RN tidak menghasilkan kode *web* yang berjalan di *browser* tersembunyi. Sebaliknya, kode JavaScript Anda berkomunikasi dengan *runtime* *native* melalui sebuah *bridge* untuk membuat dan memanipulasi elemen UI *native* yang sesungguhnya. Inilah yang membuat aplikasi RN terasa seperti aplikasi *native* sejati.

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

6. **Apa yang dimaksud dengan Cross-Platform Native Development? Bandingkan keuntungan dan kekurangannya dengan pendekatan native, serta jelaskan peran 'bridge' dalam arsitekturnya.**

7. **Posisikan React Native dalam ekosistem pengembangan aplikasi mobile. Bagaimana React Native berbeda dari ReactJS dalam hal target, sintaks dasar, dan styling?**

8. **Analisis tantangan utama dalam pengembangan aplikasi mobile dibandingkan dengan web. Bagaimana pendekatan cross-platform seperti React Native mengatasi tantangan ini?**

9. **Uraikan tahapan Pengujian dan Build, Signing, serta Release dalam Mobile App Development menggunakan React Native!**

10. **Berdasarkan penjelasan diatas, jelaskan kenapa React native menjadi pilihan dalam development application mobile saat ini?**

