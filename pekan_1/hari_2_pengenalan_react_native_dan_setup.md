# Hari ke-2: Pengenalan React Native & Setup Environment

## 1. Tujuan Pembelajaran

Setelah menyelesaikan module ini, peserta diharapkan mampu:

- Memahami konsep dasar React Native sebagai framework cross-platform untuk membangun aplikasi mobile native menggunakan JavaScript/TypeScript.
- Membedakan React Native CLI dan Expo, termasuk perbedaan arsitektur, kelebihan/kekurangan, serta skenario penggunaan optimal untuk masing-masing.
- Menginstal dan mengonfigurasi environment development lengkap untuk React Native CLI v0.80 pada platform Android dan iOS, termasuk pemahaman fungsi setiap tool/prasyarat.
- Membuat proyek React Native pertama menggunakan CLI dan menjalankannya di emulator/simulator.
- Mengidentifikasi potensi isu setup umum (seperti konflik versi) dan solusinya berdasarkan dokumentasi resmi.
- Menjelaskan mengapa setup CLI lebih cocok untuk proyek kompleks dibandingkan Expo, serta transisi potensial ke Expo jika diperlukan.
- Memahami struktur folder dan file proyek React Native CLI untuk navigasi yang lebih baik di VS Code.

Tujuan ini membangun fondasi kuat untuk pengembangan selanjutnya, memastikan Anda siap berkontribusi ke proyek real tanpa hambatan teknis awal.

## 2. Materi Pembelajaran

### A. Pengenalan React Native

React Native adalah framework open-source yang dikembangkan oleh Meta (sebelumnya Facebook) sejak 2015, memungkinkan developer membangun aplikasi mobile native untuk iOS dan Android menggunakan JavaScript dan React. Berbeda dengan React (web), React Native merender UI menggunakan komponen native (seperti UIView di iOS atau View di Android), bukan HTML/CSS, sehingga aplikasi terasa "native" dengan performa mendekati aplikasi asli. Keunggulan utama: *code sharing* hingga 90% antar platform, hot reloading untuk development cepat, dan akses ke API native via bridge (JavaScript ke native code).

Di balik layar, React Native menggunakan:

- **Yoga Layout Engine**: Untuk cross-platform styling mirip Flexbox.
- **Metro Bundler**: Packer JS untuk bundling kode dan aset.
- **New Architecture** (diaktifkan default di v0.80): Menggantikan legacy bridge dengan JSI (JavaScript Interface) untuk komunikasi synchronous, mengurangi latency hingga 50% pada operasi kompleks seperti animasi.

React Native v0.80 (rilis 12 Juni 2025) memperkenalkan dukungan React 19.1 (dengan hooks baru seperti `useOptimistic`), perubahan API JS untuk TypeScript yang lebih ketat, dan freezing legacy architecture untuk mendorong migrasi ke Fabric/TurboModules. Ini membuatnya ideal untuk proyek baru di 2025, dengan dukungan jangka panjang hingga 2027.

#### React Native CLI vs Expo: Perbedaan, Kelebihan, Kekurangan, dan Kapan Menggunakan

React Native CLI dan Expo adalah dua workflow utama untuk memulai proyek. React Native CLI adalah tool resmi dari tim React Native untuk generate proyek "bare" (tanpa framework tambahan), sementara Expo adalah framework/build system yang dibangun di atas React Native untuk simplify development.

**Perbedaan Mendasar**:

- **Arsitektur**: CLI menghasilkan proyek dengan akses penuh ke folder native (`ios/` dan `android/`), memungkinkan modifikasi kode Objective-C/Swift (iOS) atau Java/Kotlin (Android). Expo menggunakan "managed workflow" di mana native code disembunyikan; developer berinteraksi via Expo SDK. Expo juga punya "bare workflow" (eject dari managed) yang mirip CLI tapi dengan tools Expo tambahan.
- **Build Process**: CLI butuh build lokal via Xcode/Android SDK tools; Expo gunakan cloud build (EAS Build) atau Expo Go app untuk testing tanpa build.
- **Dependencies**: CLI instal library via npm/yarn + manual linking (autolinking di v0.60+); Expo batasi ke Expo SDK-compatible libs, tapi dukung custom native via config plugins.
- **Development Tools**: CLI gunakan Metro bundler standalone; Expo tambah Expo Dev Tools, OTA updates (over-the-air), dan Expo Router untuk navigation.

