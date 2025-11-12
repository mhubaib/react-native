# Hari ke-13 - Networking

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

- Memahami Fetch API sebagai dasar networking built-in React Native, serta mengimplementasikannya untuk HTTP requests (GET/POST/PUT/DELETE) dengan pengelolaan yang baik seperti headers, timeouts, dan AbortController untuk performa tinggi dan pembatalan request.
- Mengimplementasikan Axios sebagai alternatif library untuk requests yang lebih fleksibel, termasuk konfigurasi instance, interceptors untuk headers dinamis, dan pengaturan timeouts global untuk konsistensi di seluruh aplikasi.
- Mengelola pengelolaan status koneksi internet menggunakan NetInfo library untuk deteksi online/offline real-time, listener changes, dan conditional rendering (e.g., show loading/offline states), sambil mengintegrasikannya dengan requests untuk adaptasi jaringan.
- Menerapkan praktik terbaik seperti HTTPS enforcement, request cancellation untuk unmount, dan polling dengan intervals optimal, untuk aplikasi yang responsif di berbagai kondisi jaringan (WiFi/3G/5G).
- Mengintegrasikan networking dengan useEffect/useState untuk lifecycle management, memastikan requests hanya berjalan saat diperlukan dan dibersihkan dengan benar.
- Menerapkan troubleshooting umum seperti request hanging atau inconsistent NetInfo di emulators, untuk networking yang scalable di RN 0.75+.
- Membangun prototipe fitur networking sederhana dengan koneksi check dan requests optimal, siap untuk integrasi dengan navigasi atau state management lanjutan.

Tujuan ini selaras dengan update RN 0.75+ (2025): Emphasis pada Fetch dengan AbortController untuk cancel requests, dan NetInfo v10+ dengan better isInternetReachable untuk accurate connectivity di 5G/edge networks.

## 2. Materi Pembelajaran

Bagian ini menjelaskan pengelolaan networking yang baik secara mendalam, termasuk Fetch/Axios untuk requests efisien, dan NetInfo untuk status koneksi. Networking di RN menggunakan JS Fetch polyfill untuk kompatibilitas iOS/Android, dengan Axios menambahkan abstractions seperti instance configs. Per 2025, best practices menekankan HTTPS only dan cancellation untuk prevent memory leaks. Integrasikan dengan useEffect untuk lifecycle, fokus pada optimalisasi seperti timeouts dan listener koneksi.

### A. Fetch API: Built-in Networking untuk HTTP Requests yang Optimal

**Tujuan:** Fetch adalah API standar untuk async HTTP, mendukung promises dan modern syntax, ideal untuk pengelolaan requests sederhana dengan kontrol penuh atas headers, methods, dan cancellation untuk performa tinggi.

**Methods dan Options:**

| Method/Option | Tipe | Deskripsi | Default | Catatan Platform & Best Practices |
|---------------|------|-----------|---------|----------------------------------|
| `fetch(url, options)` | Function | Kirim request; return Promise<Response>. | - | Gunakan async/await: `const response = await fetch(url); const data = await response.json();`. |
| `method` | 'GET'/'POST'/'PUT'/'DELETE'/'PATCH' | HTTP verb. | 'GET' | POST: { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(data) }; gunakan untuk CRUD optimal. |
| `headers` | Headers object | Custom headers (e.g., Accept, User-Agent). | {} | { 'Accept': 'application/json', 'Content-Type': 'application/json' }; set global di app untuk consistency. |
| `body` | string/Blob/FormData | Payload untuk non-GET. | null | JSON.stringify(data) untuk JSON; FormData untuk file uploads dengan multipart. |
| `signal` | AbortSignal | Cancel request (dari AbortController). | null | const controller = new AbortController(); fetch(url, { signal: controller.signal }); controller.abort(); untuk unmount atau timeout. |
| `timeout` | number (ms) | Auto-abort setelah delay (via signal). | No default | Custom: setTimeout(() => controller.abort(), 8000); 5-10s optimal untuk mobile latency. |
| `response.json()` | Method | Parse JSON response. | - | Async: await response.json(); pastikan headers set 'application/json' untuk auto-parse. |
| `response.headers` | Headers | Akses response headers (e.g., ETag untuk caching future). | - | Gunakan untuk conditional requests (e.g., if-match headers). |

