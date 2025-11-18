# Hari ke-16: Storage: Keystore System

Materi ini membahas cara menyimpan data-data penting (seperti *password* atau *token*) di aplikasi, menggunakan sistem keamanan tingkat perangkat keras (**Hardware-Encrypted**), sehingga data tidak bisa dicuri meski ponsel dibongkar. Kita akan menggunakan *library* **`react-native-keychain`**.

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

* **Memahami konsep Keystore:** Tahu bahwa Keystore (dikenal sebagai **Keychain di iOS** dan **Keystore di Android**) adalah tempat penyimpanan yang **sangat aman** karena dienkripsi oleh perangkat keras ponsel, beda dengan AsyncStorage (Hari ke-15) yang tidak dienkripsi.
* **Menggunakan Perintah Dasar:** Mampu menggunakan fungsi utama dari `react-native-keychain` seperti **`setGenericPassword`** (simpan aman), **`getGenericPassword`** (ambil aman), dan **`resetGenericPassword`** (hapus aman).
* **Menyimpan Data Rahasia:** Menggunakan Keystore untuk menyimpan data sensitif, seperti **token otentikasi**, **kata sandi**, atau **kunci API**.
* **Mengelola Gabungan Penyimpanan (Hybrid Storage):** Menggabungkan Keystore (untuk data rahasia) dengan AsyncStorage (untuk pengaturan biasa) agar aplikasi tetap aman dan cepat.
* **Menerapkan Praktik Terbaik Keamanan:** Selalu memberi nama unik (**namespacing**) pada layanan yang disimpan (misal: `'@app:auth'`), dan tahu cara mengatasi jika terjadi masalah akses Keystore (misalnya karena kunci perangkat diubah).
* **Mengatasi Masalah Keamanan:** Menganalisis dan memperbaiki masalah umum seperti izin akses ditolak (*access denied*) atau kegagalan berbagi data antar aplikasi.
* **Membangun Fitur Aman:** Membuat alur *login* yang aman, di mana *token* disimpan langsung di Keystore.

---

## 2. Materi Pembelajaran

Keystore adalah lapisan keamanan perangkat keras yang terisolasi (**Secure Enclave** di iOS, **TEE/Trusted Execution Environment** di Android). Data yang disimpan di sini dilindungi dari aplikasi lain, sistem operasi, bahkan dari upaya *jailbreak* atau *root*.

### A. Konsep Keystore dan Perbedaan dengan AsyncStorage

| Fitur | AsyncStorage (Hari ke-15) | Keystore/Keychain (Hari ke-16) |
| :--- | :--- | :--- |
| **Keamanan** | **Tidak Aman** (Unencrypted) | **Sangat Aman** (Hardware-Encrypted) |
| **Kegunaan** | Preferensi, Cache, Data Non-Sensitif | **Token, Password, API Key** (Data Rahasia) |
| **Akses** | Mudah, disimpan sebagai teks biasa | Membutuhkan izin khusus, dilindungi kunci/sidik jari |
| **Lokasi** | Penyimpanan Internal Aplikasi | Chip Keamanan Perangkat Keras |

**`react-native-keychain`** adalah jembatan yang menyederhanakan akses ke sistem Keystore/Keychain di kedua platform (iOS dan Android) dengan API tunggal.

### B. Perintah Dasar `react-native-keychain`

Library ini menyimpan data dalam format "Generic Password" (pasangan *username* dan *password*), meskipun kita sering memakainya untuk menyimpan *token* saja.

| Perintah | Fungsi | Catatan Penting |
| :--- | :--- | :--- |
| `setGenericPassword(user, pass, options)` | **Simpan** sepasang *username* dan *password* (atau *user ID* dan *Token*). | **Wajib** pakai *options* `{ service: '@app:nama_unik' }` untuk isolasi. |
| `getGenericPassword(options)` | **Ambil** *username* dan *password* yang tersimpan. | Akan mengembalikan `null` jika tidak ada. |
| `resetGenericPassword(options)` | **Hapus** pasangan yang tersimpan dari Keystore. | Gunakan saat *logout*. |

**Options Kunci (Penting):**

