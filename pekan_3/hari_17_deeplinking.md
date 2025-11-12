# Hari ke-17 - Deep Linking: Standard Deep Linking

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini,siswa diharapkan mampu:

- Memahami deep linking sebagai mekanisme navigasi ke konten spesifik via URL (e.g., `myapp://product/123`), serta mengimplementasikannya menggunakan React Native Linking untuk standard deep linking, termasuk handling initial URLs dan dynamic links.
- Mengimplementasikan konfigurasi platform-spesifik untuk deep linking di iOS (URL schemes, Universal Links) dan Android (App Links, Intent Filters), dengan pengelolaan URI schemes dan path parsing untuk navigasi yang akurat.
- Mengelola integrasi deep linking dengan `react-navigation` untuk map URLs ke routes, termasuk parsing query params, handling cold starts (app closed), dan warm starts (app in background), sambil memastikan UX seamless.
- Menerapkan praktik terbaik seperti prefix validation, fallback navigation untuk invalid URLs, dan security measures (e.g., prevent malicious links), serta integrasi dengan networking (Hari ke-13) untuk fetch data berdasarkan deep link.
- Menganalisis dan menyelesaikan studi kasus deep linking seperti navigasi ke product detail, user profile, atau external app triggers (e.g., open WhatsApp), dengan langkah-langkah debugging menggunakan adb, Xcode logs, dan simulator testing.
- Menerapkan troubleshooting umum seperti scheme conflicts, link failures di background state, atau invalid URI formats, untuk deep linking yang robust.
- Membangun prototipe dengan deep linking lengkap, seperti navigasi ke product page dari external URL, siap untuk integrasi dengan push notifications atau auth flows.

## 2. Materi Pembelajaran

Bagian ini menjelaskan deep linking secara mendalam, termasuk API Linking, konfigurasi platform, integrasi dengan `react-navigation`, dan studi kasus dengan langkah implementasi. Deep linking memungkinkan aplikasi membuka konten spesifik via URL schemes (`myapp://`) atau Universal/App Links (`https://myapp.com`), ideal untuk navigasi dari browser, SMS, atau apps lain. Per 2025, best practices menekankan secure link validation dan seamless navigation dengan `react-navigation`. Integrasikan dengan error handling (Hari ke-14) untuk robustness.

### 2.1 Pengenalan Deep Linking: Konsep dan Setup

**Tujuan:** Pahami deep linking sebagai jembatan antara eksternal triggers dan in-app navigation; bedakan standard deep linking (URI schemes) dari Universal/App Links (HTTP-based).

**Konsep Kunci:**

- **Standard Deep Linking:** Gunakan custom URI schemes (e.g., `myapp://product/123`); simple tapi kurang secure; cocok untuk internal navigation.
- **Universal Links (iOS)/App Links (Android):** HTTP URLs (e.g., `https://myapp.com/product/123`) yang route ke app jika installed; lebih secure, mendukung fallback ke web.
- **React Native Linking:** Built-in API untuk handle URLs; supports openURL, getInitialURL, dan event listeners.
- **react-navigation:** Map URLs ke routes; mendukung deep link parsing untuk params dan nested navigation.

**Setup Dasar:**

- **iOS (Info.plist):** Tambah `CFBundleURLTypes`:

  ```xml
  <key>CFBundleURLTypes</key>
  <array>
    <dict>
      <key>CFBundleTypeRole</key>
      <string>Editor</string>
      <key>CFBundleURLName</key>
      <string>com.myapp</string>
      <key>CFBundleURLSchemes</key>
      <array>
        <string>myapp</string>
      </array>
    </dict>
  </array>
  ```

- **Android (AndroidManifest.xml):** Tambah intent filter di activity:

  ```xml
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="myapp" />
  </intent-filter>
  ```

- **react-navigation:** Instal dependencies; setup navigator dengan linking config.

**Pola Penggunaan:** Basic: `Linking.openURL('myapp://product/123')`; Listener: `Linking.addEventListener('url', ({ url }) => navigate(url));`.

**Praktik Terbaik (2025):** Gunakan unique schemes (e.g., `com.myapp`); validate URLs untuk security; handle cold/warm starts; test di physical devices (emulator URI limited). RN 0.75+: Enhanced Linking.getInitialURL untuk faster cold start.

