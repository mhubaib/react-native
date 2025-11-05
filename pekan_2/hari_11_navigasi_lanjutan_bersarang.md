# Hari ke-11 - Navigasi Lanjutan & Arsitektur Navigasi Bersarang

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

- Memahami arsitektur navigasi bersarang, termasuk konsep navigator hierarchy, independent state management per level, dan event bubbling untuk actions yang gagal di child navigator, sehingga merancang flows scalable tanpa over-nesting.
- Mengimplementasikan pola nested umum seperti tabs dalam stack (untuk modal-like detail), drawer dalam stack (untuk sidebar dengan drill-down), dan stack dalam tabs/drawer (untuk persistent nav dengan sub-flows), dengan konfigurasi seperti headerShown: false untuk clean UI.
- Mengintegrasikan advanced features seperti param passing antar levels (e.g., via navigate with { screen, params }), dan custom actions (e.g., popToTop di parent) menggunakan hooks seperti useNavigation dan navigation.dispatch.
- Menerapkan best practices navigation seperti minimize nesting (target 2-3 levels max), unique screen IDs untuk event targeting, dan layout props untuk platform-specific augmentation, sambil mengoptimasi performa dengan detachInactiveScreens dan freezeOnBlur.
- Mengelola troubleshooting seperti multiple headers, navigation conflicts, atau param inaccessibility dengan techniques seperti getParent() untuk parent access dan initial: false untuk respect child initials.

## 2. Materi Pembelajaran

Materi hari ini menjelaskan navigasi lanjutan dan arsitektur bersarang secara mendalam, fokus pada bagaimana multiple navigators saling bertumpuk (nesting) untuk membentuk tree structure. Setiap navigator mempertahankan state independen (history, params, options), dan actions seperti navigate dibubbling dari child ke parent jika gagal. Per 2025, react navigation merekomendasikan "flatten" hierarchies dengan Groups (non-navigator wrappers) untuk organization tanpa overhead. Instalasi di CLI: `npm i @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs @react-navigation/drawer @react-navigation/material-top-tabs react-native-screens react-native-safe-area-context react-native-gesture-handler`; tambah `npx react-native link` jika manual, dan import gesture-handler di index.js.

### A. Konsep Arsitektur Navigasi Bersarang: Hierarchy dan State Management

**Tujuan:** Arsitektur bersarang membentuk tree navigators di mana child navigator dirender sebagai screen di parent, memungkinkan composability tapi berpotensi kompleks. Pahami untuk hindari pitfalls seperti duplicate UI atau stuck actions.

**Konsep Kunci:**

- **Navigator Hierarchy:** Parent navigator renders child sebagai <Screen name="ChildNav" component={ChildNavigator} />; child punya own stack/routes, tapi inherit theme/options dari parent jika tidak di-override.
- **State Independence:** Setiap level punya separate navigation state (e.g., child stack punya own history, tak affect parent tabs). Actions resolve di deepest navigator dulu; jika invalid (e.g., navigate ke non-child route), bubble ke parent.
- **UI Rendering:** Parent UI (e.g., drawer overlay) di atas child; child UI (e.g., tab bar) visible kecuali di-hide. v7.x: Layout props allow custom positioning (e.g., drawer over tabs tanpa z-index hacks).
- **Event Bubbling:** Events seperti focus/blur bubble up; gunakan navigation.getId() untuk target specific level.

**Pola Umum Nested Navigation:**

| Pola | Deskripsi | Kapan Digunakan | Contoh Setup |
|------|-----------|-----------------|--------------|
| **Tabs in Stack** | Tabs sebagai screen di stack; tab bar hilang saat push detail. | Modal-like detail dari tab (e.g., home feed → product stack). | <Stack.Screen name="Tabs" component={TabNavigator} options={{ headerShown: false }} /> |
| **Drawer in Stack** | Drawer slide atas stack header; hanya open dari first stack screen. | Sidebar dengan drill-down (e.g., auth stack → drawer menu). | <Stack.Screen name="Drawer" component={DrawerNavigator} />; set drawerLockMode='locked-closed' di child. |
| **Stack in Tabs/Drawer** | Stack di tab/drawer item; tab bar/drawer tetap visible. | Persistent nav dengan sub-flows (e.g., profile tab → edit stack). | <Tab.Screen name="ProfileStack" component={StackNavigator} />; popToTopOnBlur=true untuk reset. |
| **Deep Nesting (Stack → Tabs → Stack)** | Multiple levels; gunakan Groups untuk flatten. | Complex apps (e.g., drawer → tabs → category stack). | Minimize: Gunakan <Group> untuk bundle screens tanpa new navigator. |
| **Hybrid (Drawer + Tabs + Stack)** | Drawer root → tabs → nested stack. | Full app (e.g., menu → home tabs → detail stack). | Root: Drawer; child: Tabs; grandchild: Stack; unique IDs: <Tab.Screen id="homeTabs" ... />. |