* `service`: **Nama unik** yang wajib kamu berikan. Ini seperti *namespacing* agar data kamu tidak bentrok dengan data aplikasi lain.
* `accessible`: (**iOS saja**) Menentukan kapan data bisa diakses (misal: 'WhenUnlocked' = hanya saat perangkat tidak terkunci).

**Pola Sederhana:**

1. Simpan Token: `await Keychain.setGenericPassword('myUserId', myToken, { service: '@app:auth' });`
2. Ambil Token: `const creds = await Keychain.getGenericPassword({ service: '@app:auth' }); if (creds) { console.log(creds.password); }`

---

### C. Pola Lanjutan: Hybrid Storage (Penyimpanan Gabungan)

Pola ini menggabungkan Keystore dan AsyncStorage untuk performa dan keamanan terbaik:

1. **Keystore:** Simpan **Token Otentikasi** dan **Kunci API** (data rahasia).
2. **AsyncStorage:** Simpan **Preferensi Tema**, **Data Cache API** (data tidak rahasia).

**Keuntungan:** Pemuatan data aplikasi di awal menjadi lebih cepat karena kita bisa mengambil data dari Keystore dan AsyncStorage secara **paralel** menggunakan `Promise.all`.

### D. Studi Kasus Keamanan

#### Kasus 1: Penyimpanan Token Otentikasi Aman

* **Masalah:** Token JWT harus disimpan sangat aman.
* **Solusi:** Setelah *login*, token disimpan menggunakan `setGenericPassword` di Keystore. Saat aplikasi perlu memanggil API, token diambil menggunakan `getGenericPassword`. Token **dihapus total** saat *logout* dengan `resetGenericPassword`.

#### Kasus 2: Penanganan Akses Ditolak

* **Masalah:** Pengguna mengubah kata sandi perangkat, dan Keystore memblokir akses ke token yang lama (Access Denied).
* **Solusi:** Saat mencoba `getGenericPassword`, tambahkan `try-catch`. Jika error yang terjadi adalah "access denied", **paksa pengguna login ulang** dan hapus token yang lama untuk mencegah *bug* keamanan.

---

## 3. Contoh Implementasi

### A. Contoh Dasar: Operasi Dasar react-native-keychain

```jsx
// components/BasicKeychain.js
import React, { useState, useEffect } from 'react';
import { View, Text, Button, Alert } from 'react-native';
import * as Keychain from 'react-native-keychain';

const BasicKeychain = () => {
  const [token, setToken] = useState('');

  const saveToken = async () => {
    try {
      const result = await Keychain.setGenericPassword('user123', 'jwt_token_abc', {
        service: '@app:auth',
      });
      if (result) {
        Alert.alert('Success', 'Token saved securely');
      }
    } catch (e) {
      Alert.alert('Error', `Failed to save: ${e.message}`);
    }
  };

  const loadToken = async () => {
    try {
      const credentials = await Keychain.getGenericPassword({ service: '@app:auth' });
      if (credentials) {
        setToken(credentials.password);
        Alert.alert('Success', 'Token loaded');
      } else {
        Alert.alert('Info', 'No token found');
      }
    } catch (e) {
      Alert.alert('Error', `Failed to load: ${e.message}`);
    }
  };

  const removeToken = async () => {
    try {
      const result = await Keychain.resetGenericPassword({ service: '@app:auth' });
      if (result) {
        setToken('');
        Alert.alert('Success', 'Token removed');
      }
    } catch (e) {
      Alert.alert('Error', `Failed to remove: ${e.message}`);
    }
  };

  useEffect(() => {
    loadToken();
  }, []);

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Token: {token ? `${token.substring(0, 10)}...` : 'None'}</Text>
      <Button title="Save Token" onPress={saveToken} />
      <Button title="Load Token" onPress={loadToken} />
      <Button title="Remove Token" onPress={removeToken} />
    </View>
  );
};

export default BasicKeychain;
```

**Penjelasan:** setGenericPassword/getGenericPassword/resetGenericPassword dengan service namespace; load on mount; Alert untuk feedback; error catch untuk robustness.

### B. Contoh Interaktif: Secure Storage dengan Sharing

