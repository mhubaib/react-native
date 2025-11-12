# Hari ke-15 - Storage: AsyncStorage

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

- Memahami AsyncStorage sebagai key-value store asynchronous yang unencrypted dan thread-safe, serta mengimplementasikannya untuk operasi dasar seperti setItem, getItem, removeItem, dan multi-methods untuk batch operations, memastikan persistensi data antar app restarts.
- Mengimplementasikan pola penggunaan AsyncStorage untuk menyimpan data kecil seperti user settings, auth tokens, atau temporary states, dengan pengelolaan concurrency via async/await dan error handling dari Hari ke-14 untuk robustness.
- Mengelola teknik caching data lokal dengan AsyncStorage di berbagai kondisi seperti offline access (cache-first), TTL (time-to-live) untuk freshness, dan size limits (max 6MB), sambil memastikan cleanup rutin untuk mencegah storage bloat.
- Menerapkan praktik terbaik seperti namespacing keys (e.g., '@app:userPrefs'), serialization (JSON.stringify/parse), dan integration dengan networking (cache API responses), untuk aplikasi yang efisien dan offline-capable.
- Menganalisis dan menyelesaikan studi kasus caching seperti persistent user preferences, API response caching untuk reduced latency, dan handling quota exceeded errors, dengan langkah-langkah debugging menggunakan console dan AsyncStorage.clear() untuk testing.
- Menerapkan troubleshooting umum seperti async race conditions atau data corruption dari improper serialization, untuk storage yang scalable di RN 0.75+.
- Membangun prototipe dengan AsyncStorage lengkap, seperti caching mechanism untuk data fetch dari Hari ke-13, siap untuk integrasi dengan state management seperti Context atau Redux.

Tujuan ini selaras dengan update RN 0.75+ (2025): AsyncStorage tetap sebagai community-maintained (non-core sejak 0.60), dengan enhanced async patterns untuk better concurrency di multi-threaded environments seperti JSI.

## 2. Materi Pembelajaran

Bagian ini menjelaskan AsyncStorage secara mendalam, termasuk API dasar, pola caching, dan studi kasus dengan langkah penyelesaian. AsyncStorage adalah wrapper asynchronous untuk native key-value stores (NSUserDefaults iOS, SharedPreferences Android), unencrypted dan synchronous-free untuk UI thread safety. Per 2025, best practices menekankan TTL caching untuk data freshness dan namespacing untuk collision avoidance. Integrasikan dengan useEffect untuk lifecycle loads/saves, dan error handling dari Hari ke-14 untuk robustness.

### A. API Dasar AsyncStorage: Operasi Kunci-Value Asynchronous

**Tujuan:** AsyncStorage menyediakan API simple untuk persistensi, dengan dukungan multi-key operations untuk efficiency, memastikan data bertahan antar app kills/restarts.

**Methods Kunci (v1.23+, 2025):**

| Method | Tipe | Deskripsi | Return | Catatan Platform & Best Practices |
|--------|------|-----------|--------|----------------------------------|
| `setItem(key, value)` | async (string, string) => Promise<void> | Simpan string value untuk key. | Promise | JSON.stringify non-strings; key: '@app:username'; batasi value <1MB per item. |
| `getItem(key)` | async (string) => Promise<string|null> | Ambil value; null jika tidak ada. | Promise | await getItem(key); parse dengan JSON.parse untuk objects; handle null dengan defaults. |
| `removeItem(key)` | async (string) => Promise<void> | Hapus key. | Promise | Gunakan untuk logout (remove '@app:token'); batch dengan multiRemove untuk multiple. |
| `mergeItem(key, value)` | async (string, string) => Promise<void> | Merge JSON objects (shallow). | Promise | { ...existing, ...new }; ideal untuk incremental updates seperti settings. |
| `multiGet(keys)` | async (string[]) => Promise<[string, string|null][]> | Batch get multiple keys. | Promise | [[key1, value1], [key2, null]]; gunakan untuk load app state (e.g., user + prefs). |
| `multiSet(keyValuePairs)` | async ([string, string][]) => Promise<void> | Batch set multiple. | Promise | [['key1', 'val1'], ['key2', 'val2']]; efisien untuk initial setup. |
| `multiRemove(keys)` | async (string[]) => Promise<void> | Batch delete. | Promise | Untuk cleanup (e.g., multiRemove(['@app:token', '@app:prefs'])); async await untuk sequential. |
| `multiMerge(keyValuePairs)` | async ([string, string][]) => Promise<void> | Batch merge. | Promise | Incremental batch updates; gunakan untuk sync partial data. |
| `getAllKeys()` | async () => Promise<string[]> | List semua keys. | Promise | Untuk debug/migration; filter dengan '@app:' prefix; batasi use untuk privasi. |
| `clear()` | async () => Promise<void> | Hapus semua data. | Promise | Gunakan di logout/debug; confirm dengan Alert untuk production. |
| `flushGetRequests()` | async () => Promise<void> | Flush pending ops (legacy). | Promise | Deprecated; gunakan di old codebases; RN 0.75+: Auto-flush. |