**Pola Penggunaan:** Navigate ke nested: `navigation.navigate('Parent', { screen: 'Child', params: { screen: 'Grandchild', params: { id: 123 } } })`. Access parent: `const parentNav = navigation.getParent('ParentId'); parentNav.goBack()`.

**Praktik Terbaik:** Minimize nesting (2-3 levels max) untuk performance; gunakan Groups untuk code organization tanpa state overhead; set headerShown: false di parent screen untuk child navigation; unique screen/navigator IDs untuk targeting; initial: false di navigate untuk respect child initialRouteName. Optimasi: detachInactiveScreens=true di parents; useFocusEffect untuk child-specific effects. Test: Simulator rotasi untuk layout shifts di nested.

**Troubleshooting Tips (Umum di Nested):**

| Issue | Penyebab | Solusi |
|-------|----------|--------|
| **Duplicate Headers** | Child header render di atas parent. | Set options={{ headerShown: false }} di parent screen; gunakan getHeaderTitle di child. |
| **Navigation Stuck** | Action invalid di child, tak bubble benar. | Gunakan navigation.dispatch(CommonActions.navigate({ name: 'Target', target: 'ParentId' })); check dengan getState(). |
| **Params Not Passed** | Nested params hilang di deep levels. | Gunakan { screen: 'Child', params: { ... }, initialParams: { ... } }; persist via route.params di child. |
| **Back Button Conflicts** | Hardware back pop wrong level. | Set backBehavior='initialRoute' di tabs; use popToTop() di onStateChange listener. |
| **Events Not Firing** | Listener di child tak catch parent events. | Gunakan navigation.getParent('Id').addListener('focus', ...); atau global listeners di NavigationContainer. |
| **Perf Degradation** | Deep nesting cause re-renders. | Enable freezeOnBlur; lazy=true di tabs; memoize screens dengan React.memo. |

**Pertimbangan Platform:** iOS: Nested gestures (e.g., swipe tab di drawer) auto-resolve; Android: Back button prioritaskan deepest stack. v7.x: Better foldable handling dengan per-window nesting; predictive back animasi di Android 15+.

### B. Navigasi Lanjutan: Auth Flows, Custom Actions, dan Event Handling

**Tujuan:** Lanjutan termasuk integrasi auth guards di nested routes, custom actions untuk non-standard flows, dan event handling untuk side effects.

**Hooks dan Methods Kunci (v7.x):**

| Hook/Method | Tipe | Deskripsi | Default | Catatan & Pola |
|-------------|------|-----------|---------|---------------|
| `useNavigationState(selector)` | Hook | Akses full state tree; selector: (state) => state. | - | Pola: const isAuthenticated = useNavigationState(s => s.routes[0].params?.token); untuk auth checks. |
| `navigation.dispatch(action)` | Method | Kirim custom action (e.g., reset stack). | - | Pola: dispatch(StackActions.reset({ index: 0, actions: [{ name: 'Home' }] })); untuk post-login redirect. |
| `useFocusEffect(effect)` | Hook | Run side effects on focus (child-safe). | - | Pola: useFocusEffect(useCallback(() => { fetchData(); return () => cleanup(); }, [])); di nested screens. |
| `navigation.getParent(id?)` | Method | Akses parent navigator. | - | Pola: const parent = navigation.getParent('ParentId'); parent.goBack(); untuk cross-level actions. |
| `navigation.getState()` | Method | Snapshot current state. | - | Pola: console.log(navigation.getState()); untuk debug hierarchy di development. |

**Pola Penggunaan:** Auth Flow: Root Stack dengan AuthStack (initial) → AppDrawer (post-login); guard: if (!token) navigation.replace('Login'). Custom Action: Extend dengan middleware untuk logging. Event Handling: Global listener di NavigationContainer: <NavigationContainer onStateChange={state => console.log(state)}>.