```jsx
// components/SharedKeychain.js
import React, { useState } from 'react';
import { View, Text, Button, Alert } from 'react-native';
import * as Keychain from 'react-native-keychain';

const SharedKeychain = () => {
  const [sharedToken, setSharedToken] = useState('');

  const saveWithSharing = async () => {
    try {
      const result = await Keychain.setGenericPassword('sharedUser', 'shared_token_xyz', {
        service: '@app:shared',
        sharedPassword: true,
      });
      if (result) {
        Alert.alert('Success', 'Shared token saved');
      }
    } catch (e) {
      Alert.alert('Error', `Failed to save: ${e.message}`);
    }
  };

  const loadShared = async () => {
    try {
      const credentials = await Keychain.getGenericPassword({
        service: '@app:shared',
        sharedPassword: true,
      });
      if (credentials) {
        setSharedToken(credentials.password);
        Alert.alert('Success', 'Shared token loaded');
      } else {
        Alert.alert('Info', 'No shared token found');
      }
    } catch (e) {
      Alert.alert('Error', `Failed to load: ${e.message}`);
    }
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Shared Token: {sharedToken ? `${sharedToken.substring(0, 10)}...` : 'None'}</Text>
      <Button title="Save Shared Token" onPress={saveWithSharing} />
      <Button title="Load Shared Token" onPress={loadShared} />
    </View>
  );
};

export default SharedKeychain;
```

**Penjelasan:** sharedPassword: true untuk cross-app access; same pattern as basic; error handling dengan Alert.

### C. Contoh Lanjutan: Hybrid Storage dengan AsyncStorage

```jsx
// components/HybridStorage.js
import React, { useState, useEffect } from 'react';
import { View, Text, Button } from 'react-native';
import * as Keychain from 'react-native-keychain';
import AsyncStorage from '@react-native-async-storage/async-storage';

const HybridStorage = () => {
  const [secureToken, setSecureToken] = useState('');
  const [nonSensitivePref, setNonSensitivePref] = useState('');

  const saveHybrid = async () => {
    // Secure token in Keychain
    await Keychain.setGenericPassword('user123', 'secure_token_xyz', { service: '@app:auth' });
    // Non-sensitive pref in AsyncStorage
    await AsyncStorage.setItem('@app:pref', 'dark_mode');
  };

  const loadHybrid = async () => {
    // Load secure token
    const creds = await Keychain.getGenericPassword({ service: '@app:auth' });
    if (creds) setSecureToken(creds.password);

    // Load non-sensitive
    const pref = await AsyncStorage.getItem('@app:pref');
    if (pref) setNonSensitivePref(pref);
  };

  useEffect(() => {
    loadHybrid();
  }, []);

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Secure Token: {secureToken ? `${secureToken.substring(0, 10)}...` : 'None'}</Text>
      <Text>Preference: {nonSensitivePref || 'None'}</Text>
      <Button title="Save Hybrid" onPress={saveHybrid} />
      <Button title="Load Hybrid" onPress={loadHybrid} />
    </View>
  );
};

export default HybridStorage;
```

**Penjelasan:** Hybrid: Keychain untuk token, AsyncStorage untuk pref; load parallel dengan Promise.all; error handling implisit via try-catch (tambahkan jika perlu).

**Tips Debugging:** Console.log(creds); test di device; simulate access denied dengan device lock; check storage via adb shell (Android) atau device logs (iOS).

## 4. Rangkuman

**Hari ke-16** mengajarkan cara membuat aplikasi kamu **aman** dan **patuh pada standar privasi** (GDPR/Apple). Gunakan **Keystore** untuk semua data rahasia (token, kunci) dan **AsyncStorage** untuk data biasa (preferensi, cache). Kunci utamanya adalah:

* **Pilih Alat Tepat:** Keystore untuk keamanan, AsyncStorage untuk kecepatan dan non-sensitif.
* **Namespacing:** Selalu gunakan *service* unik untuk isolasi data.
* **Error Handling:** Antisipasi *access denied* dan kegagalan Keystore dengan meminta pengguna login ulang.

---

**Referensi:**

