# Hari ke-20 - Akses Fitur Perangkat: Biometrik & Autentikasi Lokal

## 1\. Tujuan Pembelajaran

Setelah menyelesaikan materi ini, Anda diharapkan mampu:

* **Memahami Konsep:** Mengerti cara kerja autentikasi biometrik (sidik jari, wajah) sebagai gerbang verifikasi keamanan, serta perbedaannya dengan penyimpanan password.
* **Implementasi Library:** Menggunakan library **`@sbaiahmed1/react-native-biometrics`** (versi *fork* yang lebih stabil) untuk menangani fitur biometrik di iOS dan Android.
* **Menguasai API & Opsi:** Memahami secara mendalam metode-metode utama, parameter konfigurasi, dan objek respon yang dihasilkan oleh library.
* **Kustomisasi UX:** Mengatur tampilan *prompt* biometrik (pesan, tombol batal) khususnya untuk Android agar informatif dan user-friendly.
* **Integrasi Keamanan (Keychain):** Menggabungkan biometrik dengan `react-native-keychain` untuk menciptakan sistem login yang cepat namun tetap aman (Hybrid Auth).
* **Manajemen Error & Fallback:** Menangani kasus ketika sensor tidak tersedia, user belum mendaftarkan sidik jari, atau verifikasi gagal.

-----

## 2\. Materi Pembelajaran

Kita menggunakan library **`@sbaiahmed1/react-native-biometrics`**. Library ini adalah jembatan komunikasi antara kode JavaScript React Native Anda dengan fitur keamanan native (Face ID di iOS dan BiometricPrompt di Android).

### A. Persiapan (Setup) & Izin

**Instalasi:**

```bash
npm install @sbaiahmed1/react-native-biometrics
```

**Izin (Permissions):**

* **iOS (`Info.plist`):**
    Wajib ada. Jika tidak, aplikasi akan *crash* saat fitur dipanggil.

    ```xml
    <key>NSFaceIDUsageDescription</key>
    <string>Kami membutuhkan Face ID untuk memverifikasi identitas Anda secara aman.</string>
    ```

* **Android (`AndroidManifest.xml`):**

    ```xml
    <uses-permission android:name="android.permission.USE_BIOMETRIC" />
    ```

### B. API, Method, & Options (Penjelasan Mendalam)

Sebelum menggunakan fungsi-fungsi di bawah, Anda harus menginisialisasi *instance* dari library:

```javascript
import ReactNativeBiometrics from '@sbaiahmed1/react-native-biometrics';
const rnBiometrics = new ReactNativeBiometrics(); // Inisialisasi
```

Berikut adalah rincian metode utama yang tersedia:

#### 1\. `isSensorAvailable()`

Metode ini digunakan untuk memeriksa apakah perangkat keras (HP) memiliki sensor biometrik dan apakah user sudah mengaktifkannya.

* **Input:** Tidak ada.
* **Output (Promise Object):**

    ```javascript
    {
      available: boolean,      // true jika sensor ada DAN bisa dipakai
      biometryType: string,    // Jenis sensor yang terdeteksi
      error: string            // Pesan error (jika ada)
    }
    ```

* **Nilai `biometryType`:**
  * `'TouchID'` (iOS: Fingerprint)
  * `'FaceID'` (iOS: Wajah)
  * `'Biometrics'` (Android: Bisa Wajah atau Fingerprint, digabung jadi satu istilah)

#### 2\. `simplePrompt(options)`

Metode utama untuk memunculkan dialog/popup sistem yang meminta user menempelkan jari atau memindai wajah.

* **Input (`options` Object):**

| Opsi | Tipe | Wajib? | Deskripsi & Fungsi |
| :--- | :--- | :--- | :--- |
| `promptMessage` | `string` | **Ya** | Judul atau pesan utama yang muncul di dialog. Gunakan kalimat aksi, misal: *"Konfirmasi Pembayaran"* atau *"Login Cepat"*. |
| `cancelButtonText`| `string` | **Ya (Android)** | Teks untuk tombol pembatalan. Di Android, tombol ini wajib ada agar user tidak terjebak di layar prompt. Contoh: *"Batal"* atau *"Gunakan PIN"*. |
| `fallbackPromptMessage` | `string` | Tidak | (iOS Only) Pesan yang muncul di tombol fallback jika scan gagal pertama kali. |

* **Output (Promise Object):**

    ```javascript
    {
      success: boolean, // true jika sidik jari/wajah COCOK
      error: string     // Berisi pesan jika user cancel atau gagal verifikasi
    }
    ```

#### 3\. `createKeys()` & `deleteKeys()` (Info Tambahan)

Selain autentikasi sederhana (`simplePrompt`), library ini juga mendukung pembuatan kunci kriptografi.

* `createKeys()`: Membuat pasangan kunci publik/privat yang disimpan aman di *Hardware Security Module* HP.
* `deleteKeys()`: Menghapus kunci tersebut.
    *(Untuk materi kali ini, kita fokus pada `simplePrompt` untuk autentikasi lokal).*

-----

### C. Kustomisasi Tampilan Android vs iOS

