# Hari Ke-22: Build APK & AAB

## Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran, siswa diharapkan mampu:

* Paham perbedaan APK (file tunggal untuk distribusi manual) dan AAB (bundle untuk Play Store, generate APK otomatis per device), serta implementasikan build APK debug/release dan AAB dengan Gradle, memastikan app siap distribusi tanpa error.
* Implementasikan proses signing APK/AAB dengan keystore, termasuk generate key, konfigurasi gradle.properties, dan best practices untuk security seperti key rotation.
* Kelola optimasi build seperti ProGuard/R8 untuk minification, multidex untuk large apps, dan bundle analysis untuk ukuran file, sambil memastikan kompatibilitas dengan RN 0.75+.
* Terapkan praktik terbaik seperti build variants (debug vs release), CI/CD integration (e.g., GitHub Actions), dan testing build di multiple devices, untuk workflow yang efisien.
* Analisis dan selesaikan troubleshooting umum seperti Gradle sync failures, signing errors, atau APK crash on install, dengan langkah debugging menggunakan logcat atau Android Studio.
* Tangani perbedaan platform seperti APK untuk sideloading vs AAB untuk store, untuk build yang robust dan compliant dengan Play Console policies.
* Buat prototipe build APK/AAB dari project RN, siap upload ke Play Store atau distribusi manual.

---

## Bagian I: Fondasi dan Strategi Format Distribusi Android

Tujuan utama dari fase release build adalah menghasilkan artefak biner yang teroptimasi, aman (ditandatangani), dan siap diunggah ke Google Play Store atau didistribusikan secara internal. Proses ini melibatkan pemilihan format biner yang tepat sesuai dengan tujuan distribusi.

### I.A. Memahami Artefak Rilis: APK vs. AAB

Dalam ekosistem Android modern, pengembang berurusan dengan dua format paket utama. Penting untuk dicatat bahwa Google Play Store saat ini mewajibkan aplikasi baru untuk menggunakan format Android App Bundle (AAB) untuk publikasi, menjadikannya standar industri.

#### 1. Android Package Kit (APK)

APK adalah format kemasan instalasi final yang dapat diinstal langsung ke perangkat Android. Secara tradisional, APK rilis adalah biner tunggal, seringkali disebut APK universal, yang memuat semua kode terkompilasi, sumber daya, dan native libraries (ABI) yang diperlukan untuk menjalankan aplikasi di berbagai arsitektur perangkat.

#### 2. Android App Bundle (AAB)

AAB merupakan format publikasi yang lebih baru. Berbeda dengan APK, AAB berisi semua kode terkompilasi dan sumber daya aplikasi, namun proses pembuatan APK final dan penandatanganan Google Play ditunda (deferensi) hingga saat pengunduhan oleh pengguna.
AAB memungkinkan Play Store menghasilkan Split APKs yang lebih kecil dan disesuaikan dengan perangkat pengguna.

---

### Perbandingan Kunci Format Distribusi

Optimasi yang disediakan oleh AAB memberikan keuntungan substansial dalam hal ukuran dan modularitas, sebagaimana dirangkum di bawah ini.

#### Tabel I.1: Perbandingan Fitur Kunci APK dan AAB

| Fitur Kunci     | APK (Android Package Kit)                                               | AAB (Android App Bundle)                                       |
| --------------- | ----------------------------------------------------------------------- | -------------------------------------------------------------- |
| Definisi        | Format instalasi dan distribusi final.                                  | Format publikasi yang menunda generasi APK.                    |
| Output Build    | File biner tunggal (biasanya universal).                                | Kumpulan kode dan sumber daya.                                 |
| Optimasi Ukuran | Memerlukan ABI Splits manual (kompleks, warisan).                       | Menggunakan Split APKs (otomatis di Play Store).               |
| Ukuran Download | Cenderung lebih besar karena memuat ABI/Sumber daya yang tidak relevan. | Lebih kecil (hingga 20%) karena hanya mengirimkan APK relevan. |
| Penandatanganan | Ditandatangani oleh Pengembang.                                         | Ditandatangani oleh Google Play (App Signing by Google Play).  |

---

### I.B. Pergeseran Paradigma Split APKs

Keunggulan fungsional utama AAB adalah kemampuannya untuk bekerja dengan konsep Split APKs. Dalam struktur AAB yang modular dan terorganisir, Google Play dapat menganalisis biner aplikasi dan otomatis menghasilkan APK terbagi untuk fungsionalitas yang berbeda, seperti:

* dukungan multi-bahasa,
* arsitektur native spesifik.

Secara historis, pengembang harus mengatur ABI splits secara manual di build.gradle untuk setiap arsitektur (armeabi-v7a, arm64-v8a, x86).
Dengan AAB, Play Store hanya mengunduh kode native yang relevan, membuat ukuran unduhan lebih kecil.

---

## Bagian II: Persiapan Keamanan dan Konfigurasi Kredensial (Signing)

Setiap aplikasi yang akan didistribusikan wajib ditandatangani menggunakan sertifikat digital (keystore). Gradle bertanggung jawab terhadap proses ini.

### II.A. Aturan Implementasi Penyimpanan Kredensial

Kredensial tidak boleh disimpan dalam source code atau VCS.
Solusi aman: simpan di `gradle.properties`.

#### 1. Lokasi File gradle.properties

* Global (Direkomendasikan)
  `~/.gradle/gradle.properties`

* Lokal proyek (untuk CI/CD)
  `android/gradle.properties` → wajib ditambahkan ke `.gitignore`

---

#### 2. Struktur Kredensial di gradle.properties

```kt
# Kredensial Signing
MYAPP_UPLOAD_STORE_FILE=my_release_key.keystore
MYAPP_UPLOAD_KEY_ALIAS=my-key-alias
MYAPP_UPLOAD_STORE_PASSWORD=supersecretstorepassword
MYAPP_UPLOAD_KEY_PASSWORD=supersecretkeypassword
```

---

### II.B. Properti Build System Kritis di gradle.properties

#### 1. Pengaturan Cache dan Bundling

* `org.gradle.configureondemand` **tidak boleh true**
* `org.gradle.configuration-cache=true` dapat mempercepat build

#### 2. Optimasi Arsitektur

Saat development sering menggunakan:

```kt
reactNativeArchitectures=arm64-v8a
```

**HARUS dihapus saat release** agar mendukung semua ABI.

---

## Bagian III: Konfigurasi Sistem Build di android/app/build.gradle

### III.A. Implementasi Konfigurasi Penandatanganan (signingConfigs)

```groovy
android {
    signingConfigs {
        release {
            if (project.hasProperty('MYAPP_UPLOAD_STORE_FILE')) {
                storeFile file(MYAPP_UPLOAD_STORE_FILE)
                storePassword MYAPP_UPLOAD_STORE_PASSWORD
                keyAlias MYAPP_UPLOAD_KEY_ALIAS
                keyPassword MYAPP_UPLOAD_KEY_PASSWORD
            }
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

---

### III.B. Optimasi Kode dan Sumber Daya (R8 dan ProGuard)

#### 1. Mengaktifkan Shrinking

```groovy
buildTypes {
    release {
        minifyEnabled true
        isShrinkResources true
        proguardFiles(
            getDefaultProguardFile('proguard-android-optimize.txt'),
            'proguard-rules.pro'
        )
    }
}
```

---

#### 2. Aturan ProGuard untuk Stabilitas Rilis

```kt
-keep class com.facebook.jni.** { *; }
```

---

### III.C. Dukungan Aplikasi Besar (Multidex)

#### 1. Konfigurasi defaultConfig

```groovy
multiDexEnabled true
```

#### 2. Dependensi Multidex

```groovy
implementation 'androidx.multidex:multidex:2.0.1'
```

---

### III.D. Konfigurasi Plugin React Native Kustom

```groovy
apply plugin: "com.facebook.react"

react {
    enableBundleCompression = false
}
```

---

## Bagian IV: Proses Eksekusi Build dan Distribusi

### IV.A. Perintah Build Rilis Resmi

#### 1. Membangun AAB

```bash
npx react-native build-android --mode=release
```

Output:

```md
android/app/build/outputs/bundle/release/app-release.aab
```

#### 2. Membangun APK Release

```bash
cd android && ./gradlew assembleRelease
```

Output:

```md
android/app/build/outputs/apk/release/app-release.apk
```

#### 3. Pembersihan Cache

```bash
cd android && ./gradlew clean
```

---

### IV.B. Konversi AAB ke APK Universal (Bundletool)

```bash
bundletool build-apks --bundle=/path/to/app-release.aab \
  --output=/path/to/my_app.apks \
  --mode=universal
