# Hari ke-14 - Error Handling

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

- Memahami konsep error handling di React Native, termasuk perbedaan synchronous/asynchronous errors, dan menerapkan try-catch untuk async operations serta Error Boundaries untuk component crashes, memastikan app tetap stabil tanpa full reload.
- Mengimplementasikan pola error handling lokal (component-level) dengan state management (e.g., useState untuk error flags) dan global (e.g., ErrorBoundary wrapper), termasuk logging dengan console.error/LogBox dan user feedback via Alert/Toast untuk UX yang empati.
- Mengelola error handling khusus networking dari Hari ke-13, seperti HTTP status codes (4xx/5xx), timeouts, dan connectivity issues, dengan strategi retry, fallback data, dan graceful degradation untuk aplikasi yang resilient di jaringan tidak stabil.
- Menerapkan praktik terbaik seperti error categorization (network vs user vs internal), centralized logging (e.g., console.error dengan stack traces).
- Menganalisis dan menyelesaikan studi kasus error networking umum seperti CORS violations, SSL handshake failures, atau infinite retry loops, dengan langkah-langkah debugging menggunakan Chrome DevTools atau adb logcat.
- Menerapkan troubleshooting umum seperti silent failures di async code atau memory leaks dari unhandled rejections, untuk error handling yang scalable.
- Membangun prototipe dengan error handling lengkap, seperti networking component dengan fallback UI dan retry logic, siap untuk integrasi dengan state management lanjutan.

## 2. Materi Pembelajaran

Bagian ini menjelaskan error handling secara mendalam, mulai dari fondasi hingga aplikasi praktis, dengan fokus pada networking dari Hari ke-13. Error handling di RN melibatkan JS runtime errors (e.g., undefined props) dan native bridge errors (e.g., network timeouts), ditangani dengan try-catch untuk sync/async dan ErrorBoundary untuk React-specific crashes. Per 2025, best practices menekankan "progressive enhancement" (show partial UI saat error) dan logging terstruktur untuk debugging. Sertakan studi kasus networking di sub-bagian khusus, dengan langkah penyelesaian.

### A. Fondasi Error Handling: Jenis Error dan Pola Dasar

**Tujuan:** Pahami jenis error di RN untuk strategi tepat; RN errors termasuk JS (syntax/reference), React (prop validation), native (permissions/network), dan async (promises/network).

**Jenis Error Kunci:**

| Jenis Error | Deskripsi | Contoh | Pola Handling Dasar |
|-------------|-----------|--------|---------------------|
| **Synchronous** | Terjadi saat eksekusi langsung (e.g., undefined variable). | `undefinedVar.propName` → TypeError. | Try-catch block: `try { ... } catch (e) { console.error(e); }`. |
| **Asynchronous** | Di promises/async functions (e.g., fetch fails). | `await fetch(url)` → NetworkError. | Async try-catch: `try { const res = await fetch(url); } catch (e) { setError(e); }`; .catch() pada promises. |
| **React-Specific** | Component lifecycle errors (e.g., invalid prop). | <Component invalidProp /> → PropTypes warning. | ErrorBoundary: Wrapper component catch render errors; useImperativeHandle untuk refs. |
| **Networking** | HTTP/network failures (detail di 2.3). | Timeout di fetch → AbortError. | Conditional requests + fallback; integrasi NetInfo dari Hari ke-13. |
| **Native Bridge** | RN-specific (e.g., permission denied). | PermissionsAndroid.request() fails → Permission error. | Platform-specific checks; Alert untuk user prompts. |

**Pola Penggunaan:** Lokal: useState untuk error state di component. Global: ErrorBoundary di root; unhandledrejection listener di index.js: `window.addEventListener('unhandledrejection', e => { e.preventDefault(); console.error(e.reason); });`.

**Praktik Terbaik (2025):** Selalu log dengan stack trace (console.error(e, e.stack)); kategori errors (e.g., { type: 'NETWORK', message: e.message }); user-friendly messages (e.g., 'Something went wrong' vs raw error). RN 0.75+: LogBox.ignoreLogs(['Warning:...']) untuk suppress noise.