Perilaku UI sedikit berbeda antar platform:

* **iOS:** Tampilan Face ID/Touch ID sangat standar dari Apple. Kita hanya bisa mengubah `promptMessage`. Jika gagal, iOS otomatis menawarkan opsi passcode jika diizinkan sistem.
* **Android:** Tampilan menggunakan *BiometricPrompt* standar Android.
  * Kita **wajib** mengisi `cancelButtonText`.
  * Kita **bisa** mengkustomisasi pesan agar user paham konteksnya (misal: "Scan jari untuk transfer Rp 50.000").

### D. Integrasi dengan Keychain

Penting dipahami: **Biometrik tidak menyimpan password**.
Untuk membuat fitur "Login Otomatis", kita butuh bantuan `react-native-keychain`.

* **Flow:**
    1. User Login Manual -\> Simpan Token di **Keychain**.
    2. User Buka App -\> App minta **Biometrik**.
    3. Biometrik Sukses -\> App buka **Keychain** -\> Ambil Token -\> Masuk App.

-----

## 3\. Contoh Implementasi Kode

Berikut adalah contoh kode dengan library `@sbaiahmed1/react-native-biometrics`.

### A. Contoh Cek Sensor & Tipe Biometrik

Kode ini mendeteksi apakah HP user mendukung Face ID, Touch ID, atau Biometrik Android.

```jsx
import React, { useEffect, useState } from 'react';
import { View, Text, ActivityIndicator } from 'react-native';
import ReactNativeBiometrics, { FaceID } from '@sbaiahmed1/react-native-biometrics';

const CheckSensor = () => {
  const [status, setStatus] = useState('Mengecek Sensor...');
  
  useEffect(() => {
    const rnBiometrics = new ReactNativeBiometrics();

    const check = async () => {
      const { available, biometryType } = await rnBiometrics.isSensorAvailable();

      if (available) {
        if (biometryType === FaceID) {
          setStatus('Face ID Tersedia');
        } else {
          setStatus('Sensor Biometrik Tersedia');
        }
      } else {
        setStatus('Biometrik Tidak Tersedia');
      }
    };
    check();
  }, []);

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text style={{ fontSize: 18 }}>{status}</Text>
    </View>
  );
};

export default CheckSensor;
```

### B. Contoh Prompt Autentikasi

Kode ini memicu dialog biometrik. Perhatikan penggunaan `promptMessage` dan `cancelButtonText` yang wajib untuk kompatibilitas Android.

```jsx
import React from 'react';
import { View, Button, Alert } from 'react-native';
import ReactNativeBiometrics from '@sbaiahmed1/react-native-biometrics';

const SimpleAuth = () => {
  const handleAuth = async () => {
    const rnBiometrics = new ReactNativeBiometrics();

    try {
      const { success } = await rnBiometrics.simplePrompt({
        promptMessage: 'Konfirmasi Identitas', // Muncul di popup
        cancelButtonText: 'Batalkan',          // Wajib untuk Android
      });

      if (success) {
        Alert.alert('Berhasil', 'Identitas Terverifikasi!');
      } else {
        console.log('User membatalkan atau verifikasi gagal');
      }
    } catch (error) {
      Alert.alert('Error', 'Terjadi kesalahan pada sensor');
    }
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Button title="Tes Biometrik" onPress={handleAuth} />
    </View>
  );
};

export default SimpleAuth;
```

### C. Contoh Integrasi dengan Keychain

Ini adalah implementasi paling umum di aplikasi profesional: Login sekali, selanjutnya pakai sidik jari.

*(Pastikan sudah install: `npm install react-native-keychain`)*

```jsx
import React, { useState } from 'react';
import { View, Button, Alert, TextInput, Text } from 'react-native';
import ReactNativeBiometrics from '@sbaiahmed1/react-native-biometrics';
import * as Keychain from 'react-native-keychain';

const SecureLogin = () => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  
  // 1. Login Manual & Simpan Kredensial
  const handleLoginManual = async () => {
    // Simulasi request ke server...
    if (username === 'user' && password === '1234') {
      // Simpan ke Keychain (Brankas)
      await Keychain.setGenericPassword(username, password);
      Alert.alert('Sukses', 'Login berhasil & disimpan untuk Biometrik');
    } else {
      Alert.alert('Gagal', 'Username/Password salah');
    }
  };

  // 2. Login via Biometrik
  const handleLoginBiometric = async () => {
    const rnBiometrics = new ReactNativeBiometrics();

    // Cek ketersediaan dulu
    const { available } = await rnBiometrics.isSensorAvailable();
    if (!available) {
      Alert.alert('Maaf', 'Sensor tidak tersedia');
      return;
    }

    // Tampilkan Prompt
    const { success } = await rnBiometrics.simplePrompt({
      promptMessage: 'Login Cepat',
      cancelButtonText: 'Gunakan Password',
    });

    if (success) {
      // Jika sidik jari cocok, ambil password dari Keychain
      const credentials = await Keychain.getGenericPassword();
      if (credentials) {
        Alert.alert('Welcome Back!', `Halo ${credentials.username}, Anda berhasil login.`);
      } else {
        Alert.alert('Info', 'Tidak ada data tersimpan. Login manual dulu.');
      }
    }
  };

  return (
    <View style={{ padding: 20, flex: 1, justifyContent: 'center' }}>
      <Text>Username: user, Pass: 1234</Text>
      <TextInput 
        placeholder="Username" value={username} onChangeText={setUsername} 
        style={{ borderWidth: 1, marginBottom: 10, padding: 8 }}
      />
      <TextInput 
        placeholder="Password" value={password} onChangeText={setPassword} secureTextEntry 
        style={{ borderWidth: 1, marginBottom: 10, padding: 8 }}
      />
      
      <Button title="Login Manual" onPress={handleLoginManual} />
      <View style={{ height: 20 }} />
      <Button title="Login dengan Biometrik" onPress={handleLoginBiometric} color="green" />
    </View>
  );
};

export default SecureLogin;
```