**Kelebihan dan Kekurangan** (Berdasarkan Update 2025):

| Aspek | React Native CLI | Expo |
|-------|------------------|------|
| **Kelebihan** | - Akses penuh native code untuk custom modules (e.g., AR/VR integration).<br>- Fleksibilitas tinggi untuk performa tweaking (e.g., custom TurboModules).<br>- No vendor lock-in; mudah migrasi ke native apps.<br>- Build lokal cepat untuk tim besar. | - Setup cepat (5 menit vs. 30+ menit CLI).<br>- Expo Go untuk instant preview tanpa build.<br>- OTA updates untuk fix bugs tanpa resubmit ke store.<br>- Rich ecosystem: 100+ pre-built modules (camera, push notifications).<br>- Beginner-friendly dengan docs interaktif. |
| **Kekurangan** | - Setup kompleks, rawan error konfigurasi.<br>- Build times panjang (10-30 menit per iterasi).<br>- Butuh maintenance native code secara manual.<br>- Kurang dukungan untuk rapid prototyping. | - Dependency pada Expo SDK (update tahunan bisa break code).<br>- Limited native access di managed workflow (harus eject ke bare, yang tambah kompleksitas).<br>- Build cloud gratis terbatas (pro tier $29/bln).<br>- Potensi overhead performa (~5-10% lebih lambat di app kompleks). |

**Kapan Menggunakan Masing-Masing**:

- **Gunakan React Native CLI** jika: Proyek butuh native modules custom (e.g., integrasi hardware khusus seperti Bluetooth LE), performa kritis (gaming apps), atau tim punya expertise native development. Ideal untuk enterprise apps atau saat Expo tidak support lib tertentu. Di 2025, CLI lebih matang dengan New Architecture default.
- **Gunakan Expo** jika: Beginner/rapid prototyping, MVP (Minimum Viable Product), atau app standar (e-commerce, social). 80% apps baru direkomendasikan Expo karena simplicity; eject ke bare jika butuh custom nanti. Mulai dengan Expo, switch ke CLI hanya jika terpaksa.

### B. Setup Environment Menggunakan React Native CLI v0.80

Setup CLI memerlukan prasyarat spesifik untuk compile JS ke native binary. Kita fokus pada Android (dengan command-line tools) dan iOS (macOS only). Versi: Node 20+, JDK 17+, Android SDK 35 (API Level 35), Xcode 16+ (untuk iOS 18 support). Ikuti langkah ini secara berurutan; gunakan `npx react-native doctor` (setelah instal CLI) untuk verifikasi.

#### Prasyarat Umum dan Alasan Kebutuhan

Setiap komponen diperlukan untuk menjembatani JS / TS (React) ke native runtime:

| Komponen | Versi Rekomendasi (2025) | Alasan Diperlukan |
|----------|---------------------------|-------------------|
| **Node.js** | 20.18+ (LTS) | Runtime JS untuk Metro Bundler (packaging kode) dan npm/yarn (dependency management). Tanpa ini, tak bisa run `npx react-native init`. Install via NodeSource atau Homebrew; verifikasi `node -v`. |
| **Watchman** | 2025.04.21.00+ | File watching tool dari Facebook untuk detect perubahan kode secara efisien. Diperlukan untuk hot reloading; install via Homebrew (`brew install watchman`). |
| **Yarn** (opsional, tapi direkomendasikan) | 4.5+ | Package manager lebih cepat dan reliable daripada npm untuk monorepo React Native. Gunakan untuk `yarn install`; install via `npm install -g yarn`. |

#### Setup untuk Android (Menggunakan Command-Line Tools)