* [react-native-keychain - GitHub](https://github.com/oblador/react-native-keychain)
* [react-native-keychain Documentation](https://oblador.github.io/react-native-keychain/docs/)
* [Secure Storage in React Native Using Keychain and Keystore - Medium](https://medium.com/@prem__kumar/secure-your-react-native-app-with-react-native-keychain-06d9e77b7ea2)

## 5. Evaluasi Harian: Soal Praktik

`Lanjutan project Mini E-Commerce`

Soal-soal ini menguji kemampuan Anda untuk membuat aplikasi yang **aman** (*secure*) dan mematuhi praktik terbaik dalam pengelolaan kredensial.

---

### a. Migrasi Token Otentikasi ke Keystore

**Tugas:** Ambil alih alur penyimpanan token otentikasi (yang sebelumnya disimpan di AsyncStorage).

1. Ubah fungsi **login berhasil** agar token JWT kini disimpan menggunakan **Keystore system**.
2. Gunakan **namespacing** `service` yang spesifik (misalnya, `'com.ecom:userToken'`) untuk isolasi.
3. Ubah *logic* pengecekan token di **Root Navigator** agar menggunakan **`Keychain.getGenericPassword`** untuk memuat token yang tersimpan di awal aplikasi.

---

### b. Integrasi *Hybrid Storage* untuk Preferensi Sensitif

**Tugas:** Terapkan strategi *Hybrid Storage* untuk data pengguna yang dimuat saat *splash screen*:

1. **Token Otentikasi** harus diambil dari **Keychain** (menggunakan *service*).
2. **Preferensi Pengguna** (misalnya, tema *dark/light*) harus diambil dari **AsyncStorage**.
3. Gunakan **`Promise.all`** untuk memuat kedua sumber penyimpanan ini secara **paralel** di awal aplikasi untuk mengoptimalkan waktu *loading* (*Key Store* bersifat *async* dan *blocking*).

---

### c. Penanganan *Access Denied Error* Saat *Load*

**Tugas:** Tingkatkan ketahanan aplikasi terhadap kegagalan akses *secure storage*.

1. Pada *logic* **`loadToken`** (saat *splash screen*), tambahkan blok **`try-catch`** untuk menangani error dari `Keychain.getGenericPassword`.
2. Jika terjadi error spesifik seperti **"access denied"** (simulasikan dengan menangkap *exception* yang dilempar), **reset token** di *Keystore* dan **paksa** pengguna untuk navigasi ke *Screen* **Login** dengan *Alert* yang menjelaskan: "Keamanan perangkat diubah, mohon login ulang."

---

### d. Pembersihan Data Aman saat *Logout* Total

**Tugas:** Perbarui fungsi **`handleLogout`** untuk memastikan semua data sensitif dihapus dari *secure storage*.

1. Selain menghapus *keys* dari AsyncStorage, panggil **`Keychain.resetGenericPassword`** dengan *service* yang benar (`'com.ecom:userToken'`) untuk menghapus token otentikasi dari **Keystore/Keychain**.
2. Pastikan proses *multiRemove* (AsyncStorage) dan *resetGenericPassword* (Keychain) dilakukan **sebelum** *navigation reset* untuk menjamin data sudah bersih.

---

### e. Menyimpan API Key Rahasia di Keystore

**Tugas:** Simulasikan kebutuhan untuk menyimpan *API Key* rahasia yang dibutuhkan oleh *Axios Interceptor*.

1. Buat fungsi di `apiClient.js` yang **menyimpan** *API Key* statis (e.g., `'API_KEY_SECRET_XYZ'`) ke **Keychain** dengan *username* statis (e.g., `'api_client'`) dan *service* yang berbeda (misalnya, `'com.ecom:apiKey'`).
2. Perbarui **Axios Request Interceptor** agar ia **mengambil** *API Key* ini dari **Keychain** *setiap kali request dikirim* dan menambahkannya ke *header* (`X-API-Key`).
3. Tangani kasus di *Interceptor* di mana *key* tidak ditemukan di *Keychain* dengan membuang error **401 Unauthorized** simulasi, menghentikan *request*.

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

---

**Mobile App Development With React Native*