**Praktik Terbaik (2025):** Gunakan TypeScript untuk route types (create types via infer); auth dengan replace() untuk no-back. v7.x: Built-in event resolvers untuk nested; test dengan adb/xcrun untuk CLI.

**Pertimbangan Platform:** iOS: Event bubbling smooth di gestures; Android: Custom actions handle hardware back. v7.x: Async state updates untuk slow renders.

**Integrasi Keseluruhan:** Bersarang + lanjutan: Drawer (root) → Tabs → Stack (detail), dengan auth guards dan custom dispatch untuk reset.

## 3. Contoh Implementasi

### A. Contoh Dasar: Tabs Nested di Stack (Pola Umum)

```jsx
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import HomeScreen from './screens/HomeScreen';
import DetailScreen from './screens/DetailScreen';
import Ionicons from 'react-native-vector-icons/Ionicons';

const Stack = createNativeStackNavigator();
const Tab = createBottomTabNavigator();

const Tabs = () => (
  <Tab.Navigator screenOptions={{ headerShown: false }}>
    <Tab.Screen
      name="HomeTab"
      component={HomeScreen}
      options={{
        tabBarLabel: 'Home',
        tabBarIcon: ({ color }) => <Ionicons name="home" size={24} color={color} />,
      }}
    />
    <Tab.Screen
      name="ProfileTab"
      component={HomeScreen} // Ganti dengan Profile
      options={{
        tabBarLabel: 'Profile',
        tabBarIcon: ({ color }) => <Ionicons name="person" size={24} color={color} />,
      }}
    />
  </Tab.Navigator>
);

const App = () => (
  <NavigationContainer>
    <Stack.Navigator
      initialRouteName="Tabs"
      screenOptions={{ headerStyle: { backgroundColor: '#007AFF' }, headerTintColor: '#fff' }}
    >
      <Stack.Screen
        name="Tabs"
        component={Tabs}
        options={{ headerShown: false }} // Hide parent header
      />
      <Stack.Screen
        name="Detail"
        component={DetailScreen}
        options={{ title: 'Detail' }}
      />
    </Stack.Navigator>
  </NavigationContainer>
);

// screens/HomeScreen.js
import React from 'react';
import { View, Button, Text } from 'react-native';
import { useNavigation } from '@react-navigation/native';

const HomeScreen = () => {
  const navigation = useNavigation();
  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Home Tab</Text>
      <Button title="Push Detail" onPress={() => navigation.navigate('Detail')} />
    </View>
  );
};

export default HomeScreen;
```

**Penjelasan:** Tabs sebagai screen di stack; tab bar hilang saat push Detail; headerShown: false di Tabs untuk clean nesting.

### B. Contoh Interaktif: Drawer Nested di Stack dengan Param Passing

```jsx
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { createDrawerNavigator } from '@react-navigation/drawer';
import HomeScreen from './screens/HomeScreen';
import SettingsScreen from './screens/SettingsScreen';

const Stack = createNativeStackNavigator();
const Drawer = createDrawerNavigator();

const AppDrawer = () => (
  <Drawer.Navigator initialRouteName="Home">
    <Drawer.Screen name="Home" component={HomeScreen} />
    <Drawer.Screen name="Settings" component={SettingsScreen} />
  </Drawer.Navigator>
);

const App = () => (
  <NavigationContainer>
    <Stack.Navigator initialRouteName="Drawer">
      <Stack.Screen name="Drawer" component={AppDrawer} options={{ headerShown: false }} />
      <Stack.Screen name="DeepDetail" component={SettingsScreen} />
    </Stack.Navigator>
  </NavigationContainer>
);

// screens/HomeScreen.js
import React from 'react';
import { View, Button, Text } from 'react-native';
import { useNavigation } from '@react-navigation/native';

const HomeScreen = () => {
  const navigation = useNavigation();
  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>Home in Drawer</Text>
      <Button
        title="Navigate to Deep Detail"
        onPress={() =>
          navigation.navigate('DeepDetail', {
            screen: 'Drawer', // Target parent stack
            params: { id: 123 },
          })
        }
      />
    </View>
  );
};

export default HomeScreen;
```

**Penjelasan:** Drawer di stack; navigate ke deep screen dengan params; bubble action ke parent stack.

### C. Contoh Lanjutan: Deep Nesting - Drawer Root dengan Tabs + Stack