### B. Error Boundaries: Tangkap React Component Errors

**Tujuan:** ErrorBoundary adalah class component yang catch JS errors di render phase (tidak cover event handlers/async), mencegah app crash dengan fallback UI.

**Implementasi Kunci:**

- Extend React.Component; static getDerivedStateFromError() untuk update state; componentDidCatch() untuk logging.
- Props: FallbackComponent, error, errorInfo untuk pass ke child.

**Pola Penggunaan:** Root wrapper: <ErrorBoundary><App /></ErrorBoundary>; per-component: <ErrorBoundary fallback={<Text>Error in Widget</Text>}><Widget /></ErrorBoundary>.

**Praktik Terbaik (2025):** Gunakan di root + key areas (e.g., async data fetches); log ke console.error; resetErrorBoundary via key prop. RN: Hooks-based boundaries dengan unstable_ErrorBoundary API (eksperimental).

### C. Error Handling di Networking: Studi Kasus dan Penyelesaian

**Tujuan:** Fokus pada networking errors dari Hari ke-13, dengan studi kasus umum (HTTP 4xx/5xx, timeouts, CORS, connectivity drops) dan langkah penyelesaian langkah-demi-langkah untuk debugging dan recovery.

**Studi Kasus 1: HTTP Status Errors (4xx Client/5xx Server)**

- **Deskripsi:** Server reject request (e.g., 401 Unauthorized, 404 Not Found, 500 Internal Server).
- **Gejala:** Response.ok false; Axios: err.response.status.
- **Langkah Penyelesaian:**
  1. **Diagnose:** Log response.status + body: `if (!res.ok) { console.error(`HTTP ${res.status}:`, await res.text()); }`.
  2. **Handle:** Switch (status): 401 → redirect login (`navigation.replace('Login')`); 404 → show "Not found" UI; 5xx → retry after delay.
  3. **Prevent:** Validate inputs pre-request (e.g., token validity); use optimistic updates (assume success, rollback on error).
  4. **Test:** Mock dengan nock: `nock('api').get('/users').reply(404, { error: 'Not found' });`.

**Studi Kasus 2: Timeouts dan Request Hanging**

- **Deskripsi:** Request tak selesai (e.g., slow server, network congestion); AbortError setelah timeout.
- **Gejala:** Request stuck; no response after 10s+.
- **Langkah Penyelesaian:**
  1. **Diagnose:** Monitor dengan AbortController: `const controller = new AbortController(); setTimeout(() => controller.abort(), 10000);` + log 'Timeout' on AbortError.
  2. **Handle:** Show loading spinner + timeout message; fallback to static data.
  3. **Prevent:** Set reasonable timeouts (5-15s); parallel requests untuk non-critical data; use Axios retry interceptor: `interceptors.response.use(undefined, err => { if (err.code === 'ECONNABORTED') retry(); });`.
  4. **Test:** Throttle emulator network (Chrome DevTools > Network > Throttling: Slow 3G); measure dengan console.time().

**Studi Kasus 3: CORS Violations**

- **Deskripsi:** Browser/server block cross-origin requests (e.g., dev server tanpa CORS headers).
- **Gejala:** "Access-Control-Allow-Origin" error di console; request fails preflight.
- **Langkah Penyelesaian:**
  1. **Diagnose:** Check console: `CORS policy: No 'Access-Control-Allow-Origin' header`; test dengan curl: `curl -H "Origin: http://localhost:8081" -v https://api.example.com`.
  2. **Handle:** Fallback to proxy (e.g., dev server via `npm start -- --port 8081`); show "Server unavailable" UI.
  3. **Prevent:** Production: Server-side CORS (* or specific origins); dev: Use proxy in Metro config: `devServer: { proxy: { '/api': 'https://api.example.com' } }`.
  4. **Test:** Run in release mode (`npx react-native run-android --variant=release`); verify with Postman.