**Pola Penggunaan:** GET: `fetch('https://api.example.com/users').then(res => res.json())`. POST: `fetch(url, { method: 'POST', body: JSON.stringify({ name: 'John' }) })`. Pola dengan useEffect: useEffect(() => { const controller = new AbortController(); fetchData(controller); return () => controller.abort(); }, []); untuk cleanup optimal.

**Praktik Terbaik:** Enforce HTTPS (url.startsWith('https://')); gunakan AbortController di setiap request untuk prevent leaks; set reasonable timeouts (8s untuk GET, 15s untuk POST); gunakan headers untuk rate limiting (e.g., User-Agent custom). RN 0.75+: Better support untuk streaming responses di large payloads.

**Pertimbangan Platform:** iOS: Native NSURLSession dengan HTTP/2; Android: OkHttp dengan multiplexing; keduanya optimal di 2025 untuk concurrent requests.

### B. Axios: Library untuk Pengelolaan Requests yang Konsisten

**Tujuan:** Axios menyederhanakan Fetch dengan instance configs, automatic JSON handling, dan interceptors untuk headers dinamis, memastikan pengelolaan requests yang terstruktur dan reusable di seluruh app.

**Setup Dasar:** `import axios from 'axios';` Buat instance: `const api = axios.create({ baseURL: 'https://api.example.com', timeout: 10000 });`.

**Methods dan Config Kunci (Axios 1.7+):**

| Method/Config | Tipe | Deskripsi | Default | Catatan Platform & Best Practices |
|---------------|------|-----------|---------|----------------------------------|
| `axios.get/post/put/delete(url, config)` | Function | Request shortcuts. | - | axios.post(url, data, { headers: { 'Content-Type': 'application/json' } }); auto JSON handling untuk optimal flow. |
| `baseURL` | string | Prefix semua requests. | - | Env: `baseURL: process.env.API_BASE`; switch dev/prod untuk flexibility. |
| `timeout` | number (ms) | Abort setelah delay. | 0 (no timeout) | 8000-15000; set global untuk consistency, adjust per method (e.g., longer untuk uploads). |
| `headers` | object | Global headers. | {} | { 'Content-Type': 'application/json', 'Accept': 'application/json' }; interceptors untuk dynamic updates. |
| `interceptors.request.use(fn)` | Method | Pre-request hook. | - | fn(config) => { config.headers['User-Agent'] = 'MyApp/1.0'; return config; }; untuk logging atau retries. |
| `interceptors.response.use(fn)` | Method | Post-response hook. | - | fn(res => res); untuk transform data (e.g., normalize responses). |
| `cancelToken` | CancelToken | Cancel ongoing requests. | - | const source = CancelToken.source(); axios.get(url, { cancelToken: source.token }); source.cancel(); untuk unmount optimal. |
| `transformRequest` | Array<function> | Pre-body transform. | JSON stringify | Custom: fn(data) => data; untuk encryption atau serialization. |
| `httpAgent` | Agent | Custom agent untuk pooling. | Default | Gunakan untuk keep-alive connections di long sessions. |

**Pola Penggunaan:** Instance: `api.get('/users').then(res => setData(res.data))`. Interceptor: `api.interceptors.request.use(addHeaders);`. Pola dengan useEffect: useEffect(() => { const source = CancelToken.source(); fetchData(source); return () => source.cancel(); }, []); untuk cancellation konsisten.

**Praktik Terbaik (2025):** Gunakan instance per environment; interceptors untuk common headers (e.g., Accept-Encoding: gzip); cancelToken di setiap request untuk prevent stale data. Integrasi dengan NetInfo: Check koneksi sebelum dispatch. RN CLI: Polyfill jika butuh advanced agents (jarang).

**Pertimbangan Platform:** iOS: Seamless dengan URLSession; Android: OkHttp integration untuk faster redirects. 2025: Axios 1.7+ optimized untuk RN dengan better concurrency.

### C. Pengelolaan Status Koneksi Internet dengan NetInfo

**Tujuan:** NetInfo (@react-native-community/netinfo v10+, 2025) deteksi koneksi real-time (WiFi/cellular/offline), dengan isInternetReachable untuk validasi actual access (bukan hanya connected), mendukung listener changes untuk dynamic UI (e.g., adjust request strategies).