**Pola Penggunaan:** Basic: `await AsyncStorage.setItem('username', JSON.stringify({ name: 'John' })); const user = JSON.parse(await AsyncStorage.getItem('username')) || { name: 'Guest' };`. Pola dengan useEffect: useEffect(() => { loadPrefs(); }, []); untuk app startup. Multi: `const [[key1, val1]] = await AsyncStorage.multiGet(['key1']);`.

**Praktik Terbaik (2025):** Namespacing: Prefix keys ('@app:'); serialize objects dengan JSON (handle circular refs dengan custom replacer); batasi size (6MB total); async/await untuk readability; error handling (try-catch) untuk quota exceeded. RN 0.75+: Better thread safety untuk JSI bridge.

**Pertimbangan Platform:** iOS: NSUserDefaults (fast, small); Android: SharedPreferences (encrypted option via KeyStore, tapi AsyncStorage unencrypted by default).

### B. Teknik Caching Data Lokal dengan AsyncStorage: Pola dan Strategi

**Tujuan:** Caching dengan AsyncStorage mengurangi API calls (dari Hari ke-13) dan mendukung offline UX, dengan teknik seperti cache-first, TTL, dan conditional invalidation untuk freshness dan efficiency.

**Teknik Kunci:**

| Teknik | Deskripsi | Kapan Digunakan | Contoh Pola |
|--------|-----------|-----------------|-------------|
| **Cache-First (Offline-Capable)** | Check cache sebelum API; fallback jika stale atau missing. | Data semi-static seperti user profile; kurangi latency. | if (cached) return cached; else fetch + set cache. |
| **TTL (Time-to-Live)** | Timestamp + expiry check; evict setelah TTL. | Data volatile seperti news feeds; TTL 5-30min. | { data, timestamp }; if (Date.now() - timestamp < TTL) use cache. |
| **Write-Through** | Cache + API update simultan; consistency tinggi. | Critical data seperti cart items; sync on write. | setCache(data); api.post(data); on success update cache. |
| **Conditional Invalidation** | Clear cache on events (e.g., logout, data update). | User-specific; invalidasi manual via listeners. | useEffect(() => clearCache();, [userId]);. |
| **Batch Caching** | Multi-set/get untuk related data. | App state (prefs + tokens); efisien untuk startup. | multiSet([['prefs', JSON.stringify(prefs)], ['token', token]]);. |

**Pola Penggunaan:** Custom hook: `const useCache = (key, ttl = 300000) => { const [data, setData] = useState(null); useEffect(() => { const load = async () => { const cached = await AsyncStorage.getItem(key); if (cached) { const { value, ts } = JSON.parse(cached); if (Date.now() - ts < ttl) { setData(value); return; } } // Fetch + cache }; load(); }, []); return [data, setData]; };`. Pola dengan networking: Cache API response post-fetch; check TTL sebelum use.

**Praktik Terbaik (2025):** Limit keys to <1000; size <6MB total (monitor dengan getAllKeys); namespacing + versioning (e.g., '@app:v2:prefs'); cleanup on app quit (AppState listener). Integrasi error handling: Try-catch di async ops; fallback to defaults jika corrupt (JSON.parse fail). RN 0.75+: AsyncStorage hooks dengan better concurrency untuk multi-thread access.

**Pertimbangan Platform:** iOS: Fast read/write; Android: Slower on low-end devices; keduanya unencrypted—use Keychain/Keystore untuk sensitive (library opsional).

### C. Studi Kasus Caching dengan AsyncStorage: Contoh dan Penyelesaian

**Tujuan:** Studi kasus menunjukkan aplikasi praktis caching di berbagai kondisi, dengan langkah implementasi dan debugging untuk real-world scenarios.

**Studi Kasus 1: Caching API Response untuk Offline Access (Cache-First)**

- **Deskripsi:** App fetch user list dari API; cache untuk offline view, kurangi API calls 50% (per studi Medium 2025).
- **Kondisi:** User offline setelah load pertama; data tetap accessible.
- **Langkah Implementasi:**
  1. **Setup:** Custom hook useApiCache(key): Check AsyncStorage.getItem(key); jika ada, parse + return; else fetch + setItem(JSON.stringify({ data, timestamp: Date.now() })).
  2. **Pola:** useEffect: const [data, loading] = useApiCache('@app:users'); if (loading) show spinner; fallback <Text>No data available</Text>.
  3. **Debugging:** Console.log(cache hit/miss); test offline dengan airplane mode; clear cache untuk simulate fresh fetch.
  4. **Outcome:** Reduced latency 80ms → 2ms on cache hit; UX seamless.