1. **Install JDK 17+**: Oracle JDK atau OpenJDK. Alasan: Gradle (build system Android) butuh Java compiler untuk kompilasi native code. Download dari Oracle, set `JAVA_HOME` env var (e.g., `export JAVA_HOME=/usr/lib/jvm/java-17-openjdk` di Linux). Verifikasi: `java -version`.

2. **Download Android SDK Command-Line Tools**:
   - Kunjungi <https://developer.android.com/studio#command-tools>, download "Command line tools only" (untuk Windows/Linux/macOS).
   - Alasan: Paket ini menyediakan `sdkmanager` dan tools dasar untuk manage SDK tanpa IDE berat seperti Android Studio, cocok untuk VS Code workflow.

3. **Extract Paket**:
   - Extract ke direktori pilihan, e.g., `~/android-sdk` (buat folder jika belum ada).
   - Pindahkan isi extract ke `~/android-sdk/cmdline-tools/latest/` agar struktur: `cmdline-tools/latest/bin/sdkmanager`.
   - Alasan: Memastikan tools seperti `sdkmanager` accessible dari command line.

4. **Set Environment Variables**:
   - Set `ANDROID_HOME` ke root SDK: `export ANDROID_HOME=$HOME/android-sdk` (tambah ke `~/.bashrc` atau `~/.zshrc` untuk permanen).
   - Tambah ke PATH: `$ANDROID_HOME/cmdline-tools/latest/bin`, `$ANDROID_HOME/platform-tools`, `$ANDROID_HOME/tools` (jika ada).
   - Verifikasi: `echo $ANDROID_HOME` dan `sdkmanager --version`.
   - Alasan: Memungkinkan akses tools seperti `adb` (dari platform-tools) dan `sdkmanager` dari mana saja di terminal.

5. **Install Paket SDK via sdkmanager**:
   - Jalankan: `sdkmanager "platforms;android-35" "build-tools;35.0.0" "platform-tools"`.
   - Accept licenses: `sdkmanager --licenses` (ketik 'y' untuk semua).
   - **Mengapa Setiap SDK Diperlukan**:
     - **SDK Platforms (android-35)**: Menyediakan API libraries dan system images untuk build app targeting Android 15 (API 35). Diperlukan React Native untuk kompatibilitas dengan fitur OS terbaru, seperti permissions baru di Android 15, memastikan app compile tanpa error API mismatch.
     - **Build Tools (35.0.0)**: Berisi tools seperti `aapt2` (resource compiler), `apksigner` (signing APK), dan `zipalign` (optimasi APK). Esensial untuk proses build React Native, di mana Gradle menggunakan ini untuk assemble native code, resource, dan JS bundle menjadi APK/AAB yang valid.
     - **Platform Tools**: Termasuk `adb` (Android Debug Bridge) untuk install app ke device/emulator, `logcat` untuk debugging logs. Diperlukan untuk testing: `adb install` push app ke device, dan `adb logcat` monitor error runtime di React Native bridge.

Notes: Jika butuh emulator, install `emulator` via `sdkmanager "emulator"` dan buat AVD manual via `avdmanager`. Untuk VS Code, instal extension "React Native Tools" untuk debugging.

#### Setup untuk iOS (macOS Only)

1. **Install Xcode 16+**: Dari App Store. Alasan: Compiler Swift/Objective-C, Simulator iOS, dan signing tools untuk build IPA. Buka Xcode > Preferences > Locations > Command Line Tools: Install jika belum.
2. **Install CocoaPods 1.15+**: `sudo gem install cocoapods`. Alasan: Dependency manager untuk iOS native libs (e.g., auto-link React modules). Jalankan `pod setup` setelah instal; verifikasi `pod --version`.

#### Install React Native CLI v0.80 dan Buat Proyek

1. **Hapus package lama(jika ada)**: jalankan `npm uninstall -g react-native-cli @react-native-community/cli`.
2. **Install package**: Install React Native Community CLI untuk generate project baru, jalankan `npx @react-native-community/cli@latest init AwesomeProject --version 0.80.0` (ganti *AwesomeProject* dengan nama project anda sendiri)
3. Pastikan nama project menggunakan camelCase / snake_case / PascalCase.

