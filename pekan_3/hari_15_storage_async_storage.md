# Hari ke-15: Penyimpanan Data Lokal (**AsyncStorage**)

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

* **Memahami AsyncStorage:** Tahu bahwa AsyncStorage adalah tempat penyimpanan data sederhana (seperti kotak kunci-nilai) yang **lambat** (asynchronous), **tidak dienkripsi**, dan aman digunakan bersamaan dengan bagian lain aplikasi (thread-safe). Data yang disimpan akan **tetap ada** meski aplikasi dimatikan/dihidupkan lagi.
* **Menggunakan Perintah Dasar:** Mampu menggunakan fungsi utama seperti **`setItem`** (simpan), **`getItem`** (ambil), **`removeItem`** (hapus), dan fungsi `multi-` (untuk banyak data sekaligus).
* **Menerapkan Pola Penyimpanan:** Bisa menyimpan data kecil, seperti **pengaturan pengguna**, **token masuk**, atau status sementara, sambil mengelola jika terjadi kesalahan (error handling).
* **Mengatur Caching (Penyimpanan Sementara):** Menggunakan AsyncStorage untuk menyimpan data API sementara (caching) agar bisa diakses **saat offline** (cache-first), menentukan **batas waktu data** (TTL/Time-to-Live) agar data tetap baru, dan menjaga agar total data **tidak melebihi 6MB**.
* **Menerapkan Praktik Terbaik:** Menamai kunci penyimpanan dengan teratur (contoh: `'@app:settings'`), mengubah objek menjadi teks (JSON.stringify/parse) sebelum menyimpan, dan menggabungkannya dengan proses ambil data dari internet (networking).
* **Mengatasi Masalah:** Mampu menganalisis dan memperbaiki masalah umum seperti data rusak atau kelebihan kuota penyimpanan.
* **Membangun Prototipe:** Membuat fitur penyimpanan yang siap digabungkan dengan manajemen state aplikasi (seperti Context/Redux).

**Catatan:** AsyncStorage tetap penting dan kini berjalan lebih baik di lingkungan multi-thread (seperti RN 0.75+).

---

## 2. Materi Pembelajaran

**AsyncStorage** adalah alat sederhana untuk menyimpan data *key-value* di ponsel (NSUserDefaults di iOS, SharedPreferences di Android), tetapi **tidak ada enkripsi** secara *default*.

### A. Perintah Dasar AsyncStorage (Key-Value)

AsyncStorage menyediakan perintah sederhana untuk menyimpan data yang sifatnya **Asynchronous** (prosesnya tidak instan, harus ditunggu).

| Perintah | Fungsi | Catatan Penting |
| :--- | :--- | :--- |
| `setItem(key, value)` | **Simpan** satu data. | *Semua* data harus berupa teks. Ubah objek jadi teks pakai **`JSON.stringify`**. Batasi ukuran $<1\text{MB}$ per item. |
| `getItem(key)` | **Ambil** satu data. | Gunakan **`await`** untuk menunggu. Ubah teks jadi objek lagi pakai **`JSON.parse`**. |
| `removeItem(key)` | **Hapus** satu data. | Contoh untuk menghapus token saat *logout*. |
| `multiSet(pasangan key-value)` | **Simpan** banyak data sekaligus. | Lebih cepat daripada `setItem` satu per satu. |
| `multiGet(keys)` | **Ambil** banyak data sekaligus. | Lebih cepat untuk memuat status awal aplikasi (misal: pengaturan + token). |
| `clear()` | **Hapus SEMUA** data AsyncStorage. | Hanya untuk *logout* atau *debugging* ekstrem. |

**Pola Sederhana:**

1. Simpan: `await AsyncStorage.setItem('key', JSON.stringify({obj}));`
2. Ambil: `const obj = JSON.parse(await AsyncStorage.getItem('key')) || nilai_default;`

**Praktik Terbaik (Penting!):**

* **Namespacing:** Beri awalan pada kunci (contoh: `'@app:username'`) untuk menghindari bentrok nama.
* **Serialization:** Selalu ubah Objek menjadi Teks (JSON.stringify/parse).
* **Error Handling:** Gunakan **`try-catch`** untuk mengantisipasi jika penyimpanan penuh atau data rusak.

---

### B. Teknik Caching Data Lokal (Penyimpanan Sementara)

Caching adalah menyimpan data dari API untuk mengurangi panggilan ke server dan membuat aplikasi bisa dipakai **saat offline**.

