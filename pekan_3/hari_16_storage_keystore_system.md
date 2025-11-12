# Hari ke-16 - Storage: Keystore System

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini,siswa diharapkan mampu:

- Memahami Keystore system sebagai penyimpanan aman hardware-encrypted (Keychain iOS, Keystore Android), serta mengimplementasikannya menggunakan react-native-keychain untuk operasi dasar seperti setGenericPassword, getGenericPassword, dan resetGenericPassword, memastikan kredensial aman dari reverse engineering atau app compromise.
- Mengimplementasikan pola penggunaan react-native-keychain untuk menyimpan data sensitif seperti auth tokens, passwords, atau API keys, dengan pengelolaan options seperti service namespacing dan secure sharing antar apps.
- Mengelola fitur lanjutan seperti integration dengan AsyncStorage dari Hari ke-15 untuk hybrid storage (secure untuk tokens, unsecure untuk prefs), sambil menangani errors seperti access denied.
- Menerapkan praktik terbaik seperti namespacing services untuk isolation, secure enclave usage di iOS untuk tamper-proof storage, dan fallback mechanisms untuk devices dengan constraints, memastikan aplikasi yang compliant dengan GDPR/Apple privacy guidelines.
- Menganalisis dan menyelesaikan troubleshooting umum seperti keychain access denied atau cross-app sharing failures, dengan langkah-langkah debugging menggunakan console dan device simulators.
- Menerapkan aksesibilitas dan security audits (e.g., avoid logging sensitive data), untuk storage yang scalable dan secure di RN 0.75+.
- Membangun prototipe dengan Keystore lengkap, seperti secure login flow dengan token storage, siap untuk integrasi dengan networking (Hari ke-13) atau auth flows.

## 2. Materi Pembelajaran

Bagian ini menjelaskan Keystore system secara mendalam, termasuk API react-native-keychain, pola penggunaan, dan pertimbangan security. Keystore menyimpan data di secure hardware (Secure Enclave iOS, Trusted Execution Environment Android), terlindung dari app code dan OS access. react-native-keychain adalah wrapper cross-platform yang menyederhanakan akses, dengan dukungan secure sharing antar apps. Per 2025, best practices menekankan least privilege (store minimal data) dan regular key rotation. Integrasikan dengan error handling (Hari ke-14) untuk robustness.

### A. Pengenalan Keystore System: Konsep dan Setup react-native-keychain

**Tujuan:** Keystore adalah secure storage untuk sensitive data, membedakan dari AsyncStorage (Hari ke-15) yang unencrypted; pahami untuk memilih use case yang tepat (secure: tokens; unsecure: prefs).

**Konsep Kunci:**

- **iOS Keychain:** Encrypted key-value store di Secure Enclave; access via entitlements; supports sharing via app groups.
- **Android Keystore:** Hardware-backed (TEE); supports key aliases; access control via permissions.
- **react-native-keychain:** Unified API; stores as generic passwords (username/password pairs); options for service isolation and sharing.
- **Security Benefits:** Data aman dari app uninstall/reinstall (persistent); anti-tampering; protected from root/jailbreak (to extent hardware allows).

**Setup Dasar:** Instal `npm i react-native-keychain`; auto-link RN 0.60+; iOS: Tambah `keychain-access-groups` di Info.plist jika sharing; Android: Permissions di manifest. Import: `import * as Keychain from 'react-native-keychain';`.

**Pola Penggunaan:** Basic: `Keychain.setGenericPassword('username', 'password', { service: '@app:auth' });`. Dengan sharing: `{ sharedPassword: true }` untuk cross-app.

**Praktik Terbaik (2025):** Gunakan service namespacing untuk isolation; minimize stored data (tokens only); fallback to AsyncStorage jika Keystore unavailable (e.g., emulator). RN 0.75+: Better integration dengan JSI untuk faster async ops.

**Pertimbangan Platform:** iOS: Requires provisioning profile untuk enclave; Android: API 23+ untuk advanced features; test di physical devices (emulators lack secure hardware).

### B. API react-native-keychain: Methods dan Options