```

---

## IV.C. Praktik Terbaik Lanjutan: CI/CD dan Troubleshooting

### 1. Integrasi CI/CD

Gunakan environment secrets untuk kredensial.

---

### 2. Troubleshooting Umum

| Masalah             | Penyebab & Solusi                                       |
| ------------------- | ------------------------------------------------------- |
| Gradle Sync Failed  | `sdk.dir` tidak ditemukan.                              |
| Release Build Crash | R8 menghapus kelas penting → tambahkan aturan ProGuard. |
| Bundel JS Hilang    | `configureondemand=true` → harus dihapus.               |
| Cache Korup         | Hapus folder `.gradle` di home.                         |

---

## Ringkasan Konfigurasi Sistem Build Esensial

| Blok                     | Lokasi             | Baris Kritis                       | Tujuan              |
| ------------------------ | ------------------ | ---------------------------------- | ------------------- |
| Kredensial Signing       | gradle.properties  | MYAPP_UPLOAD_...                   | keamanan            |
| Disable Config On Demand | gradle.properties  | org.gradle.configureondemand=false | memastikan bundling |
| signingConfigs           | build.gradle       | if(hasProperty...)                 | memuat kredensial   |
| buildTypes.release       | build.gradle       | signingConfig                      | attach signing      |
| Optimasi R8              | build.gradle       | minifyEnabled                      | shrinking           |
| Multidex                 | build.gradle       | multiDexEnabled                    | >64k method fix     |
| Aturan JNI keep          | proguard-rules.pro | -keep class com.facebook.jni       | cegah crash         |

---

## Rangkuman Materi

Modul ini membahas proses membangun APK dan AAB melalui Gradle, mencakup:

* signing
* optimasi build (R8, multidex)
* troubleshooting, dan konfigurasi build esensial untuk memastikan aplikasi siap upload ke Play Store.

---

## Referensi

* [Perbedaan AAB dan APK, keunggulan AAB, dan mekanisme split APK](https://www.hexnode.com/mobile-device-management/help/what-is-the-difference-between-aab-and-apk-file-for-android/)
* [Android App Bundle (AAB) dan *Split APK Generation*](https://www.hexnode.com/mobile-device-management/help/what-is-the-difference-between-aab-and-apk-file-for-android/)
* [Konfigurasi Manual *ABI Splits* dan `versionCodeOverride` di Gradle (Metode Warisan)](https://stackoverflow.com/questions/36286458/android-gradle-apk-splits-setting-versioncode)
* [Mengaktifkan Multidex untuk mengatasi Batas 64K Metode](https://rnfirebase.io/enabling-multidex)
* [Optimasi Kecepatan Build Gradle dan Penggunaan `reactNativeArchitectures`](https://reactnative.dev/docs/build-speed)
* [Panduan Lengkap Keystore, Konfigurasi *Signing*, dan Perintah Build AAB](https://reactnative.dev/docs/signed-apk-android)
* [Mengaktifkan Code Shrinking (R8) dan Resource Shrinking](https://developer.android.com/topic/performance/app-optimization/enable-app-optimization)
* [Pengaturan Configuration Caching untuk Peningkatan Kecepatan *Build*](https://reactnative.dev/docs/build-speed)
* [Konfigurasi Kustom Plugin Gradle React Native](https://reactnative.dev/docs/react-native-gradle-plugin)
* [Peran R8 sebagai *Code Shrinker* Android](https://developer.android.com/topic/performance/app-optimization/enable-app-optimization)
* [Pengelolaan *ABI Splits* Manual dan Kompleksitas *Versioning* (Metode Warisan):](https://stackoverflow.com/questions/36286458/android-gradle-apk-splits-setting-versioncode)
* [Solusi Error Keystore/Signing Config di Gradle](https://stackoverflow.com/questions/43021047/react-native-keystore-file-not-set-for-signing-config-release)
* [Troubleshooting Gradle Sync Failed dan Lokasi SDK](https://stackoverflow.com/questions/35073409/how-to-fix-gradle-sync-errors-setting-up-new-react-native-project-for-android-an)
* [Solusi *Crash Release Build* (Aturan ProGuard untuk JNI):](https://stackoverflow.com/questions/60927048/react-native-app-release-build-crashes-on-start-works-fine-in-debug-why)
* [Konversi AAB ke Universal APK untuk *Sideloading* menggunakan Bundletool](https://stackoverflow.com/questions/53040047/generate-an-apk-file-from-an-aab-file-android-app-bundle)

---
