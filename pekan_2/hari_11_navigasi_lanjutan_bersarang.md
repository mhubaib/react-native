# Hari ke-11: Navigasi Lanjutan & Arsitektur Navigasi Bersarang

## 1\. Tujuan Pembelajaran

Setelah menyelesaikan sesi pembelajaran ini, peserta diharapkan mampu:

* **Memahami Konsep Dasar:** Menguasai konsep arsitektur navigasi bersarang (*nested navigation*), termasuk hierarki navigator, pengelolaan *state* yang independen per tingkat, dan mekanisme *event bubbling* untuk aksi yang gagal (misalnya, `Maps` ke rute yang tidak dikenal di *child navigator*).
* **Menerapkan Pola Bersarang:** Mengimplementasikan pola *nested navigation* umum, seperti **Tabs di dalam Stack** (untuk detail seperti modal), **Drawer di dalam Stack** (untuk *sidebar* dengan alur *drill-down*), dan **Stack di dalam Tabs/Drawer** (untuk navigasi yang persisten dengan sub-alur).
* **Mengintegrasikan Fitur Lanjutan:** Mengintegrasikan fitur-fitur lanjutan seperti pengiriman parameter lintas tingkat (*cross-level param passing*) dan implementasi aksi navigasi khusus (*custom actions*) menggunakan *hooks* seperti `useNavigation` dan metode `navigation.dispatch`.
* **Mengoptimalkan dan Menerapkan *Best Practices*:** Menerapkan praktik terbaik seperti meminimalkan kedalaman bersarang (maksimal 2-3 tingkat), menggunakan *unique screen IDs* untuk penargetan peristiwa, dan mengoptimalkan performa dengan properti seperti `detachInactiveScreens` dan `freezeOnBlur`.
* **Melakukan *Troubleshooting*:** Mengelola isu umum seperti munculnya *header* ganda (*multiple headers*), konflik navigasi, atau kesulitan mengakses parameter dengan teknik seperti `getParent()` untuk mengakses *parent navigator*.

-----

## 2\. Materi Pembelajaran

Materi hari ini membahas secara mendalam mengenai navigasi lanjutan dan arsitektur bersarang. Fokus utamanya adalah bagaimana beberapa *navigator* dapat saling bertumpuk (*nesting*) untuk membentuk struktur pohon (*tree structure*) alur aplikasi.

Setiap *navigator* mempertahankan *state* independen (riwayat, parameter, dan opsi) di levelnya sendiri. Aksi navigasi (seperti `Maps`) akan diselesaikan di *navigator* paling dalam (*deepest navigator*); jika gagal, aksi tersebut akan **"menggelembung" (*bubble up*)** ke *navigator* induk (*parent*) berikutnya hingga berhasil atau mencapai *root navigator*.

**Pembaruan (Per 2025):** *React Navigation* sangat merekomendasikan penggunaan **Groups** (pembungkus non-navigator) untuk tujuan pengorganisasian kode tanpa menambah *overhead* *state* navigasi yang tidak perlu, sehingga membantu "meratakan" (*flatten*) hierarki.

### 2.1. Instalasi dan Setup

Instalasi komponen yang diperlukan melalui *Command Line Interface (CLI)*:

```bash
npm install @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs @react-navigation/drawer @react-navigation/material-top-tabs react-native-screens react-native-safe-area-context react-native-gesture-handler
```

**Catatan:** Pastikan Anda juga telah mengimpor `react-native-gesture-handler` di file `index.js` atau `App.js` sebagai baris pertama, dan jika menggunakan *Native Stack*, pastikan *package* `react-native-screens` telah diaktifkan dengan menambahkan `enableScreens()` di awal aplikasi.

### 2.2. Konsep Arsitektur Navigasi Bersarang: Hierarki dan Pengelolaan State

Tujuan arsitektur bersarang adalah untuk memungkinkan komposisi alur yang rumit. Pahami konsep-konsep ini untuk menghindari jebakan seperti duplikasi UI atau aksi navigasi yang terperangkap (*stuck actions*).