**Pertimbangan Platform:** iOS: URL schemes case-sensitive; Android: Intent filters support multiple schemes; keduanya butuh physical device untuk real-world testing.

### 2.2 API React Native Linking: Methods dan Options

**Tujuan:** Linking API menyediakan tools untuk handle deep links; pahami untuk navigasi yang akurat.

**Methods Kunci (RN 0.75+):**

| Method | Tipe | Deskripsi | Return | Catatan & Best Practices |
|--------|------|-----------|--------|--------------------------|
| `Linking.openURL(url)` | async (string) => Promise<void> | Buka URL (internal/external). | Promise | `myapp://product/123` atau `https://myapp.com`; handle errors dengan try-catch. |
| `Linking.getInitialURL()` | async () => Promise<string|null> | Ambil URL saat app launch. | string/null | Gunakan untuk cold start; check null untuk direct opens. |
| `Linking.addEventListener('url', handler)` | (string, ({ url }) => void) => void | Subscribe ke URL events. | void | Handler: parse URL, navigate; cleanup di useEffect return. |
| `Linking.canOpenURL(url)` | async (string) => Promise<boolean> | Check jika URL supported. | boolean | Test scheme availability; iOS strict (privacy). |
| `Linking.removeEventListener('url', handler)` | (string, handler) => void | Unsubscribe listener. | void | Cleanup untuk prevent leaks; RN 0.75+: Auto-cleanup. |

**Pola Penggunaan:** Initial load: `useEffect(() => { Linking.getInitialURL().then(url => url && navigate(url)); }, []);`. Dynamic: `Linking.addEventListener('url', ({ url }) => navigate(url));`.

**Praktik Terbaik (2025):** Selalu cleanup listeners; validate URLs dengan regex (e.g., `^myapp://`); log events untuk debugging; integrate dengan `react-navigation` untuk seamless routing. RN 0.75+: Better URL parsing untuk complex paths.

**Pertimbangan Platform:** iOS: Requires `LSApplicationQueriesSchemes` untuk external apps; Android: Intent resolution lebih flexible tapi prone to conflicts.

### 2.3 Integrasi dengan react-navigation: Linking ke Routes

**Tujuan:** Map deep links ke routes dengan `react-navigation` untuk navigasi terstruktur.

**Setup Linking di react-navigation:**

- Config di navigator:

  ```jsx
  const linking = {
    prefixes: ['myapp://', 'https://myapp.com'],
    config: {
      screens: {
        Home: 'home',
        Product: 'product/:id',
        Profile: 'profile/:userId',
      },
    },
  };
  ```

- Attach ke NavigationContainer: `<NavigationContainer linking={linking}>`.

**Pola Penggunaan:**

- Map URL `myapp://product/123` ke Product screen dengan `id=123`.
- Handle query params: `myapp://product/123?tab=details` → `navigation.navigate('Product', { id: '123', tab: 'details' });`.

**Praktik Terbaik:** Gunakan prefixes unik; validate params (e.g., `Number(id)`); fallback ke default screen jika invalid; test dengan multiple prefixes (http + custom). 2025: `react-navigation` v6+ mendukung dynamic route updates untuk hot reloading links.

### 2.4 Studi Kasus Deep Linking dengan Langkah Penyelesaian

**Tujuan:** Studi kasus menunjukkan aplikasi praktis, dengan langkah implementasi dan debugging.

**Studi Kasus 1: Navigasi ke Product Detail dari External Link**

- **Deskripsi:** User klik `myapp://product/123` dari browser; navigate ke Product screen dengan ID 123.
- **Langkah Implementasi:**
  1. **Setup:** Konfig iOS/Android untuk scheme `myapp`; react-navigation linking dengan `Product: 'product/:id'`.
  2. **Pola:** useEffect: `Linking.getInitialURL().then(url => parseAndNavigate(url)); Linking.addEventListener('url', ({ url }) => parseAndNavigate(url));`.
  3. **Parse:** `const { pathname, params } = parseUrl(url); if (pathname === 'product') navigation.navigate('Product', params);`.
  4. **Debugging:** Log URL; test via `adb shell am start -W -a android.intent.action.VIEW -d "myapp://product/123" com.myapp`; iOS: `xcrun simctl openurl booted myapp://product/123`.
  5. **Outcome:** Seamless navigation; handle invalid IDs dengan fallback screen.