| Teknik Caching | Deskripsi | Kapan Digunakan |
| :--- | :--- | :--- |
| **Cache-First (Offline)** | Cek data lokal dulu. Jika ada, pakai itu. Baru ambil dari API jika tidak ada atau *stale* (basi). | Data yang jarang berubah (profil pengguna, daftar kategori). |
| **TTL (Time-to-Live)** | Tambahkan **timestamp** (waktu simpan) pada data. Data akan "basi" dan harus di-fetch ulang jika sudah melewati batas waktu (TTL). | Data yang *agak* sering berubah (berita, feed). Contoh: TTL 30 menit. |
| **Conditional Invalidation** | Hapus *cache* secara sengaja saat ada kejadian tertentu (misal: pengguna *logout* atau ganti data). | Data spesifik pengguna. |
| **Batch Caching** | Gunakan `multiSet`/`multiGet` untuk menyimpan/mengambil data terkait secara berkelompok. | Efisien untuk memuat status aplikasi di awal. |

**Praktik Terbaik Caching:**

* Jaga agar total ukuran data **<6MB**.
* Gunakan TTL untuk data yang perlu *fresh*.
* Bersihkan data lama secara rutin (cleanup) agar tidak memenuhi memori.

---

### C. Studi Kasus

#### Kasus 1: Caching Data API untuk Offline Access

* **Masalah:** Ingin pengguna bisa melihat daftar produk meski internet mati (offline).
* **Solusi:** Terapkan **Cache-First** Strategy. Setelah ambil data produk dari API, **simpan** ke AsyncStorage. Saat aplikasi dibuka, **cek dulu** AsyncStorage. Jika ada, tampilkan.

#### Kasus 2: Menyimpan Pengaturan Pengguna dengan TTL

* **Masalah:** Simpan tema ('dark'/'light'). Ingin sesekali *sync* pengaturan dari server tanpa harus selalu *fetch* di setiap buka aplikasi.
* **Solusi:** Simpan pengaturan dengan **TTL 24 Jam**. Jika sudah lebih dari 24 jam, anggap *cache* kadaluarsa dan coba *fetch* ulang dari server (jika ada koneksi).

#### Kasus 3: Optimasi Startup Aplikasi (Batch Load)

* **Masalah:** Aplikasi lambat di awal karena harus mengambil 3 data penting (token, pengaturan, status terakhir) secara berurutan.
* **Solusi:** Gunakan **`AsyncStorage.multiGet(['token', 'prefs', 'status'])`** untuk mengambil ketiganya **secara bersamaan**, membuat waktu *loading* lebih cepat.

---

## 3. Contoh Implementasi

Kode dibawah ini menunjukkan cara sederhana untuk **Simpan**, **Ambil**, dan **Hapus** data username, serta contoh penggunaan **Hook Caching** dengan TTL 5 menit.

### A. Contoh Dasar: Operasi Dasar AsyncStorage

```jsx
// components/BasicStorage.js
import React, { useState, useEffect } from 'react';
import { View, Text, Button } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';

const BasicStorage = () => {
  const [username, setUsername] = useState('');

  const saveUser = async () => {
    try {
      await AsyncStorage.setItem('@app:username', 'John Doe');
      console.log('User saved');
    } catch (e) {
      console.error('Save error:', e);
    }
  };

  const loadUser = async () => {
    try {
      const user = await AsyncStorage.getItem('@app:username');
      if (user !== null) {
        setUsername(user);
      }
    } catch (e) {
      console.error('Load error:', e);
    }
  };

  const removeUser = async () => {
    try {
      await AsyncStorage.removeItem('@app:username');
      setUsername('');
    } catch (e) {
      console.error('Remove error:', e);
    }
  };

  useEffect(() => {
    loadUser();
  }, []);

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Username: {username}</Text>
      <Button title="Save User" onPress={saveUser} />
      <Button title="Load User" onPress={loadUser} />
      <Button title="Remove User" onPress={removeUser} />
    </View>
  );
};

export default BasicStorage;
```

**Penjelasan:** setItem/getItem/removeItem dengan try-catch; load on mount; UI update dengan state.

### B. Contoh Interaktif: Caching dengan TTL