#### Konsep Kunci

* **Hierarki Navigator:** Sebuah *child navigator* dirender sebagai `component` dari sebuah `<Screen>` di *parent navigator*. Contoh: `<Stack.Screen name="Tabs" component={TabNavigator} />`. *Child* memiliki tumpukan/alur sendiri, tetapi dapat mewarisi tema atau opsi dari *parent*.
* **Independensi *State*:** Setiap level dalam hierarki memiliki *state* navigasi yang terpisah. Misalnya, riwayat tumpukan di *child stack* tidak memengaruhi *state* pada *parent tabs*.
* **Mekanisme *Event Bubbling*:** Aksi navigasi (seperti `Maps('Detail')`) pertama kali dicoba oleh *navigator* yang sedang fokus. Jika `Detail` bukan rute yang valid di *navigator* tersebut, aksi akan **menggelembung** (*bubble up*) ke *parent navigator* berikutnya untuk mencoba resolusi, dan seterusnya.
* **Akses Lintas Level:** Untuk memicu aksi pada *parent navigator* dari *child screen*, gunakan `navigation.getParent('ParentId')`.

### 2.3. Pola Umum Navigasi Bersarang

| Pola | Deskripsi | Skenario Penggunaan | Konfigurasi Kunci |
| :--- | :--- | :--- | :--- |
| **Tabs di dalam Stack** | *Tab Bar* berada di dalam *Stack*. Saat menavigasi ke halaman detail (*push*), *Tab Bar* akan hilang. | Digunakan untuk alur detail seperti modal. Contoh: Dari *Home Feed* (Tab) ke *Product Detail* (Stack). | `<Stack.Screen name="Tabs" component={TabNavigator} options={{ headerShown: false }} />` |
| **Stack di dalam Tabs/Drawer** | Setiap item *Tab* atau *Drawer* berisi *Stack Navigator* sendiri. | Digunakan untuk navigasi yang persisten dengan sub-alur mandiri. Contoh: Di dalam tab *Profile*, terdapat *Stack* untuk *Edit Profile*. | `<Tab.Screen name="ProfileStack" component={StackNavigator} />`; Gunakan `popToTopOnBlur=true` untuk reset fokus tab. |
| **Drawer di dalam Stack** | *Drawer* berfungsi sebagai menu utama aplikasi yang diakses dari layar pertama *Stack*. | Digunakan saat autentikasi berada di *Stack* (e.g., *AuthStack*) dan setelah login, *Drawer* muncul di atas tumpukan tersebut. | `drawerLockMode='locked-closed'` di *child* untuk mengunci *Drawer* saat berada di rute non-awal *Stack*. |
| **Hierarki Mendalam (3+ Tingkat)** | Kombinasi kompleks (e.g., Drawer → Tabs → Stack). | Aplikasi skala besar dengan banyak modul. **Sangat disarankan untuk diminimalisir.** | Gunakan `<Group>` untuk membungkus *screen* demi pengorganisasian tanpa menambah *overhead* *navigator* baru. |

### 2.4. Navigasi Lanjutan: Param Passing, Custom Actions, dan Optimasi

#### A. Pengiriman Parameter Lintas Tingkat

Navigasi ke *screen* yang bersarang dilakukan dengan menargetkan **nama navigator** sebagai rute utama dan menyertakan objek `screen` dan `params` secara berurutan.

```javascript
navigation.navigate('ParentName', {
  screen: 'ChildName',
  params: {
    screen: 'GrandchildName',
    id: 123, // Parameter untuk Grandchild
    initialTab: 'TabA' // Parameter untuk Child (jika Child adalah Tab Navigator)
  }
});
```

#### B. *Custom Actions* dan *State Management*

Untuk aksi yang tidak standar, seperti *reset* tumpukan setelah *login* atau navigasi ke *parent* dari *child* yang sangat dalam, gunakan metode dan *hooks* berikut:

| Hook/Method | Tipe | Deskripsi | Contoh Pola Penggunaan |
| :--- | :--- | :--- | :--- |
| `navigation.dispatch(action)` | Method | Mengirimkan aksi navigasi kustom (*custom action*) seperti `reset` atau `jumpTo`. | `dispatch(StackActions.reset({ index: 0, routes: [{ name: 'Home' }] }));` (Reset Stack) |
| `navigation.getParent(id?)` | Method | Mengakses objek *parent navigator* untuk memicu aksi di level atas. | `const parent = navigation.getParent(); parent.goBack();` (Aksi lintas level) |
| `useFocusEffect(effect)` | Hook | Menjalankan *side effect* hanya saat *screen* benar-benar fokus, aman untuk *nested screen*. | Digunakan untuk *fetch data* saat *tab* diaktifkan. |
| `useNavigationState(selector)` | Hook | Mengakses seluruh *state* navigasi aplikasi. Berguna untuk *Auth Guard*. | `const isAuthenticated = useNavigationState(s => s.routes[0].params?.token);` |

#### C. Praktik Terbaik (Best Practices) dan Optimasi

1. **Minimalkan Kedalaman *Nesting***: Usahakan maksimal 2-3 tingkat. Gunakan **Groups** (`<Group>`) untuk pengorganisasian kode tanpa menambah kompleksitas *state* navigasi.
2. ***Header* yang Bersih:** Setel `options={{ headerShown: false }}` pada `<Stack.Screen>` yang berisi *child navigator* (misalnya, *TabNavigator*) untuk menghindari *header* ganda (*duplicate headers*).
3. **Optimasi Performa:**
      * Gunakan `detachInactiveScreens={true}` di *parent navigator* untuk menghemat memori.
      * Gunakan `freezeOnBlur={true}` (memerlukan `react-native-screens`) untuk mencegah *re-render* di *screen* yang tidak aktif.
4. **Penargetan Spesifik:** Berikan `id` unik pada *navigator* atau *screen* (e.g., `<Tab.Screen id="HomeTabs" ... />`) agar dapat menargetkan aksi atau *listener* secara spesifik.

-----

## 3\. Contoh Implementasi

### 3.1. Contoh Dasar: Tabs Nested di Stack

**Pola:** *Tabs* sebagai *screen* di dalam *Stack*. Saat *push* ke **Detail**, *Tab Bar* akan hilang.

```jsx
// App.js - Setup
// ... (Imports: Stack, Tab, NavigationContainer)

const Tabs = () => (
  <Tab.Navigator screenOptions={{ headerShown: false }}>
    {/* Tab Screens di sini */}
    <Tab.Screen name="HomeTab" component={HomeScreen} />
    <Tab.Screen name="ProfileTab" component={ProfileScreen} />
  </Tab.Navigator>
);

const App = () => (
  <NavigationContainer>
    <Stack.Navigator>
      <Stack.Screen
        name="RootTabs"
        component={Tabs}
        options={{ headerShown: false }} // 🔑 HIDE HEADER Parent untuk Child Tabs
      />
      <Stack.Screen name="Detail" component={DetailScreen} />
    </Stack.Navigator>
  </NavigationContainer>
);

// screens/HomeScreen.js - Navigasi dari Tab ke Stack
// ...
const HomeScreen = () => {
  const navigation = useNavigation();
  return (
    // ...
    <Button title="Push Detail" onPress={() => navigation.navigate('Detail')} />
  ); // navigasi langsung ke rute 'Detail' di Parent Stack.
};
```

**Penjelasan:** *Root Stack* membungkus `Tabs`. Ketika `HomeScreen` (yang berada di dalam `Tabs`) memanggil `navigation.navigate('Detail')`, aksi ini akan **menggelembung** ke *Parent Stack Navigator* yang memiliki rute `Detail`, menyebabkan *Tab Bar* hilang karena `Detail` kini berada di atas tumpukan.

### 3.2. Contoh Lanjutan: Deep Nesting (Drawer Root dengan Tabs + Stack)