```jsx
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createDrawerNavigator } from '@react-navigation/drawer';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import Ionicons from 'react-native-vector-icons/Ionicons';
import HomeScreen from './screens/HomeScreen';
import DetailScreen from './screens/DetailScreen';
import ProfileScreen from './screens/ProfileScreen';

const Stack = createNativeStackNavigator();
const Tab = createBottomTabNavigator();
const Drawer = createDrawerNavigator();

const HomeTabs = () => (
  <Tab.Navigator screenOptions={{ headerShown: false }}>
    <Tab.Screen
      name="HomeTab"
      component={HomeScreen}
      options={{ tabBarIcon: ({ color }) => <Ionicons name="home" size={24} color={color} /> }}
    />
    <Tab.Screen
      name="ProfileTab"
      component={ProfileScreen}
      options={{ tabBarIcon: ({ color }) => <Ionicons name="person" size={24} color={color} /> }}
    />
  </Tab.Navigator>
);

const HomeStack = () => (
  <Stack.Navigator>
    <Stack.Screen name="Tabs" component={HomeTabs} options={{ headerShown: false }} />
    <Stack.Screen name="Detail" component={DetailScreen} />
  </Stack.Navigator>
);

const App = () => (
  <NavigationContainer>
    <Drawer.Navigator initialRouteName="Home" screenOptions={{ headerShown: false }}>
      <Drawer.Screen
        name="Home"
        component={HomeStack}
        options={{ drawerIcon: ({ color }) => <Ionicons name="home" size={24} color={color} /> }}
      />
      <Drawer.Screen
        name="Settings"
        component={ProfileScreen}
        options={{ drawerIcon: ({ color }) => <Ionicons name="settings" size={24} color={color} /> }}
      />
    </Drawer.Navigator>
  </NavigationContainer>
);

// screens/HomeScreen.js (dari 3.1, dengan navigate ke Detail)
```

**Penjelasan:** Drawer root → Stack (tabs + detail); deep nesting. Rebuild untuk CLI.

### D. Contoh Tambahan: Custom Action dengan useNavigationState untuk Auth Guard

```jsx
// components/AuthGuard.js
import React from 'react';
import { View, Text } from 'react-native';
import { useNavigationState } from '@react-navigation/native';

const AuthGuard = ({ children, requiredToken }) => {
  const isAuthenticated = useNavigationState(state => {
    // Check root params or global state
    const root = state.routes[0];
    return root.params?.token === requiredToken;
  });

  if (!isAuthenticated) {
    return (
      <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
        <Text>Login Required</Text>
      </View>
    );
  }

  return children;
};

// Integrasi di App.js (wrap nested screens)
const HomeStack = () => (
  <AuthGuard requiredToken="abc123">
    <Stack.Navigator>
      {/* Screens */}
    </Stack.Navigator>
  </AuthGuard>
);
```

**Penjelasan:** Hook akses state untuk guard; custom logic berdasarkan params. Extend dengan Redux untuk global auth.

**Tips Debugging:** Gunakan `console.log(navigation.getState())` untuk inspect hierarchy; test dengan adb/xcrun di CLI.

## 4. Rangkuman

Hari ke-11 mendalami navigasi lanjutan melalui arsitektur bersarang: Hierarchy concepts (independent state, bubbling), pola umum (tabs in stack, drawer in stack), props (screen/options untuk nesting), dan lanjutan seperti auth guards/useNavigationState untuk custom actions. Kunci: Minimize levels (2-3 max), headerShown: false untuk clean UI, unique IDs untuk targeting, dan Groups untuk flatten. Integrasi ini ciptakan flows scalable yang performant dan maintainable, siap untuk production dengan TypeScript typing dan foldable support di v7.x.

**Latihan Selanjutnya:** Bangun full app dengan drawer → tabs → stack, tambah auth guard di nested route.

**Referensi:**