**Setup Dasar:** `import NetInfo from '@react-native-community/netinfo';` Subscribe: `const unsubscribe = NetInfo.addEventListener(state => { setIsConnected(state.isConnected); });` Cleanup: `unsubscribe()` di useEffect return.

**Methods dan Props Kunci (v10+):**

| Method/Prop | Tipe | Deskripsi | Default | Catatan Platform & Best Practices |
|-------------|------|-----------|---------|----------------------------------|
| `NetInfo.fetch()` | Promise<ConnectionInfo> | One-time check status. | - | await NetInfo.fetch(); return { isConnected: true, isInternetReachable: true, type: 'wifi' }; untuk initial load. |
| `isConnected` | boolean | Device connected to network (WiFi/cellular). | false | Basic check; false jika airplane mode; gunakan untuk gate requests. |
| `isInternetReachable` | boolean | Actual internet access (ping test). | null | Async: await NetInfo.fetch().then(state => state.isInternetReachable); true jika reachable untuk critical fetches. |
| `type` | 'none'/'wifi'/'cellular'/'other'/'unknown' | Connection type. | 'unknown' | 'cellular' untuk metered; throttle payloads di cellular untuk data saving. |
| `effectiveType` | '2g'/'3g'/'4g'/'slow-2g' | Speed estimate. | 'unknown' | Adjust quality (e.g., low-res images di '2g'); optimal untuk adaptive loading. |
| `addEventListener(listener)` | Listener<ConnectionInfo> | Subscribe changes. | - | useEffect: const sub = NetInfo.addEventListener(setConnection); return sub(); untuk real-time UI updates. |
| `removeEventListener` | Listener | Unsubscribe. | - | Selalu cleanup untuk hindari leaks; v10+: Auto-cleanup di unmount. |
| `configure(options)` | object | Custom ping URL (e.g., { url: '<https://example.com/ping>' }). | Default (google.com) | { url: '<https://yourapi.com/health>' }; untuk backend-specific checks. |

**Pola Penggunaan:** Hook custom: `const [isOnline, setIsOnline] = useState(true); useEffect(() => { const sub = NetInfo.addEventListener(state => { setIsOnline(state.isConnected && state.isInternetReachable); }); return () => sub(); }, []);`. Pola adaptif: if (!isOnline) disable polling; adjust timeouts di 'cellular'.

**Praktik Terbaik (2025):** Kombinasikan isConnected + isInternetReachable untuk accuracy; listener di App root untuk global state; use effectiveType untuk bandwidth optimization. v10+: Better 5G detection di Android 15+; test di real devices dengan network toggle.

**Pertimbangan Platform:** iOS: NWPathMonitor native; Android: ConnectivityManager; keduanya accurate di 2025, tapi emulator laggy—test device.

**Integrasi Keseluruhan:** Networking flow: NetInfo check → Conditional request (Axios/Fetch) → UI adaptif berdasarkan type.

## 3. Contoh Implementasi

### A. Contoh Dasar: Fetch API dengan Timeout dan Cancellation

```jsx
// components/UserList.js
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, ActivityIndicator } from 'react-native';

const UserList = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), 8000); // 8s timeout

    fetch('https://jsonplaceholder.typicode.com/users', {
      signal: controller.signal,
      headers: { 'Accept': 'application/json' },
    })
      .then(res => res.json())
      .then(data => setUsers(data))
      .catch(err => {
        if (err.name === 'AbortError') console.log('Request timed out or cancelled');
      })
      .finally(() => {
        clearTimeout(timeoutId);
        setLoading(false);
      });

    return () => {
      controller.abort();
      clearTimeout(timeoutId);
    };
  }, []);

  if (loading) return <ActivityIndicator size="large" />;

  return (
    <FlatList
      data={users}
      keyExtractor={item => item.id.toString()}
      renderItem={({ item }) => <Text>{item.name}</Text>}
    />
  );
};

export default UserList;
```

**Penjelasan:** Fetch GET dengan AbortController dan setTimeout untuk optimal cancellation; cleanup di return.

### B. Contoh Interaktif: Axios Instance dengan Interceptors