**Pola:** Drawer (Root) → Stack (Level 2) → Tabs (Level 3) → Detail (Level 2)

```jsx
// App.js - Setup
// ... (Imports: Stack, Tab, Drawer, NavigationContainer)

// Level 3: Tab Navigator
const HomeTabs = () => (
  <Tab.Navigator screenOptions={{ headerShown: false }}>
    {/* Tabs Screen */}
    <Tab.Screen name="HomeTab" component={HomeScreen} />
    <Tab.Screen name="ProfileTab" component={ProfileScreen} />
  </Tab.Navigator>
);

// Level 2: Stack Navigator (membungkus Tabs dan Detail)
const HomeStack = () => (
  <Stack.Navigator screenOptions={{ headerShown: false }}>
    <Stack.Screen name="Tabs" component={HomeTabs} /> // Tabs sebagai rute awal
    <Stack.Screen name="Detail" component={DetailScreen} options={{ headerShown: true }} /> // Header terlihat
  </Stack.Navigator>
);

// Level 1: Drawer Navigator (Root)
const App = () => (
  <NavigationContainer>
    <Drawer.Navigator initialRouteName="Home">
      <Drawer.Screen name="Home" component={HomeStack} options={{ headerShown: false }} /> // Stack di dalam Drawer
      <Drawer.Screen name="Settings" component={SettingsScreen} />
    </Drawer.Navigator>
  </NavigationContainer>
);

// screens/HomeScreen.js - Navigasi Lintas Level (dari Tabs ke Detail di Stack)
// ...
const HomeScreen = () => {
  const navigation = useNavigation();
  return (
    // ...
    <Button 
      title="Go to Detail" 
      onPress={() => navigation.navigate('Detail')} // Akses 'Detail' di Parent Stack
    />
  ); 
};
```

**Penjelasan:** Ini adalah pola aplikasi penuh. `HomeScreen` berada di *Tabs* (Level 3), yang dibungkus oleh *Stack* (Level 2). Ketika tombol **"Go to Detail"** ditekan, ia menavigasi ke `Detail` yang ada di *Stack Navigator* Level 2, menyebabkan *Tab Bar* menghilang tetapi *Drawer* (Level 1) tetap berfungsi sebagai *root*.

### 3.3. Contoh Tambahan: Custom Action dan Param Passing

**Skenario:** Dari *Screen* yang dalam, *reset* *Parent Stack* dan kirim *param*.

```javascript
// screens/DeepScreen.js
import { useNavigation, StackActions } from '@react-navigation/native';
// ...

const DeepScreen = () => {
  const navigation = useNavigation();
  
  const resetToHomeAndPassParam = () => {
    // 1. Dapatkan Parent Stack (misalnya ID-nya adalah 'HomeStack')
    const parentStack = navigation.getParent('HomeStack'); // Gunakan ID yang ditetapkan di Drawer
    
    // 2. Jika Parent Stack ada, reset Stack-nya
    if (parentStack) {
      parentStack.dispatch(
        StackActions.reset({
          index: 0, // Kembali ke awal Stack (yaitu Tabs)
          routes: [{ 
            name: 'Tabs', // Nama rute awal Stack
            params: { welcomeMessage: 'Reset Berhasil!' } // Opsional: kirim param ke Tabs
          }],
        })
      );
    }
  };

  return (
    // ...
    <Button title="Reset Stack & Back to Tabs" onPress={resetToHomeAndPassParam} />
  );
};
```

-----

## 4\. Rangkuman

Hari ke-11 telah menguraikan navigasi lanjutan melalui arsitektur bersarang: konsep Hierarki (independensi *state*, *bubbling*), pola umum (Tabs di Stack, Stack di Tabs), properti esensial (`screen/options` untuk *nesting*), dan teknik lanjutan seperti *auth guards* dan `useNavigationState` untuk aksi kustom. Kunci utama dalam membangun arsitektur yang *scalable* dan *performant* adalah: **meminimalkan kedalaman (*nesting* 2-3 tingkat maks.)**, menggunakan **`headerShown: false`** untuk UI yang bersih, dan memanfaatkan **Groups** untuk organisasi kode yang efisien.