**Studi Kasus 4: Connectivity Drops (No Internet)**

- **Deskripsi:** Request gagal karena offline (integrasi NetInfo dari Hari ke-13).
- **Gejala:** TypeError: Failed to fetch; NetInfo.isConnected false.
- **Langkah Penyelesaian:**
  1. **Diagnose:** Gunakan NetInfo: `NetInfo.fetch().then(state => console.log(state.isInternetReachable ? 'Online' : 'Offline'));`.
  2. **Handle:** Pause requests; show offline banner; queue dengan setTimeout retry on reconnect listener.
  3. **Prevent:** Pre-check: if (!isOnline) return fallback; use service workers (opsional via PWA mode).
  4. **Test:** Toggle airplane mode; simulate di emulator (Settings > Network > Flight mode).

**Praktik Terbaik untuk Networking Errors (2025):** Centralized handler via Axios interceptors; categorize (network/server/client) untuk logging; user-centric messages (e.g., "Connection lost—retry?"); retry dengan exponential backoff (1s, 2s, 4s). RN 0.75+: Enhanced Fetch error objects dengan network state.

### D. Praktik Terbaik Global dan Troubleshooting

**Tujuan:** Strategi lintas-domain untuk error handling yang konsisten.

- **Global:** ErrorBoundary di root; unhandledrejection listener; integrate console.error untuk crash reports.
- **User Feedback:** Alert for critical; Toast (react-native-toast-message) for non-blocking; loading spinners dengan text (e.g., "Loading..." → "Still loading...").
- **Testing:** Jest: `jest.mock('react-native', () => ({ fetch: () => Promise.reject(new Error('Mock error')) }));`; snapshot error states.
- **Troubleshooting Umum:** Silent async errors → add .catch(console.error) di semua promises; RN bridge errors → check Metro logs (`npx react-native start --reset-cache`).

**Pertimbangan Platform:** iOS: Native crash reports via Xcode; Android: adb logcat untuk uncaught exceptions. 2025: Console enhancements untuk RN CLI debugging.

**Integrasi Keseluruhan:** Error handling + networking: Try-catch di fetch/useEffect; NetInfo gate + fallback UI; ErrorBoundary wrap components.

## 3. Contoh Implementasi

Kode siap di React Native CLI. Gunakan useState/useEffect untuk state; integrasikan NetInfo dari Hari ke-13.

### A. Contoh Dasar: Error Handling Lokal dengan Try-Catch di Fetch

```jsx
// components/UserListWithError.js
import React, { useState, useEffect } from 'react';
import { View, Text, FlatList, Alert, ActivityIndicator } from 'react-native';

const UserListWithError = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    const fetchUsers = async () => {
      try {
        const response = await fetch('https://jsonplaceholder.typicode.com/users', {
          signal: controller.signal,
          headers: { 'Accept': 'application/json' },
        });
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
        const data = await response.json();
        setUsers(data);
      } catch (err) {
        if (err.name === 'AbortError') {
          console.log('Request aborted');
          return;
        }
        setError(err.message);
        Alert.alert('Network Error', err.message || 'Failed to fetch data');
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();

    return () => controller.abort();
  }, []);

  if (loading) return <ActivityIndicator size="large" />;
  if (error) return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Error: {error}</Text>
    </View>
  );

  return (
    <FlatList
      data={users}
      keyExtractor={item => item.id.toString()}
      renderItem={({ item }) => <Text>{item.name}</Text>}
    />
  );
};

export default UserListWithError;
```

**Penjelasan:** Try-catch async di useEffect; check response.ok; Alert untuk user; finally untuk loading; AbortController cleanup.

### B. Contoh Interaktif: ErrorBoundary untuk Component-Level Errors