**Tujuan:** API menyediakan methods sederhana untuk secure ops; pahami untuk implementasi aman.

**Methods Kunci (v2.0+, 2025):**

| Method | Tipe | Deskripsi | Return | Catatan & Best Practices |
|--------|------|-----------|--------|--------------------------|
| `setGenericPassword(username, password, options)` | async (string, string, object) => Promise<boolean> | Simpan pair username/password. | true/false (success) | Options: { service: '@app:auth' }; username for token ID, password for secret. |
| `getGenericPassword(options)` | async (object) => Promise<{ username, password }|null> | Ambil pair; null jika tidak ada. | Object/null | Options: { service: '@app:auth' }; handle null dengan login flow. |
| `resetGenericPassword(options)` | async (object) => Promise<boolean> | Hapus pair. | true/false | Options: { service: '@app:auth' }; gunakan di logout; confirm dengan user dialog. |
| `setInternetCredentials(hostname, username, password, options)` | async (string, string, string, object) => Promise<boolean> | Simpan web creds (legacy). | true/false | Deprecated; gunakan generic untuk modern; options sama dengan setGenericPassword. |
| `getInternetCredentials(hostname, options)` | async (string, object) => Promise<{ username, password }|null> | Ambil web creds. | Object/null | Serupa generic; migrasi ke generic untuk simplicity. |

**Options Kunci (Shared):**

| Option | Tipe | Deskripsi | Default | Catatan Platform |
|--------|------|-----------|---------|------------------|
| `service` | string | Namespace unik (e.g., bundle ID). | App bundle | Wajib untuk isolation; '@com.myapp:auth' untuk avoid conflicts. |
| `accessible` | 'AfterFirstUnlock'/'AfterFirstUnlockThisDeviceOnly'/'Always'/'WhenUnlocked'/'WhenUnlockedThisDeviceOnly' | Timing access. | 'Any' | iOS only; 'AfterFirstUnlock' for post-boot access; 'Always' for background. |
| `sharedPassword` | boolean | Enable sharing antar apps. | false | iOS: App groups; Android: Keystore sharing; gunakan untuk SSO. |

**Pola Penggunaan:** Secure token: `await setGenericPassword('userId', token, { service: '@app:auth' }); const creds = await getGenericPassword({ service: '@app:auth' }); if (creds) useToken(creds.password);`. Pola logout: `await resetGenericPassword({ service: '@app:auth' });`.

**Praktik Terbaik (2025):** Selalu gunakan service untuk namespace; test di device (emulator limited); fallback to AsyncStorage jika Keystore unavailable; avoid storing large data (use AsyncStorage for non-sensitive). Integrasi error (Hari ke-14): Try-catch di async calls; handle false returns dengan user prompts. RN 0.75+: Better support untuk Keychain APIs di iOS 18+ dengan async queries.

**Pertimbangan Platform:** iOS: Keychain sharing via groups (add to entitlements); Android: Keystore aliases untuk key rotation; keduanya support secure notes untuk metadata.

### C. Pola Penggunaan Lanjutan: Secure Sharing dan Hybrid Storage

**Tujuan:** Integrasi secure sharing untuk SSO; hybrid dengan AsyncStorage untuk tiered storage (secure untuk tokens, unsecure untuk prefs).

**Pola Secure Sharing:**

- Set: `{ sharedPassword: true, service: '@app:shared' }`; get dari app lain.
- Pola: On login, set shared token; other apps get via same service.

**Hybrid Storage:** Sensitive (tokens) di Keychain; non-sensitive (theme) di AsyncStorage; sync via event listeners (e.g., on login, set both).

**Praktik Terbaik:** Minimize stored data; rotate tokens periodically; test in incognito mode (clear storage). 2025: Support untuk post-quantum keys di Android 15+ via library options.

### D. Studi Kasus Keystore dengan Langkah Penyelesaian

**Tujuan:** Studi kasus menunjukkan aplikasi praktis, dengan langkah implementasi dan debugging.

**Studi Kasus 1: Secure Auth Token Storage**