-----

## 5\. Evaluasi Harian: Soal Praktik

Berikut adalah soal praktik lanjutan untuk proyek **Mini E-Commerce**, berfokus pada navigasi bersarang dan aksi lintas level.

### Soal Praktik 1: Perancangan Arsitektur Hierarki Tiga Tingkat

**Tugas:** Rancang ulang arsitektur navigasi aplikasi menjadi **hierarki tiga tingkat** (Drawer → Stack → Tabs) dengan ketentuan:

1. **Level 1 (Root):** **Drawer Navigator** (misalnya `RootDrawer`).
2. **Level 2 (Primary):** **Stack Navigator** (misalnya `HomeStack`) yang menjadi rute utama di dalam `RootDrawer`.
3. **Level 3 (Secondary):** **Material Top Tabs Navigator** yang menjadi **rute awal** (`initialRouteName`) dari `HomeStack`.

**Persyaratan UI:** Pastikan *header* dari **`HomeStack` (Level 2)** **tetap terlihat** di atas *Top Tabs* untuk menampilkan judul dan tombol menu, sementara *header* dari **Material Top Tabs (Level 3)** itu sendiri **dihilangkan** melalui opsi yang tepat.

-----

### Soal Praktik 2: Navigasi Lintas Tingkat (Tabs → Stack Detail)

**Tugas:** Terapkan fungsionalitas di **Top Tabs (Level 3)** yang memungkinkan navigasi dari tab produk langsung ke halaman **Detail Produk** yang berada di **`HomeStack` (Level 2)**.

**Persyaratan Aksi:**

1. Aksi navigasi harus membuat **Top Tabs Bar menghilang**.
2. **Drawer (Level 1)** harus tetap dapat dibuka (diakses).
3. Kirim `ID Produk` sebagai **parameter** saat menavigasi ke halaman **Detail Produk**.

-----

### Soal Praktik 3: Pengembalian ke Root Navigator dan Reset Stack

**Tugas:** Tambahkan fungsionalitas *reset stack* di halaman **Detail Produk** (Level 2). Buat sebuah tombol yang, ketika ditekan, memicu dua aksi kustom secara berurutan:

1. Melakukan **`Reset`** seluruh **`HomeStack` (Level 2)** ke rute awalnya, yaitu **Top Tabs**.
2. Secara programatik **menutup** **Drawer Navigator (Level 1)**.

-----

### Soal Praktik 4: Kontrol Aksi ke Parent Navigator Secara Eksplisit

**Tugas:** Di halaman **Detail Produk** (Level 2), buat tombol aksi baru berlabel **"Kembali ke Drawer Home"**. Tombol ini harus secara eksplisit memicu aksi `goBack()` pada **Parent Navigator** terdekat (yaitu **Drawer Navigator Level 1**) menggunakan metode `navigation.getParent()`.

**Tujuan:** Memastikan navigasi kembali ke item Drawer sebelumnya (bukan *pop* di Stack).

-----

### Soal Praktik 5: Implementasi Auth Guard di Level Nested

**Tugas:** Terapkan **Auth Guard** pada salah satu *Screen* di **Top Tabs (Level 3)**, misalnya di tab **'Profile'**.

**Persyaratan Logika:**

1. Gunakan *hook* **`useNavigationState`** atau mekanisme *state* navigasi lainnya untuk memeriksa **status otentikasi** (simulasikan *token* otentikasi yang disimpan di parameter *root navigasi*).
2. Jika pengguna **belum terotentikasi**, konten tab **'Profile'** harus menampilkan komponen *placeholder* sederhana bertuliskan **"Harap Login untuk mengakses"**, dan konten utama tab **tidak boleh dimuat**.

-----

## Ketentuan

* Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
* Soal wajib dikerjakan mandiri sesuai instruksi

***Mobile App Development With React Native***