**Studi Kasus 2: Persistent User Preferences dengan TTL (Settings Cache)**

- **Deskripsi:** Simpan theme ('dark'/'light') dan language; TTL 24h untuk auto-refresh dari server sync; handle quota exceeded jika prefs terlalu besar.
- **Kondisi:** User ubah prefs offline; apply on reconnect; evict jika expired.
- **Langkah Implementasi:**
  1. **Setup:** setPrefs(prefs): await AsyncStorage.setItem('@app:prefs', JSON.stringify({ prefs, ts: Date.now() })); getPrefs(): const cached = await getItem('@app:prefs'); if (Date.now() - ts < 86400000) return prefs.
  2. **Pola:** Context provider: useState dari getPrefs(); listener on prefs change untuk setPrefs; fallback default prefs jika expired/missing.
  3. **Debugging:** Log getAllKeys() untuk check size; simulate expiry dengan manual timestamp edit; test quota: multiSet large data hingga fail (catch error).
  4. **Outcome:** Persistent UX; sync on reconnect; prevent bloat dengan TTL.

**Studi Kasus 3: Batch Caching untuk App Startup (Multi-Key Load)**

- **Deskripsi:** Load multiple data (token, prefs, lastSeen) on app launch; batch untuk efisiensi, handle partial failure jika satu key corrupt.
- **Kondisi:** App crash jika satu data missing; optimize load time dari 500ms ke 100ms.
- **Langkah Implementasi:**
  1. **Setup:** loadAppState(): const keys = ['@app:token', '@app:prefs', '@app:lastSeen']; const results = await multiGet(keys); parse each; fallback defaults jika null.
  2. **Pola:** useEffect on App load: const [state, setState] = useState({}); loadAppState().then(setState); show loading hingga all loaded.
  3. **Debugging:** Console.log(results); test corrupt: setItem corrupt JSON, catch JSON.parse error dengan default; measure time dengan console.time('loadState').
  4. **Outcome:** Faster startup; resilient to partial corruption.

**Praktik Terbaik untuk Caching (2025):** Gunakan TTL untuk volatile data; batch ops untuk >3 keys; monitor storage dengan getAllKeys periodically (e.g., on app background). Integrasi networking: Cache post-fetch; invalidasi on error (e.g., 304 Not Modified). RN 0.75+: Better async batching untuk concurrent reads.

**Pertimbangan Platform:** iOS: Synchronous fallback untuk small reads; Android: Async required untuk UI thread safety.

**Integrasi Keseluruhan:** AsyncStorage + networking: Cache API responses; load prefs on app start; error handling untuk quota/JSON errors.

## 3. Contoh Implementasi

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

### C. Contoh Lanjutan: Batch Caching dan Studi Kasus

Untuk studi kasus, gunakan kode di atas sebagai base.

**Studi Kasus 1: Caching API Response untuk Offline Access**

- Implementasi: Gunakan useTTLCache di fetchUsers; test offline: Data tetap tampil jika dalam TTL.
- Debugging: Console.log('Cache hit'); airplane mode untuk simulate.

**Studi Kasus 2: Persistent User Preferences**

- Implementasi: multiGet(['@app:theme', '@app:lang']); fallback defaults jika null.
- Debugging: getAllKeys() untuk verify; manual set corrupt JSON untuk test parse error.

**Studi Kasus 3: Batch Startup Load**

- Implementasi: loadAppState(): multiGet(keys); setState partial jika satu fail.
- Debugging: console.time('load'); test dengan removeItem random keys.

**Tips Debugging:** Console.log(keys/values); AsyncStorage.clear() untuk reset; test quota dengan loop setItem hingga fail.

## 4. Rangkuman

Hari ke-15 membangun persistensi dengan AsyncStorage melalui API dasar (set/get/remove/multi), pola caching (TTL, cache-first, batch), dan studi kasus (offline API cache, prefs TTL, startup batch) dengan langkah implementasi/debugging. Kunci: Namespacing + JSON serialization; TTL untuk freshness; multi-ops untuk efficiency. Integrasi ini ciptakan apps offline-capable dan performant, siap untuk advanced storage seperti SQLite.

**Latihan Selanjutnya:** Implementasikan caching untuk networking dari Hari ke-13; tambah TTL listener untuk auto-refresh.

**Referensi:**