-----

## 4\. Rangkuman

Di Hari ke-20 ini, kita telah mempelajari:

1. Library **`@sbaiahmed1/react-native-biometrics`** lebih andal untuk proyek modern.
2. **`simplePrompt`** adalah metode inti untuk verifikasi, namun di Android, Anda wajib menyertakan **`cancelButtonText`** agar tidak error.
3. Kombinasi **Biometrik** (sebagai verifikator) dan **Keychain** (sebagai penyimpanan) adalah standar industri untuk fitur "Login Cepat" yang aman.

-----

## 5\. Referensi

* [npm: @sbaiahmed1/react-native-biometrics](https://www.google.com/search?q=https://www.npmjs.com/package/%40sbaiahmed1/react-native-biometrics)
* [npm: react-native-keychain](https://www.npmjs.com/package/react-native-keychain)
* [Android Developer: BiometricPrompt](https://developer.android.com/training/sign-in/biometric-auth)
* [Apple Developer: LocalAuthentication](https://developer.apple.com/documentation/localauthentication)

-----

## 6\. Evaluasi Harian: Soal Praktik

`Proyek Lanjutan: Mini E-Commerce`

### 1\. Integrasi Hybrid (Biometrik + Keystore) untuk Login

**Tugas:**
Implementasikan fitur login di mana token pengguna disimpan secara aman setelah login manual pertama kali.

* Gunakan `Keychain.setGenericPassword` saat login manual sukses.
* Buat tombol "Login Cepat" yang memicu `rnBiometrics.simplePrompt`.
* Jika sukses, ambil token dengan `Keychain.getGenericPassword` dan arahkan user ke halaman utama.

### 2\. Konfirmasi Transaksi dengan Konteks (UX Android)

**Skenario:** User akan mentransfer uang sebesar Rp 500.000.
**Tugas:**

* Panggil `simplePrompt` dengan `promptMessage`: "Konfirmasi Transfer Rp 500.000".
* Atur `cancelButtonText` menjadi "Batalkan Transaksi".
* **Logic:** Jika hasil `success` adalah `true`, jalankan fungsi `processPayment()`. Jika gagal/batal, tampilkan Alert "Transaksi Dibatalkan".

### 3\. Penanganan Kasus "Not Enrolled" (Belum Daftar)

**Skenario:** HP user memiliki sensor sidik jari, tetapi user belum pernah mendaftarkan jarinya di pengaturan HP.
**Tugas:**

* Panggil `isSensorAvailable()`.
* Analisis hasilnya. Jika `available` bernilai `false` namun error mengindikasikan "Not Enrolled" (atau cek dokumentasi error code spesifik library), jangan panggil `simplePrompt`.
* Gantinya, tampilkan Alert: "Sidik jari belum diatur di HP ini. Silakan atur di Settings" dan arahkan ke input PIN manual.

### 4\. Keamanan: Force Logout saat Lockout

**Skenario:** Seseorang mencoba memaksa masuk menggunakan biometrik orang lain berkali-kali hingga sensor terkunci (*Lockout*).
**Tugas:**

* Simulasikan atau tangani kondisi *error* saat pemanggilan `simplePrompt`.
* Jika terdeteksi error yang bersifat fatal atau lockout, lakukan tindakan preventif:
    1. Hapus data sensitif di Keychain (`resetGenericPassword`).
    2. Navigasi paksa ke halaman Login awal.
        *Jelaskan dalam komentar kode Anda mengapa langkah ini penting.*

### 5\. Tampilan Dinamis (Face ID vs Touch ID)

**Skenario:** Buat aplikasi terasa lebih personal dan pintar.
**Tugas:**

* Gunakan `isSensorAvailable()` saat komponen dimuat (`useEffect`).
* Simpan tipe biometrik ke dalam state (`biometryType`).
* Saat user menekan tombol login, sesuaikan pesan prompt:
  * Jika tipe adalah `FaceID`, isi `promptMessage` dengan "Pindai Wajah untuk Masuk".
  * Jika tipe adalah `TouchID`/`Biometrics`, isi dengan "Tempelkan Jari untuk Masuk".

-----

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

-----

**Mobile App Development With React Native*  