- **Deskripsi:** Simpan JWT token post-login; akses aman untuk high security.
- **Langkah Implementasi:**
  1. **Setup:** On login: `await setGenericPassword('userId', token, { service: '@app:auth' });`.
  2. **Pola:** useEffect: `const creds = await getGenericPassword({ service: '@app:auth' }); if (creds) { setToken(creds.password); }`; on logout: `await resetGenericPassword({ service: '@app:auth' });`.
  3. **Debugging:** Console.log(creds ? 'Access granted' : 'No token'); test dengan device restart; fallback: if (error) show login prompt.
  4. **Outcome:** Token aman; easy access; prevent leaks on uninstall.

**Studi Kasus 2: Hybrid Storage untuk User Profile**

- **Deskripsi:** Sensitive (token) di Keychain; non-sensitive (avatar URL) di AsyncStorage; sync on login.
- **Langkah Implementasi:**
  1. **Setup:** Login: `await setGenericPassword('userId', token, { service: '@app:auth' }); await AsyncStorage.setItem('@app:profile', JSON.stringify({ avatar: 'url' }));`.
  2. **Pola:** Load: `const [creds, profile] = await Promise.all([getGenericPassword({ service: '@app:auth' }), AsyncStorage.getItem('@app:profile')]);`.
  3. **Debugging:** Log 'Keychain access: success/fail'; test corruption dengan manual edit storage (adb shell); ensure sync on error: if Keychain fail, fallback to AsyncStorage token (less secure).
  4. **Outcome:** Tiered security; fast load dengan parallel ops; resilient to partial failures.

**Studi Kasus 3: Secure Key Rotation on Logout**

- **Deskripsi:** Rotate token on logout; clear both Keychain dan AsyncStorage; handle access denied errors.
- **Langkah Implementasi:**
  1. **Setup:** Logout: `try { await resetGenericPassword({ service: '@app:auth' }); await AsyncStorage.removeItem('@app:token'); } catch (e) { if (e.message.includes('access denied')) promptUserReauth(); }`.
  2. **Pola:** Listener on logout event; verify clear dengan getGenericPassword (expect null).
  3. **Debugging:** Console.error(e.message) untuk categorize; test denied access dengan device lock; manual clear: adb shell untuk Android storage wipe.
  4. **Outcome:** Clean slate on logout; error-specific recovery; prevent stale data leaks.

**Praktik Terbaik untuk Studi Kasus (2025):** Test di physical devices; audit keys dengan custom getAllKeys (via service filter); comply with privacy (minimize data, consent prompts).

**Pertimbangan Platform:** iOS: Keychain queries via SecItemCopyMatching for advanced; Android: Keystore aliases untuk rotation; keduanya support secure notes untuk metadata.

**Integrasi Keseluruhan:** Keystore + AsyncStorage: Secure tokens di Keychain, prefs di AsyncStorage; load hybrid on app start; error handling untuk access denials.

## 3. Contoh Implementasi

Kode siap di React Native CLI. Import: `import * as Keychain from 'react-native-keychain';`.

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

Hari ke-16 membangun secure storage melalui Keystore system dengan react-native-keychain: API dasar (set/get/resetGenericPassword), pola secure sharing (sharedPassword), dan hybrid dengan AsyncStorage untuk tiered persistence. Kunci: Service namespasing untuk isolation; secure options seperti accessible untuk timing; error handling untuk access denials. Integrasi ini ciptakan apps aman yang lindungi sensitive data, resilient to device changes, dan compliant dengan privacy standards, siap untuk advanced features seperti key rotation.

**Latihan Selanjutnya:** Implementasikan secure login dengan token storage; hybrid cache tokens dari networking Hari ke-13.

**Referensi:**

- [react-native-keychain - GitHub](https://github.com/oblador/react-native-keychain)
- [react-native-keychain Documentation](https://oblador.github.io/react-native-keychain/docs/)
- [Secure Storage in React Native Using Keychain and Keystore - Medium](https://medium.com/@prem__kumar/secure-your-react-native-app-with-react-native-keychain-06d9e77b7ea2)

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