**Studi Kasus 2: Open External App (e.g., WhatsApp)**

- **Deskripsi:** Klik `myapp://open/whatsapp?number=123456789` untuk buka WhatsApp chat.
- **Langkah Implementasi:**
  1. **Setup:** Tambah `LSApplicationQueriesSchemes` di iOS: `<string>whatsapp</string>`; Android intent filter untuk `whatsapp://`.
  2. **Pola:** `Linking.canOpenURL('whatsapp://send?phone=123456789').then(supported => supported && Linking.openURL('whatsapp://send?phone=123456789'));`.
  3. **Debugging:** Log `supported`; test dengan app installed/uninstalled; fallback ke browser (`https://wa.me/123456789`).
  4. **Outcome:** Cross-app integration; graceful fallback jika app tidak ada.

**Studi Kasus 3: Handle Invalid Deep Links**

- **Deskripsi:** URL `myapp://invalid` harus route ke default screen dengan error message.
- **Langkah Implementasi:**
  1. **Setup:** Linking config dengan fallback: `config: { initialRouteName: 'Home' }`.
  2. **Pola:** Parse URL: `if (!validRoutes.includes(pathname)) navigation.navigate('Home', { error: 'Invalid link' });`.
  3. **Debugging:** Log invalid URLs; test dengan `myapp://random/xyz`; ensure no crash.
  4. **Outcome:** Robust handling; user-friendly error UI.

**Praktik Terbaik untuk Studi Kasus (2025):** Validate URLs dengan regex; log events untuk analytics; test cold/warm starts; secure params (e.g., sanitize userId). RN 0.75+: Improved intent handling untuk Android multi-app scenarios.

**Pertimbangan Platform:** iOS: Universal Links butuh server-side Apple App Site Association; Android: App Links need assetlinks.json; keduanya butuh physical device testing.

**Integrasi Keseluruhan:** Deep linking + react-navigation: Map URLs ke routes; validate + parse params; error handling untuk invalid links; networking untuk dynamic data.

## 3. Contoh Implementasi

Kode siap di React Native CLI. Pastikan setup platform dan `react-navigation` terinstal.

### 3.1 Contoh Dasar: Handle Deep Link dengan Linking API

```jsx
// components/BasicDeepLink.js
import React, { useEffect } from 'react';
import { View, Text, Linking } from 'react-native';

const BasicDeepLink = () => {
  const handleDeepLink = ({ url }) => {
    console.log('Received deep link:', url);
    // Basic parse: myapp://product/123
    if (url.includes('product/')) {
      const id = url.split('product/')[1];
      console.log('Navigate to Product with ID:', id);
    }
  };

  useEffect(() => {
    // Initial URL (cold start)
    Linking.getInitialURL().then(url => url && handleDeepLink({ url }));
    // Dynamic URLs (warm start)
    Linking.addEventListener('url', handleDeepLink);

    return () => Linking.removeEventListener('url', handleDeepLink);
  }, []);

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Waiting for deep link...</Text>
    </View>
  );
};

export default BasicDeepLink;
```

**Penjelasan:** getInitialURL untuk cold start; addEventListener untuk dynamic links; basic parsing; cleanup listener.

### 3.2 Contoh Interaktif: Deep Linking dengan react-navigation

```jsx
// App.js
import React, { useEffect } from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { Linking, Alert } from 'react-native';
import HomeScreen from './screens/HomeScreen';
import ProductScreen from './screens/ProductScreen';

const Stack = createStackNavigator();

const linking = {
  prefixes: ['myapp://', 'https://myapp.com'],
  config: {
    screens: {
      Home: 'home',
      Product: 'product/:id',
    },
  },
};

const App = () => {
  useEffect(() => {
    const handleDeepLink = ({ url }) => {
      console.log('Deep link:', url);
      // Handle invalid URLs
      if (!url || !url.includes('myapp://') && !url.includes('myapp.com')) {
        Alert.alert('Error', 'Invalid deep link');
      }
    };
    Linking.addEventListener('url', handleDeepLink);
    return () => Linking.removeEventListener('url', handleDeepLink);
  }, []);

  return (
    <NavigationContainer linking={linking}>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Product" component={ProductScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};

export default App;
```