**Catatan Keamanan**: Gunakan VPN jika di belakang proxy; hindari root/sudo kecuali diperlukan.

#### Struktur Folder dan File Proyek React Native CLI

Setelah membuat proyek React Native menggunakan CLI, Anda akan melihat struktur folder standar yang memisahkan kode JS/TypeScript dari native code. Ini memudahkan navigasi di VS Code (gunakan extension untuk syntax highlighting). Berikut ringkasan struktur tipikal untuk proyek "MyFirstApp":

```bash
MyFirstApp/
├── android/                  # Kode native Android (Java/Kotlin, Gradle config)
│   ├── app/                  # Sumber kode utama app Android
│   │   ├── src/main/         # Java/Kotlin files, assets, res (resources seperti drawable)
│   │   └── build.gradle      # Konfigurasi build app (dependencies, signing)
│   ├── build.gradle          # Konfigurasi Gradle root (plugins, repositories)
│   ├── settings.gradle       # Include modules (e.g., app)
│   └── gradlew               # Wrapper script untuk run Gradle commands
├── ios/                      # Kode native iOS (Swift/Objective-C, Xcode project) – macOS only
│   ├── MyFirstApp/           # Xcode project files
│   │   ├── AppDelegate.mm    # Entry point iOS app
│   │   └── MyFirstApp.xcworkspace # Workspace untuk CocoaPods
│   ├── Podfile               # Dependencies iOS via CocoaPods
│   └── MyFirstAppTests/      # Unit tests iOS
├── src/ or ./ (root JS files) # Kode JavaScript/TypeScript utama (bisa custom)
│   ├── App.js                # Komponen utama app (render root view)
│   ├── index.js              # Entry point JS (register App component ke native)
│   ├── components/           # Folder opsional untuk reusable UI components
│   ├── screens/              # Folder opsional untuk screen/page components
│   └── assets/               # Aset statis (images, fonts, JSON)
├── node_modules/             # Dependencies npm/yarn (jangan edit manual)
├── package.json              # Metadata proyek, scripts (e.g., "start"), dependencies React Native
├── metro.config.js           # Konfigurasi Metro Bundler (bundling JS, resolver paths)
├── babel.config.js           # Konfigurasi Babel (transpile JSX/TS ke JS)
├── .gitignore                # Rules untuk Git (ignore node_modules, build outputs)
├── .eslintrc.js              # Konfigurasi ESLint untuk code linting (opsional)
├── tsconfig.json             # Konfigurasi TypeScript (jika pakai TS)
└── README.md                 # Dokumentasi proyek
```

**Penjelasan Singkat**:

- **android/ dan ios/**: Esensial untuk build native; edit jika butuh custom modules (e.g., tambah permissions di AndroidManifest.xml).
- **JS Files (App.js, index.js)**: Core React Native; edit di VS Code untuk development.
- **Config Files (package.json, metro.config.js)**: Handle dependencies dan bundling; modifikasi untuk custom setup.
Struktur ini mendukung cross-platform: Ubah JS sekali, build ke dua platform. Gunakan VS Code's Explorer untuk navigasi cepat.

## 3. Contoh Implementasi

### A. Contoh: Membuat dan Menjalankan Proyek Pertama

Asumsikan setup selesai. Kita buat app sederhana "Hello World" dengan tombol.

1. **Buat Proyek**:

   ```bash
   npx @react-native-community/cli@latest init AwesomeProject --version 0.80
   cd AwesomeProject
   npx react-native run-android
   ```

2. **Edit App.js** (ganti isi default):

   ```javascript
   import React, { useState } from 'react';
   import { View, Text, Button, StyleSheet } from 'react-native';

   const App = () => {
     const [count, setCount] = useState(0);
     return (
       <View style={styles.container}>
         <Text style={styles.text}>Hello, React Native v0.80!</Text>
         <Button title={`Count: ${count}`} onPress={() => setCount(count + 1)} />
       </View>
     );
   };

   const styles = StyleSheet.create({
     container: { flex: 1, justifyContent: 'center', alignItems: 'center' },
     text: { fontSize: 20, marginBottom: 20 },
   });

   export default App;
   ```


3. **Jalankan di Android**:
   - Start emulator via `avdmanager` (jika setup) atau hubungkan device.
   - `npx react-native run-android`.
   - Metro bundler otomatis start di port 8081. Alasan: Compile JS ke bundle, push ke device via ADB (dari platform-tools).

4. **Jalankan di iOS** (macOS):
   - `npx pod-install` (instal dependencies CocoaPods).
   - `npx react-native run-ios`.
   - Alasan: Build via Xcode, launch di Simulator.

**Troubleshooting Contoh**: Jika error "SDK not found", cek `ANDROID_HOME`. Hot reload: Shake device > Reload. Ini verifikasi full stack berjalan.

## 4. Rangkuman

Pada Hari ke-2, kita telah mengenal React Native sebagai bridge JS-native untuk app cross-platform, dengan v0.80 membawa inovasi seperti React 19.1 untuk pengembangan lebih stabil. React Native CLI unggul di fleksibilitas native (pilih jika custom modules), sementara Expo di simplicity (pilih untuk prototyping cepat)—mulai dengan Expo jika ragu, eject nanti. Setup CLI melibatkan Node (JS runtime), JDK/Android SDK cmdline (build tools minimal untuk VS Code), Xcode/CocoaPods (iOS deps), dan Watchman (file watching), semuanya esensial untuk kompilasi dan reloading. Struktur proyek memisahkan native (android/ios) dari JS core, ideal untuk VS Code. Praktikkan dengan proyek "HelloWorld" untuk solidifikasi. Kendala umum: Versi mismatch—selalu jalankan `react-native doctor`. Lanjut ke Hari ke-3 untuk komponen dasar. Selamat coding, dan eksplor docs resmi untuk deep dive!

## Referensi

- [9] React Native 0.80 - React 19.1, JS API Changes, Freezing Legacy Architecture. React Native Blog. Diakses dari <https://reactnative.dev/blog/2025/06/12/react-native-0.80> (12 Juni 2025).
- [15] Expo Documentation: Expo vs. Bare React Native CLI – Key Differences, Pros/Cons, and When to Use Each. Expo Docs. Diakses dari <https://docs.expo.dev/versions/latest/> (Update 2025).

## Evaluasi Harian

1. **Jelaskan konsep dasar React Native sebagai framework cross-platform, termasuk perbedaan utamanya dengan React untuk web. Sertakan penjelasan singkat tentang peran New Architecture di React Native v0.80 dan bagaimana hal itu memengaruhi performa aplikasi mobile.**

2. **Bandingkan React Native CLI dan Expo dari segi arsitektur serta proses build. Diskusikan satu kelebihan dan satu kekurangan masing-masing, lalu berikan contoh skenario proyek di mana Anda akan memilih salah satu di atas, beserta alasannya.**  

3. **Dalam setup environment Android menggunakan command-line tools, jelaskan mengapa SDK Platforms (android-35), Build Tools (35.0.0), dan Platform Tools masing-masing diperlukan untuk React Native. Berikan contoh bagaimana ketidakhadiran salah satu komponen tersebut dapat menyebabkan masalah saat menjalankan proyek pertama Anda di VS Code.**

4. **Bahas prasyarat umum setup React Native CLI v0.80, seperti Node.js, Watchman, dan Yarn, termasuk alasan mengapa masing-masing diperlukan untuk menjembatani JavaScript ke native runtime.**

5. **Deskripsikan struktur folder utama dalam proyek React Native CLI, termasuk fungsi folder `android/`, `ios/`, dan file-file JS seperti `App.js` serta `metro.config.js`. Jelaskan bagaimana struktur ini mendukung pengembangan cross-platform dan navigasi di VS Code.**