```jsx
// hooks/useTTLCache.js
import { useState, useEffect } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

const TTL = 300000; // 5 minutes

export const useTTLCache = (key, fetchFn) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  const getFromCache = async () => {
    try {
      const cached = await AsyncStorage.getItem(key);
      if (cached) {
        const { value, timestamp } = JSON.parse(cached);
        if (Date.now() - timestamp < TTL) {
          setData(value);
          setLoading(false);
          return;
        }
      }
    } catch (e) {
      console.error('Cache read error:', e);
    }
  };

  const setToCache = async (value) => {
    try {
      await AsyncStorage.setItem(key, JSON.stringify({ value, timestamp: Date.now() }));
      setData(value);
    } catch (e) {
      console.error('Cache write error:', e);
    }
  };

  useEffect(() => {
    getFromCache();
  }, []);

  const refresh = async () => {
    setLoading(true);
    try {
      const freshData = await fetchFn();
      await setToCache(freshData);
      setLoading(false);
    } catch (e) {
      console.error('Fetch error:', e);
      setLoading(false);
    }
  };

  return [data, loading, refresh];
};
```

**Integrasi:**

```jsx
// components/CachedData.js
import React from 'react';
import { View, Text, Button } from 'react-native';
import { useTTLCache } from '../hooks/useTTLCache';

const CachedData = () => {
  const fetchUsers = () => fetch('https://jsonplaceholder.typicode.com/users').then(res => res.json());
  const [data, loading, refresh] = useTTLCache('@app:users', fetchUsers);

  if (loading) return <Text>Loading...</Text>;

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>{data ? data[0]?.name : 'No data'}</Text>
      <Button title="Refresh" onPress={refresh} />
    </View>
  );
};

export default CachedData;
```

**Penjelasan:** Hook check TTL sebelum fetch; refresh manual; fallback jika expired.


---

## 4. Rangkuman

**Hari ke-15** ini mengajari kita cara menggunakan **AsyncStorage** untuk membuat aplikasi yang **datanya persisten** (tidak hilang) dan **mampu bekerja offline** (caching). Kunci utama yang harus diingat:

* **Namespacing** dan **JSON** untuk data yang rapi.
* **TTL** untuk menjaga data tetap *fresh*.
* **`multi-methods`** untuk operasi yang lebih cepat.

Ini adalah fondasi kuat sebelum melangkah ke penyimpanan data yang lebih kompleks seperti basis data SQLite.

---

**Referensi:**

* [Storing Data Permanently in React Native using AsyncStorage 2025](https://medium.com/@mahesh.nikate/storing-data-permanently-in-react-native-using-asyncstorage-2025-91a79b104fdb)
* [Best Practices of using Data Caching in React Native Projects](https://medium.com/@tusharkumar27864/best-practices-of-using-data-caching-redis-local-storage-in-react-native-projects-e151c76b2df0)
* [Optimizing Performance with Caching in React Native](https://blog.stackademic.com/optimizing-performance-with-caching-in-react-native-a-step-by-step-guide-a9cac5cd5389)
* [Boosting React Native Performance with Caching](https://blog.mrinalmaheshwari.com/boosting-react-native-performance-with-caching-made-simple-79d5269f0cc3)
* [React Native Data Persistence: A Comprehensive Guide](https://blog.krybot.com/t/react-native-data-persistence-a-comprehensive-guide/22294)

## 5. Evaluasi Harian (Soal Praktik)

`Lanjutan project Mini E-Commerce`

Tugas-tugas ini berfokus untuk menguji kemampuanmu dalam mengelola data agar persisten / offline - capable:

* **a. Persistensi Token Otentikasi:** Simpan **token masuk** di AsyncStorage. Cek token ini saat aplikasi pertama kali dibuka untuk menentukan apakah pengguna langsung ke **Home** atau harus ke **Login**. (Guard Flow)
* **b. Cache-First Kategori Produk:** Terapkan **TTL (30 menit)** untuk data kategori produk. Gunakan *cache* jika ada dan belum kadaluarsa, terutama saat **offline**.
* **c. Optimasi Multi-Key Load:** Gunakan **`multiGet`** saat load aplikasi, untuk mengambil data - data penting, e.g (token, tema, status notifikasi) secara serentak untuk mempercepat *loading*.
* **d. Persistensi Cart (Keranjang):** Gunakan **`mergeItem`** saat ada perubahan kecil (misal: tambah jumlah item) di keranjang. Tangani simulasi **error Quota Exceeded** jika penyimpanan penuh.
* **e. Cleanup Saat Logout:** Buat fungsi *logout* terpusat yang menggunakan **`multiRemove`** untuk menghapus semua 3 data sensitif (token, prefs, dll.) sekaligus.

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

---

**Mobile App Development With React Native*