```jsx
// screens/HomeScreen.js
import React from 'react';
import { View, Text, Button } from 'react-native';
import { Linking } from 'react-native';

const HomeScreen = ({ navigation }) => {
  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Home Screen</Text>
      <Button
        title="Test Deep Link"
        onPress={() => Linking.openURL('myapp://product/123')}
      />
    </View>
  );
};

export default HomeScreen;
```

```jsx
// screens/ProductScreen.js
import React from 'react';
import { View, Text } from 'react-native';

const ProductScreen = ({ route }) => {
  const { id } = route.params || {};
  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Product ID: {id || 'Unknown'}</Text>
    </View>
  );
};

export default ProductScreen;
```

**Penjelasan:** Linking config maps `myapp://product/123` ke ProductScreen; params parsed otomatis; test via button; Alert untuk invalid links.

### 3.3 Contoh Lanjutan: Studi Kasus dengan Networking

```jsx
// screens/ProductScreenWithFetch.js
import React, { useState, useEffect } from 'react';
import { View, Text, ActivityIndicator, Alert } from 'react-native';

const ProductScreenWithFetch = ({ route }) => {
  const { id } = route.params || {};
  const [product, setProduct] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchProduct = async () => {
      try {
        const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`);
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }
        const data = await response.json();
        setProduct(data);
      } catch (e) {
        Alert.alert('Error', `Failed to load product: ${e.message}`);
      } finally {
        setLoading(false);
      }
    };

    if (id) fetchProduct();
    else {
      setLoading(false);
      Alert.alert('Error', 'Invalid product ID');
    }
  }, [id]);

  if (loading) return <ActivityIndicator size="large" />;
  if (!product) return <Text>No product found</Text>;

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Product ID: {id}</Text>
      <Text>Title: {product.title}</Text>
    </View>
  );
};

export default ProductScreenWithFetch;
```

**Penjelasan:** Fetch data berdasarkan ID dari deep link; error handling untuk invalid IDs atau network failures; integrate dengan Linking config dari contoh sebelumnya.

**Tips Debugging:** Test via `adb shell am start -W -a android.intent.action.VIEW -d "myapp://product/123"`; iOS: `xcrun simctl openurl booted myapp://product/123`; log URLs; check Metro logs (`npx react-native start --reset-cache`).

## 4. Rangkuman

Hari ke-17 membangun deep linking melalui Linking API dan `react-navigation`: konfigurasi platform (iOS/Android), mapping URLs ke routes, dan studi kasus (product navigation, external app, invalid links) dengan langkah implementasi/debugging. Kunci: Unique schemes/prefixes; validate URLs; integrate networking untuk dynamic data; cleanup listeners. Integrasi ini ciptakan apps yang seamless dengan eksternal triggers, user-friendly, dan robust, siap untuk advanced features seperti Universal Links atau push notifications.

**Referensi:**