```jsx
// api/apiClient.js
import axios from 'axios';

const api = axios.create({
  baseURL: 'https://jsonplaceholder.typicode.com',
  timeout: 10000, // Global timeout
  headers: { 'Accept': 'application/json' },
});

// Request interceptor untuk headers dinamis
api.interceptors.request.use(config => {
  // Tambah headers umum, e.g., User-Agent
  config.headers['User-Agent'] = 'MyApp/1.0';
  return config;
});

// Response interceptor untuk transform
api.interceptors.response.use(
  res => {
    // Auto-transform jika diperlukan
    return res;
  }
);

export default api;
```

**Integrasi di Component:**

```jsx
// components/UsersWithAxios.js
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, ActivityIndicator } from 'react-native';
import api from '../api/apiClient';

const UsersWithAxios = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const source = axios.CancelToken.source(); // Cancellation

    api.get('/users', { cancelToken: source.token })
      .then(res => setUsers(res.data))
      .catch(err => {
        if (axios.isCancel(err)) console.log('Request cancelled');
      })
      .finally(() => setLoading(false));

    return () => source.cancel();
  }, []);

  if (loading) return <ActivityIndicator size="large" />;

  return (
    <FlatList
      data={users}
      keyExtractor={item => item.id.toString()}
      renderItem={({ item }) => <Text>{item.name}</Text>}
    />
  );
};

export default UsersWithAxios;
```

**Penjelasan:** Instance Axios dengan global timeout/headers; interceptor sederhana; cancelToken untuk unmount.

### C. Contoh Lanjutan: Pengelolaan Status Koneksi dengan NetInfo

```jsx
// hooks/useNetInfo.js
import { useState, useEffect } from 'react';
import NetInfo from '@react-native-community/netinfo';

export const useNetInfo = () => {
  const [isOnline, setIsOnline] = useState(true);
  const [connectionType, setConnectionType] = useState('unknown');

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(state => {
      const reachable = state.isConnected && state.isInternetReachable;
      setIsOnline(reachable);
      setConnectionType(state.type || 'unknown');
    });

    // Initial fetch
    NetInfo.fetch().then(state => {
      setIsOnline(state.isConnected && state.isInternetReachable);
      setConnectionType(state.type);
    });

    return () => unsubscribe();
  }, []);

  return { isOnline, connectionType };
};
```

**Integrasi di Component dengan Networking:**

```jsx
// components/ConnectedUsers.js
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, ActivityIndicator } from 'react-native';
import { useNetInfo } from '../hooks/useNetInfo';
import api from '../api/apiClient';

const ConnectedUsers = () => {
  const { isOnline, connectionType } = useNetInfo();
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    if (!isOnline) {
      setLoading(false);
      return;
    }

    const source = api.CancelToken.source();

    api.get('/users', { cancelToken: source.token })
      .then(res => setUsers(res.data))
      .finally(() => setLoading(false));

    return () => source.cancel();
  }, [isOnline]);

  if (loading) return <ActivityIndicator size="large" />;

  return (
    <View style={{ flex: 1 }}>
      <Text>Connection: {isOnline ? 'Online' : 'Offline'} ({connectionType})</Text>
      <FlatList
        data={users}
        keyExtractor={item => item.id.toString()}
        renderItem={({ item }) => <Text>{item.name}</Text>}
      />
    </View>
  );
};

export default ConnectedUsers;
```

**Penjelasan:** Custom hook NetInfo listener + initial fetch; conditional request berdasarkan isOnline; UI tampilkan type; cancellation integrasi.

**Tips Debugging:** Gunakan Chrome DevTools Network tab untuk throttle; test NetInfo di real device dengan WiFi toggle.

## 4. Rangkuman

Hari ke-13 membangun pengelolaan networking yang baik melalui Fetch/Axios untuk requests optimal (methods/headers/timeouts/cancellation), dan NetInfo untuk status koneksi real-time (isConnected/reachable, listener adaptif). Kunci: Async/await dengan AbortController/CancelToken; instance configs untuk reusability; hook NetInfo untuk conditional flows. Integrasi ini ciptakan apps responsif di jaringan variatif, dengan cleanup yang mencegah leaks, siap untuk error/caching lanjutan.

**Latihan Selanjutnya:** Bangun POST form dengan NetInfo check dan Axios interceptor; integrasikan ke navigasi.