```jsx
// components/ErrorBoundary.js
import React from 'react';
import { View, Text, Button } from 'react-native';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('ErrorBoundary caught:', error, errorInfo);
    // Opsional: Send to logging service
  }

  render() {
    if (this.state.hasError) {
      return (
        <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
          <Text>Something went wrong: {this.state.error?.message}</Text>
          <Button title="Retry" onPress={() => this.setState({ hasError: false, error: null })} />
        </View>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

**Integrasi:**

```jsx
// App.js
import React from 'react';
import ErrorBoundary from './components/ErrorBoundary';
import UserListWithError from './components/UserListWithError';

const App = () => (
  <ErrorBoundary>
    <UserListWithError />
  </ErrorBoundary>
);
```

**Penjelasan:** Class component catch render errors; fallback UI + retry; log di componentDidCatch.

### C. Contoh Lanjutan: Studi Kasus Networking Errors dengan Penyelesaian

Gunakan contoh di atas sebagai base; tambah handling untuk studi kasus.

```jsx
// components/NetworkErrorHandler.js
import React, { useState, useEffect } from 'react';
import { View, Text, Button, Alert } from 'react-native';
import { NetInfo } from '@react-native-community/netinfo'; // Dari Hari ke-13

const NetworkErrorHandler = () => {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [retryCount, setRetryCount] = useState(0);

  const fetchData = async () => {
    const controller = new AbortController();
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/posts/1', {
        signal: controller.signal,
        headers: { 'Accept': 'application/json' },
      });
      if (!response.ok) {
        if (response.status === 404) {
          throw new Error('Resource not found - check URL or ID');
        } else if (response.status >= 500) {
          throw new Error('Server error - please try again later');
        }
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      const result = await response.json();
      setData(result);
      setError(null);
    } catch (err) {
      if (err.name === 'AbortError') {
        setError('Request timed out - network slow');
      } else {
        setError(err.message);
      }
      Alert.alert('Error', err.message, [{ text: 'Retry', onPress: () => fetchData() }]);
    } finally {
      controller.abort();
    }
  };

  useEffect(() => {
    fetchData();
  }, []);

  const handleRetry = () => {
    setRetryCount(prev => prev + 1);
    if (retryCount < 3) {
      setTimeout(fetchData, 1000 * retryCount); // Exponential backoff
    }
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      {data ? <Text>{data.title}</Text> : <Text>{error || 'Loading...'}</Text>}
      {error && <Button title={`Retry (${retryCount}/3)`} onPress={handleRetry} />}
    </View>
  );
};