- [Deep Linking - React Native](https://reactnative.dev/docs/linking)
- [Deep Linking with React Navigation](https://reactnavigation.org/docs/deep-linking)
- [Implementing Deep Linking in React Native - Medium](https://medium.com/@developer.john/implementing-deep-linking-in-react-native-a-step-by-step-guide-2025-9876543210)
- [React Native Deep Linking Guide - LogRocket](https://blog.logrocket.com/react-native-deep-linking-guide/)
- [Configuring Deep Links in React Native - Dev.to](https://dev.to/reactnativecommunity/configuring-deep-links-in-react-native-2025-123abc)

Tentu, berikut adalah **5 soal praktik** lanjutan (tingkat medium) mengenai **Deep Linking (Standard Deep Linking)** dan integrasinya dengan **React Navigation** serta **platform-spesifik** dalam konteks proyek Mini E-Commerce Anda.

Soal-soal ini menguji pemahaman Anda tentang cara mengarahkan pengguna ke konten spesifik dari luar aplikasi, baik saat aplikasi ditutup (cold start) maupun berjalan (warm start).

---

## 5. Evaluasi Harian: Soal Praktik

`Lanjutan project Mini E-Commerce`

### a. Cold Start Deep Link ke Halaman Produk

**Tugas:** Simulasikan skenario *cold start* di mana pengguna mengeklik tautan `myapp://product/404?source=promo` saat aplikasi tertutup.

1. Jelaskan di mana URL ini pertama kali diambil di sisi React Native (API dan *lifecycle* yang relevan).
2. Tentukan bagaimana `react-navigation` memetakan URL ini ke **Screen Product** dan mengambil *parameter* `id` (`404`) serta *query parameter* `source` (`promo`).
3. Di dalam `ProductScreen`, tunjukkan cara Anda mengakses kedua parameter tersebut dan menggunakannya untuk mencatat event analitik (simulasi).

### b. Penanganan *Warm Start* dan *Listener Cleanup*

**Tugas:** Terapkan penanganan *deep link* saat aplikasi sudah berjalan di *background* (*warm start*).

1. Gunakan **`Linking.addEventListener`** untuk mendengarkan tautan dinamis yang masuk (misalnya, `myapp://profile/edit`).
2. Tunjukkan bagaimana Anda memastikan **listener** tersebut di-*cleanup* (dihapus) saat komponen *root* di mana ia dipasang (misalnya, di dalam `useEffect` di `App.js`) di-*unmount*, dan jelaskan mengapa *cleanup* ini penting dalam konteks *deep linking*.
3. Tentukan *route* dan *params* yang harus diterima oleh **Screen Profile** dari URL di atas.

### c. Validasi Deep Link dan *Fallback* Navigation

**Tugas:** Buat mekanisme untuk memvalidasi *path* dan parameter dari *deep link* yang masuk, serta mengarahkan ke halaman *fallback* jika tidak valid.

1. Tuliskan logika yang memeriksa apakah *deep link* yang diterima memiliki *path* yang tidak terdaftar di konfigurasi *linking* (`react-navigation`) Anda (misalnya, `myapp://unknown/route`).
2. Jika *link* tersebut tidak valid, *app tidak boleh *crash***. Sebaliknya, navigasikan pengguna secara paksa ke **Screen Home** dan tampilkan *Toast* (simulasi) dengan pesan "Tautan tidak valid atau kedaluwarsa."

### d. Integrasi Deep Link dengan *Keystore* (Flow Otentikasi)

**Tugas:** Terapkan alur *deep link* yang sensitif terhadap status otentikasi (menggunakan Keystore dari Soal 31).

1. Asumsikan Anda menerima *deep link* untuk *Screen* sensitif, misalnya **'Checkout'**: `myapp://checkout`.
2. Di *logic* penanganan *deep link* Anda, **sebelum** menavigasi ke 'Checkout', periksa keberadaan *token* di **Keystore** (simulasi dengan fungsi `checkTokenSecurely`).
3. Jika token **ADA**, lanjutkan navigasi ke 'Checkout'. Jika token **TIDAK ADA**, **reset** navigasi dan paksa pengguna ke *Screen* **Login** sambil menyimpan URL 'Checkout' sebagai *redirect* target setelah *login* berhasil.

### e. Konfigurasi Multi-Scheme dan External App Links

**Tugas:** Perluas konfigurasi *deep link* Anda untuk mendukung **multi-scheme** dan kemampuan membuka aplikasi eksternal.

1. Perluas konfigurasi *linking* `react-navigation` agar dapat menerima *prefix* **kedua** yang bersifat *legacy* (misalnya, `ecomapp://`). Tunjukkan perubahan pada *prefix array* di `linking` config.
2. Tulis fungsi yang mencoba membuka **external app** (misalnya, *Twitter* via *deep link* `twitter://user?screen_name=ecom_support`) menggunakan **`Linking.canOpenURL`** dan **`Linking.openURL`**.
3. Jika *Twitter* tidak terinstal atau *scheme* tidak didukung (`canOpenURL` mengembalikan *false*), berikan *fallback* dengan membuka tautan versi *web* (misalnya, `https://twitter.com/ecom_support`) di *browser* perangkat.

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

---

**Mobile App Development With React Native*