- [Nesting navigators | React Navigation](https://reactnavigation.org/docs/nesting-navigators)
- [Mastering React Native Navigation for Beginners (2025 Edition)](https://medium.com/react-native-journal/mastering-react-native-navigation-for-beginners-2025-edition-44b42cced32d)
- [Your Complete Guide to React Native Navigation in 2025](https://www.peanutsquare.com/your-complete-guide-to-react-native-navigation-in-2025/)
- [Best Practices for Creating Navigation in a React Native Project](https://medium.com/@tusharkumar27864/react-native-navigation-simplified-best-practices-and-tips-748df1217a70)
- [React Navigation Patterns: Build Intuitive Mobile Apps](https://softwarehouse.au/blog/navigation-patterns-with-react-navigation-building-intuitive-mobile-flows/)

Tentu, saya akan membuat 5 contoh soal praktik lanjutan yang sesuai dengan materi **Navigasi Bersarang** dan **Arsitektur Navigasi Lanjutan** (Hari ke-11), melanjutkan skenario proyek Mini E-Commerce dan hasil dari soal-soal sebelumnya.

Soal-soal ini diformulasikan sebagai **perintah lugas** tanpa instruksi teknis.

---

## 5. Evaluasi Harian: Soal Praktik

`Lanjutan project Mini E-Commerce`

### Soal Praktik 1: Perancangan Arsitektur Hierarki Tiga Tingkat

**Tugas:** Rancang ulang seluruh arsitektur navigasi aplikasi menjadi **hierarki tiga tingkat** (Drawer → Stack → Tabs) untuk alur yang kompleks:

1. **Tingkat 1 (Root):** Gunakan **Drawer Navigator** sebagai navigasi utama (Root).
2. **Tingkat 2 (Primary):** Buat **Stack Navigator** sebagai layar utama di dalam salah satu item Drawer. *Stack* ini harus berisi *Tab Navigator* sebagai rute awalnya.
3. **Tingkat 3 (Secondary):** Gunakan **Material Top Tabs Navigator** (dari Soal Praktik sebelumnya) sebagai rute awal *Stack* Tingkat 2.

Pastikan *header* dari *Stack* Tingkat 2 **tetap terlihat** di atas *Top Tabs* untuk menampilkan judul dan tombol menu, tetapi *header* dari *Top Tabs* itu sendiri **dihilangkan**.

---

### Soal Praktik 2: Navigasi Lintas Tingkat untuk Halaman Detail

**Tugas:** Di dalam **Top Tabs** (Tingkat 3), tambahkan fungsionalitas tombol yang memungkinkan navigasi dari tab produk (**Tingkat 3**) langsung ke halaman **Detail Produk** yang berada di **Stack Navigator** (Tingkat 2). Saat navigasi terjadi, pastikan **Top Tabs Bar menghilang**, tetapi **Drawer** (Tingkat 1) tetap dapat diakses atau dibuka. Kirim **ID Produk** sebagai *parameter* saat menavigasi ke halaman **Detail Produk**.

---

### Soal Praktik 3: Pengembalian ke Root Navigator dan Reset Stack

**Tugas:** Tambahkan fungsionalitas *reset stack* di halaman **Detail Produk** (Tingkat 2). Buat sebuah tombol yang, ketika ditekan:

1. Melakukan **Reset** seluruh Stack Navigator (Tingkat 2) ke rute awalnya, yaitu **Top Tabs**.
2. Secara programatik **menutup** Drawer Navigator (Tingkat 1).

Ini menyimulasikan pengalaman "Selesai Berbelanja" atau "Kembali ke Beranda Utama" yang bersih tanpa riwayat detail.

---

### Soal Praktik 4: Kontrol Aksi ke Parent Navigator

**Tugas:** Di halaman Detail Produk (Tingkat 2), buat tombol aksi baru berlabel **"Kembali ke Drawer Home"**. Tombol ini harus secara eksplisit memicu aksi `goBack()` pada **Parent Navigator** (yaitu Drawer Navigator Tingkat 1) untuk kembali ke item navigasi sebelumnya di Drawer, tanpa mempengaruhi Stack di dalamnya.

---

### Soal Praktik 5: Implementasi Auth Guard di Level Nested

**Tugas:** Terapkan **Auth Guard** di salah satu *Screen* **Top Tabs** (Tingkat 3), misalnya di tab **'Profile'** atau **'Settings'**. Gunakan *hook* **`useNavigationState`** atau mekanisme *state* navigasi lainnya untuk memeriksa **status otentikasi** (simulasikan token otentikasi di parameter root navigasi). Jika pengguna **belum terotentikasi**, konten tab **tidak boleh dimuat**, dan sebaliknya harus menampilkan komponen placeholder sederhana bertuliskan "Harap Login untuk mengakses".

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

---

**Mobile App Development With React Native*