export default NetworkErrorHandler;
```

**Penjelasan:** Handle 404/5xx dengan specific messages; AbortError untuk timeout; retry dengan backoff; Alert + button untuk user interaction.

**Tips Debugging:** Gunakan Chrome DevTools Network tab; test errors dengan browser throttling atau mock servers (npx http-server --cors).

## 4. Rangkuman

Hari ke-14 membangun error handling robust melalui pola dasar (try-catch, ErrorBoundary), aplikasi di networking (status checks, specific handling), dan studi kasus (HTTP errors, timeouts, CORS, connectivity) dengan langkah diagnosis/prevent/test. Kunci: Categorize errors untuk targeted fixes; fallback UI + retry untuk UX; cleanup dengan AbortController. Integrasi ini ciptakan apps yang fail gracefully, user-friendly, dan debuggable, siap untuk caching/logging lanjutan.

**Latihan Selanjutnya:** Extend NetworkErrorHandler dengan NetInfo integration; tambah ErrorBoundary di app root.

**Referensi:**

- [Error Handling - React Native](https://reactnative.dev/docs/error-handling)
- [Error Boundaries - React](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)
- [Handling Errors in React Native Apps - LogRocket](https://blog.logrocket.com/handling-errors-react-native-apps/)
- [Best Practices for Error Handling in React Native - Medium](https://medium.com/@shubhamkashyap/best-practices-for-error-handling-in-react-native-2025-edition-1234567890)

## 5. Evaluasi Harian: Soal Praktik

`Lanjutan project Mini E-Commerce`

Soal-soal ini berfokus pada ketahanan aplikasi (*resilience*) dan pengalaman pengguna (*UX*) yang empatik saat terjadi kegagalan.

---

### a. Implementasi *Error Boundary* Global

**Tugas:** Terapkan **Error Boundary** sebagai *wrapper* di tingkat **Root Navigasi** aplikasi Anda (di sekitar `NavigationContainer`). Jika terjadi kegagalan rendering (misalnya, kesalahan sintaks yang tidak terduga pada salah satu *Screen* atau *Widget*), *aplikasi tidak boleh *crash***. Sebaliknya, tampilkan **fallback UI** global yang ramah pengguna, berisikan:

1. Pesan umum: "Aplikasi mengalami masalah tak terduga."
2. Tombol **"Mulai Ulang Aplikasi"** yang berfungsi me-*reset* seluruh *state* (misalnya, menggunakan *key* prop pada *ErrorBoundary* atau *reload* sederhana).
3. Catat error dan *component stack trace* ke **konsol** menggunakan `componentDidCatch`.

---

### b. *Graceful Degradation* untuk Gagal Ambil Data

**Tugas:** Modifikasi *Screen* **'Product Detail'**. Ketika *request* GET data produk (menggunakan *fetch* atau *Axios*) gagal karena *HTTP Status Code* **404 (Not Found)** atau **500 (Internal Server Error)**:

1. **Hentikan** loading spinner.
2. Tampilkan **fallback data** lokal (simulasikan objek produk sederhana dengan gambar dan nama statis).
3. Berikan pesan Toast (notifikasi non-blocking) kepada pengguna: "Gagal memuat data terbaru. Menampilkan versi arsip."
4. Pastikan *status code* error (404 atau 500) dicatat secara terpisah di konsol.

---

### c. Logika *Retry* dengan *Exponential Backoff*

**Tugas:** Terapkan logika **Retry Otomatis** pada *request* GET untuk daftar produk di *Screen* **'Product List'**. Jika *request* gagal (karena *timeout* atau error jaringan lainnya):

1. Lakukan **maksimal 3 kali** percobaan *retry* secara otomatis.
2. Gunakan pola **Exponential Backoff** untuk interval *retry*: *Retry* pertama setelah 1 detik, *retry* kedua setelah 2 detik, dan *retry* ketiga setelah 4 detik.
3. Jika 3 *retry* gagal, tampilkan UI error permanen dengan tombol **"Coba Lagi Manual"**.

---

### d. Error Handling dan *Feedback* Pengguna pada Formulir POST

**Tugas:** Pada *Screen* **'Checkout'** (saat mengirim data POST ke *endpoint* simulasi):

1. Gunakan **Axios Interceptor Response** untuk mendeteksi *HTTP Status Code* **400 (Bad Request)** yang mengindikasikan validasi input gagal.
2. Jika 400 terdeteksi, **tangkap** pesan error dari *response body* server (simulasikan *response* seperti `{ errors: { field: 'Alamat wajib diisi' } }`).
3. Jangan menggunakan *Toast* atau *Alert*. Sebaliknya, tampilkan pesan error ini **langsung di bawah *field* formulir** yang relevan (misalnya, di bawah input alamat).

---

### e. Sinkronisasi Status Koneksi dengan UI Global

**Tugas:** Integrasikan **NetInfo** dengan mekanisme *Error State* Global di aplikasi E-Commerce Anda.

1. Buat *state* global (misalnya di *Context* atau *Global Hook*) yang mencerminkan status **ketersediaan internet** (`isInternetReachable`).
2. Jika koneksi **hilang**, seluruh layar (kecuali *Error Boundary* dari Soal 21) harus menampilkan **Banner/Toast persisten** di bagian atas layar dengan pesan: "Koneksi terputus. Menggunakan mode *offline*."
3. Saat koneksi **kembali**, *banner* tersebut harus **otomatis hilang**, dan log ke konsol: "Koneksi pulih. Melanjutkan operasi."

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

---

**Mobile App Development With React Native*