- [Storing Data Permanently in React Native using AsyncStorage 2025](https://medium.com/@mahesh.nikate/storing-data-permanently-in-react-native-using-asyncstorage-2025-91a79b104fdb)
- [Best Practices of using Data Caching in React Native Projects](https://medium.com/@tusharkumar27864/best-practices-of-using-data-caching-redis-local-storage-in-react-native-projects-e151c76b2df0)
- [Optimizing Performance with Caching in React Native](https://blog.stackademic.com/optimizing-performance-with-caching-in-react-native-a-step-by-step-guide-a9cac5cd5389)
- [Boosting React Native Performance with Caching](https://blog.mrinalmaheshwari.com/boosting-react-native-performance-with-caching-made-simple-79d5269f0cc3)
- [React Native Data Persistence: A Comprehensive Guide](https://blog.krybot.com/t/react-native-data-persistence-a-comprehensive-guide/22294)

## 5. Evaluasi Harian: Soal Praktik

`Lanjutan project Mini E-Commerce`

Soal-soal ini menguji kemampuan Anda untuk membuat aplikasi yang *offline-capable* dan efisien dalam manajemen data.

---

### a. Persistensi Token Otentikasi dan *Guarded Flow*

**Tugas:** Integrasikan AsyncStorage untuk mengelola **token otentikasi** pengguna.

1. Setelah *simulasi login* berhasil, **simpan** token ke AsyncStorage dengan *key* yang di-namespaced (misalnya, `'@ecom:authToken'`).
2. Di **Root Navigator** aplikasi Anda, buat *logic* untuk memeriksa token ini saat aplikasi **pertama kali dimuat**.
3. Jika token **ditemukan**, navigasikan pengguna langsung ke **Top Tabs Navigator** (Screen Home). Jika **tidak ditemukan**, navigasikan ke *Screen* **Login**.

---

### b. *Cache-First* Strategy dengan TTL untuk Kategori Produk

**Tugas:** Implementasikan *cache-first strategy* dengan **Time-to-Live (TTL)** untuk data **kategori produk** (data yang dianggap semi-statis).

1. Buat fungsi yang pertama-tama mencoba memuat data kategori dari AsyncStorage.
2. Data yang tersimpan harus memiliki **timestamp**. Jika data **kadaluarsa** (misalnya, lebih dari 30 menit), *fetch* data baru dari API dan perbarui *cache*.
3. Jika pengguna **offline**, dan *cache* **tidak expired**, gunakan data dari *cache* tersebut. Jika *cache* expired dan offline, tampilkan pesan error.

---

### c. Multi-Key Load Optimization di *Splash Screen*

**Tugas:** Optimalkan waktu *loading* awal aplikasi dengan menggunakan **`AsyncStorage.multiGet`**.

1. Saat aplikasi *loading* (di *Splash Screen*), gunakan `multiGet` untuk mengambil 3 *keys* penting secara bersamaan: **token otentikasi**, **preferensi pengguna** (misalnya, tema *dark/light*), dan **status notifikasi terakhir**.
2. Proses *parsing* dan *state* aplikasi (otentikasi, tema) hanya boleh berlanjut setelah **semua 3 *key*** berhasil diambil (atau bernilai `null`).
3. Ukur waktu *loading* menggunakan `console.time()` untuk membandingkan efisiensi *multiGet* dengan *getItem* sequential.

---

### d. Persistensi *Cart State* dan *Merge* Incremental

**Tugas:** Terapkan persistensi untuk status **Keranjang Belanja (Cart)** Anda, yang memungkinkan pengguna menutup aplikasi dan melanjutkan belanja.

1. Gunakan **`AsyncStorage.mergeItem`** setiap kali item ditambahkan ke keranjang (untuk menghemat *write cycle*). *Merge* hanya *field* yang berubah (misalnya, `quantity` dari produk tertentu).
2. Jika terjadi error **Quota Exceeded** saat menyimpan (simulasikan *try-catch* error untuk *setItem*), tampilkan *Alert* yang menyarankan pengguna untuk **menghapus beberapa item** dari keranjang mereka (simulasi *storage warning*).

---

### e. *Cleanup* Data Sensitif Saat *Logout*

**Tugas:** Buat fungsi **`handleLogout`** yang terpusat. Ketika pengguna menekan tombol **Logout** di *Screen* Profile/Settings:

1. Gunakan **`AsyncStorage.multiRemove`** untuk menghapus **minimal 3 *keys*** yang sensitif atau bersifat pribadi secara serentak: `authToken`, `userPrefs`, dan `lastViewedProduct`.
2. Setelah *data sensitif* dihapus, **reset** navigasi untuk memastikan pengguna kembali ke *Screen* Login dan menghapus *stack* navigasi yang lama.

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

---

**Mobile App Development With React Native*