**Referensi:**

- [Networking - React Native](https://reactnative.dev/docs/network)
- [Mastering Network Requests in React Native - LinkedIn](https://www.linkedin.com/pulse/mastering-network-requests-react-native-essential-tips-speed-p5wpe)
- [@react-native-community/netinfo - GitHub](https://github.com/react-native-netinfo/react-native-netinfo)
- [Building a Reliable Internet Connectivity Checker in React Native - Medium](https://medium.com/@Sunil_Kumar_RN/building-a-reliable-internet-connectivity-checker-in-react-native-f20b77ce0d76)
- [Managing network connection status in React Native - LogRocket Blog](https://blog.logrocket.com/managing-network-connection-status-in-react-native/)

## 5. Evaluasi Harian: Soal Praktik

`Lanjutan project Mini E-Commerce`

- Gunakan API berikut: `https://dummyjson.com/docs/products`

### a. Pengelolaan *Lifecycle* dengan Fetch API dan Pembatalan (*Cancellation*)

**Tugas:** Buat komponen *Screen* **'Product List'** yang memuat data daftar produk (GET request) menggunakan **Fetch API** dari *endpoint* simulasi. Pastikan *request* tersebut **selalu dibatalkan** jika pengguna **meninggalkan layar** (komponen di-*unmount*) atau jika *request* memakan waktu lebih dari **7 detik** (*timeout*). Implementasikan pembatalan ini menggunakan **`AbortController`** dan **`useEffect`** *cleanup function*.

---

### b. Validasi Koneksi dan Adaptasi UI dengan NetInfo

**Tugas:** Sebelum memuat data di *Screen* **'Product List'**, wajib melakukan pengecekan koneksi menggunakan **NetInfo**.

1. Jika *status* menunjukkan **tidak ada koneksi internet** (*isInternetReachable* adalah `false`), **hentikan** *request* dan tampilkan **UI *placeholder*** yang mengatakan "Anda sedang Offline. Cek koneksi Anda."
2. Tampilkan **jenis koneksi** yang sedang digunakan (`wifi` atau `cellular`) di bagian bawah layar.

---

### c. Konfigurasi Axios Instance untuk Global Header

**Tugas:** Gunakan **Axios** untuk mengelola semua *request* (simulasi login, detail, dll.) di aplikasi E-Commerce Anda. Buat **Axios *instance*** terpisah (`apiClient`) yang memiliki konfigurasi:

1. **`baseURL`** yang mengarah ke API simulasi Anda.
2. **`timeout`** global 10 detik.
3. Gunakan **Interceptor Request** untuk secara otomatis menambahkan *custom header* **`X-Client-Platform: React-Native`** ke setiap *request* yang dikirim, tanpa harus menuliskannya di setiap panggilan API.

---

### d. Simulasi *Login* dengan Axios POST dan Data Transformasi

**Tugas:** Buat *Screen* **'Login'** sederhana yang melakukan *request* **POST** menggunakan **Axios *instance*** (dari Soal 18) ke *endpoint* simulasi login.

1. Kirim data simulasi (`username` dan `password`) di *body* *request*.
2. Tambahkan **Interceptor Response** pada *Axios instance* Anda yang bertugas memeriksa *status code* *response*. Jika *status* adalah **200 (OK)**, ubah data *response* menjadi objek sederhana `{ success: true, token: 'simulated_token_xyz' }` sebelum data tersebut sampai ke komponen.
3. Tampilkan `token` yang diterima di konsol *jika* login berhasil.

---

### e. Polling Bersyarat dan Optimasi Bandwidth

**Tugas:** Di *Screen* **'Cart (Keranjang)'**, implementasikan fitur **polling** (GET request berulang) untuk memperbarui total belanja setiap **15 detik**. Namun, lakukan optimasi bandwidth:

1. Hentikan **Polling** sepenuhnya jika **NetInfo** mendeteksi *connection type* adalah **'cellular'** (jaringan seluler) untuk menghemat kuota pengguna.
2. Gunakan **`useEffect`** dan **`setInterval`** untuk implementasi *polling*, dan pastikan `setInterval` **dibersihkan** dengan benar di *cleanup function* saat komponen *unmount* atau *connection type* berubah.

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

---

**Mobile App Development With React Native*
