# Hari ke-10 - Screen Navigation: Drawer Navigation & Material Top Tabs Navigation

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

- Mengimplementasikan Drawer Navigator untuk navigasi samping, termasuk swipe gestures, custom drawer content (e.g., user profile, settings), overlay effects, dan nested dengan Stack/Tabs untuk alur hybrid, dengan optimasi seperti edge swipe dan keyboard handling.
- Mengimplementasikan Material Top Tabs Navigator untuk tab horizontal atas, termasuk swipe transitions, lazy loading, custom indicators, dan integrasi dengan ScrollView untuk infinite tabs, sambil menangani overflow di landscape atau foldables.
- Mengintegrasikan kedua navigator dalam struktur kompleks (e.g., drawer root dengan nested top tabs), menggunakan NavigationContainer untuk theme consistency, dan hooks seperti useFocusEffect untuk side effects per route.
- Menerapkan praktik terbaik seperti custom drawer/top tab components untuk branding.
- Membangun prototipe aplikasi dengan drawer sidebar (e.g., menu utama) dan top tabs di home screen (e.g., categories), siap untuk full auth flows.

## 2. Materi Pembelajaran

Materi hari ini menjelaskan Drawer dan Material Top Tabs Navigator secara mendalam, membangun atas Stack/Bottom Tabs. Kedua navigator mendukung declarative routing via `<NavigationContainer>`, dengan v7 menambahkan layout props (e.g., `layout: { drawerPosition: 'left' }`) untuk fine-tuning. Untuk React Native CLI, instalasi: `npm i @react-navigation/drawer @react-navigation/material-top-tabs react-native-gesture-handler react-native-reanimated react-native-screens react-native-safe-area-context` (untuk drawer), dan `npm i react-native-pager-view` untuk top tabs.

### A. Drawer Navigation: Menu Samping untuk Akses Sekunder

**Tujuan:** Drawer Navigator menyediakan slide-out menu dari edge (kiri/kanan) untuk secondary navigation, ideal untuk settings atau user actions, dengan gesture swipe dan overlay untuk fokus. Di v7, dukung rail mode (compact drawer) dan better animation interpolation untuk foldables.

**Setup Dasar:** Gunakan `createDrawerNavigator()`; wrap screens di `<Drawer.Navigator>`; enable `react-native-gesture-handler` dengan import di index.js: `import 'react-native-gesture-handler';`. Untuk CLI, rebuild app setelah instal: `npx react-native run-android` atau `npx react-native run-ios`.

**Props Kunci (Drawer.Navigator & Screen, v7.x):**

| Prop | Tipe | Deskripsi | Default | Catatan Platform & Best Practices |
|------|------|-----------|---------|----------------------------------|
| `initialRouteName` | string | Route awal drawer. | First screen | Set untuk user-specific start (e.g., based on auth). |
| `screenOptions` | object/function | Global config (drawer, gestures). | {} | Function: `({ route }) => ({ drawerItemStyle: { backgroundColor: route.name === 'Settings' ? '#f0f0f0' : 'white' } })`; dynamic styling. |
| `drawerPosition` | 'left'/'right' | Sisi slide-out. | 'left' | 'right' untuk RTL languages; layout prop di v7: `layout: { drawerPosition: 'left' }`. |
| `drawerType` | 'front'/'back'/'slide'/'permanent'/'permanent-left'/'permanent-right' | Drawer behavior. | 'front' | 'permanent' untuk always-visible (tablet); 'slide' untuk overlay. |
| `drawerStyle` | ViewStyle | Drawer container (width, bg). | {} | { width: 280, backgroundColor: '#fff' }; use safe area insets untuk padding. |
| `drawerContent` | Component | Custom drawer renderer. | Default | (props) => <CustomDrawerContent {...props} />; kunci untuk branding/user profile. |
| `drawerContentOptions` | object | Config content (activeTintColor). | {} | { activeTintColor: '#007AFF', inactiveTintColor: 'gray' }; accessibility: labelStyle fontSize 16. |
| `overlayColor` | Color | Backdrop tint saat open. | 'rgba(0,0,0,0.5)' | 'transparent' untuk no dim; custom opacity untuk subtle focus. |
| `drawerHideStatusBarOnOpen` | boolean | Sembunyikan status bar saat open. | false | true untuk immersive; kombinasikan dengan StatusBar.setHidden. |
| `gestureEnabled` | boolean | Swipe dari edge. | true | false di locked modes; custom: `edgeWidth: 20` untuk narrow trigger. |
| `keyboardDismissMode` | 'none'/'on-drag'/'interactive' | Keyboard handling saat drag. | 'none' | 'on-drag' untuk forms di drawer; iOS 'interactive' untuk smooth. |
| `drawerLockMode` | 'unlocked'/'locked-closed'/'locked-open' | Lock drawer. | 'unlocked' | 'locked-closed' saat loading; dynamic via setOptions. |
| `sceneContainerStyle` | ViewStyle | Content area (padding). | {} | { paddingLeft: drawerWidth } untuk permanent; integrasi safe areas. |
| `edgeWidth` | number | Swipe trigger width. | 20 | 0 untuk full-screen swipe; test thumb reachability (44pt min). |
| `minDrawerWidth` | number | Min width drawer. | 0 | Untuk responsive di foldables; auto-scale dengan Dimensions. |
| `listeners` | object | Events (drawerOpen, stateChange). | {} | useEffect: navigation.addListener('drawerOpen', onOpen); untuk analytics. |
| `layout` | object (v7) | Augmented layout (e.g., { drawerType: 'permanent' }). | {} | Baru di v7: `layout: { drawerPosition: 'left', drawerType: 'back' }`; composable dengan screen props. |

**Pola Penggunaan:** Open: `navigation.openDrawer()`; Close: `navigation.closeDrawer()`; Toggle: `navigation.toggleDrawer()`. Nested: Drawer root dengan Stack/Tabs di screens (e.g., HomeStack di 'Home' drawer item). Pola custom: Gunakan DrawerContentScrollView di drawerContent untuk scrollable menu.

**Praktik Terbaik (2025):** Gunakan 'front' type untuk mobile; 'permanent' di tablet (>600px width); custom content untuk user avatar/logout. Enable gesture + keyboardDismissMode untuk forms. v7: Layout props untuk hybrid dengan bottom tabs (no overlap). Test RTL: drawerPosition auto-flip. Untuk CLI, rebuild setelah link gesture-handler.

**Pertimbangan Platform:** iOS: Slide animation + swipe; Android: Reveal + back-to-close. v7: Predictive back support di Android 15+; foldables: minDrawerWidth adaptif.

### B. Material Top Tabs Navigation: Tab Horizontal untuk Content Segmented

**Tujuan:** Material Top Tabs Navigator menyediakan swipeable tabs atas dengan indicator animasi, ideal untuk segmented content (e.g., categories, views), menggunakan react-native-pager-view untuk hardware acceleration. Di v7, dukung lazy pagination dan custom scroll sync.

**Setup Dasar:** Instal `@react-navigation/material-top-tabs` + `react-native-pager-view`; gunakan `createMaterialTopTabNavigator()`. Untuk CLI, tambahkan `npx react-native link react-native-pager-view` jika manual, dan rebuild app.

**Props Kunci (TopTab.Navigator & Screen, v7.x):**

| Prop | Tipe | Deskripsi | Default | Catatan Platform & Best Practices |
|------|------|-----------|---------|----------------------------------|
| `initialRouteName` | string | Tab awal. | First tab | Dynamic berdasarkan params (e.g., last viewed category). |
| `screenOptions` | object/function | Global config (tabBar, pagination). | {} | Function: `({ route }) => ({ tabBarLabel: route.params?.label })`; theme-aware. |
| `tabBarPosition` | 'top'/'bottom' | Posisi bar. | 'top' | 'bottom' untuk inverted flows; v7: Layout prop untuk dynamic. |
| `tabBarStyle` | ViewStyle | Bar container (bg, height). | {} | { backgroundColor: '#f0f0f0', elevation: 8 }; platform shadow/elevation. |
| `tabBarActiveTintColor` | Color | Active label/indicator. | Primary | '#007AFF'; animasi: tabBarIndicatorStyle untuk custom width. |
| `tabBarInactiveTintColor` | Color | Inactive label. | Muted | 'gray'; opacity 0.7 untuk subtle. |
| `tabBarLabelStyle` | TextStyle | Label font. | {} | { fontSize: 12, fontWeight: 'bold' }; accessibility: min fontScale. |
| `tabBarIndicatorStyle` | ViewStyle | Indicator line/box. | {} | { backgroundColor: '#007AFF', height: 3 }; custom: Animated.Value untuk width. |
| `tabBarItemStyle` | ViewStyle | Per-tab padding. | {} | { width: screenWidth / tabs.length }; auto-calc untuk even spacing. |
| `lazy` | boolean | Mount tab saat focus. | true | false untuk sync scroll (e.g., shared ScrollView); perf: true di dynamic tabs. |
| `lazyPreloadMaxDistance` | number | Preload tabs sebelum/after current. | 0 | 1 untuk adjacent; hemat memori di infinite tabs. |
| `swipeEnabled` | boolean | Horizontal swipe. | true | false di locked modes; custom: `getGestureResponder` untuk conflict resolution. |
| `paginationEnabled` | boolean | Pager swipe. | true | false untuk button-only; v7: Better sync dengan ScrollView. |
| `scrollEnabled` | boolean | Scroll tab bar jika overflow. | false | true di >5 tabs; auto-hide handles di landscape. |
| `showPageIndicator` | boolean | Dots untuk pagination. | false | true di carousel-like; custom style untuk positioning. |
| `upperCaseLabel` | boolean | Uppercase labels. | Platform default | false untuk modern lowercase; Material Design compliance. |
| `activeTabKey` | string | Force active tab. | From state | Dynamic untuk external control (e.g., URL hash). |
| `transitionSpec` | TransitionSpec | Swipe anim timing. | Default | { open: { duration: 200, easing: Easing.bezier(0.25, 0.46, 0.45, 0.94) } }; spring untuk bounce. |
| `layout` | object (v7) | Augmented layout (e.g., { tabBarPosition: 'top' }). | {} | Baru: `layout: { tabBarStyle: { position: 'absolute' } }`; composable dengan nested nav. |
| `listeners` | object | Events (tabPress, swipeStart). | {} | navigation.addListener('tabPress', onPress); untuk lazy load data. |

**Pola Penggunaan:** Switch: `navigation.navigate('Tab2')`; Nested: Top tabs di Stack screen (e.g., CategoryStack → TopTabs). Pola infinite: Dynamic routes dengan useState untuk add tabs.

**Praktik Terbaik (2025):** Gunakan lazy=true + preload untuk perf; custom indicator dengan Reanimated untuk smooth. Integrasi ScrollView: `syncScroll={true}` untuk shared scrolling. v7: Layout props untuk hybrid dengan drawer (no z-index conflicts). Test swipe: Disable di forms untuk prevent accidental switches. Untuk CLI, pastikan pager-view linked dan rebuild.

**Pertimbangan Platform:** iOS: Cupertino-style labels; Android: Material indicator. v7: PagerView acceleration di Android 14+; foldables: ScrollEnabled adaptif ke width.

**Integrasi Keseluruhan:** Drawer root dengan nested Top Tabs di 'Explore' item; gunakan useIsFocused untuk tab-specific effects.

## 3. Contoh Implementasi

Kode siap di React Native CLI (bare workflow). Instal dependencies seperti dijelaskan; untuk icons, gunakan react-native-vector-icons (instal `npm i react-native-vector-icons`, link dengan `npx react-native link react-native-vector-icons`, dan setup Android/iOS manual per docs). Rebuild app setelah setup: `npx react-native run-android` atau `npx react-native run-ios`. Asumsikan screens sederhana.

### A. Contoh Dasar: Drawer Navigator Sederhana

```jsx
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createDrawerNavigator } from '@react-navigation/drawer';
import HomeScreen from './screens/HomeScreen';
import SettingsScreen from './screens/SettingsScreen';
import Ionicons from '@react-native-vector-icons/ionicons';

const Drawer = createDrawerNavigator();

const App = () => (
  <NavigationContainer>
    <Drawer.Navigator
      initialRouteName="Home"
      screenOptions={{
        drawerStyle: { backgroundColor: '#f0f0f0', width: 280 },
        drawerActiveTintColor: '#007AFF',
        drawerInactiveTintColor: 'gray',
        gestureEnabled: true,
        drawerType: 'front',
      }}
    >
      <Drawer.Screen
        name="Home"
        component={HomeScreen}
        options={{ title: 'Beranda', drawerIcon: ({ color }) => <Ionicons name="home" size={24} color={color} /> }}
      />
      <Drawer.Screen
        name="Settings"
        component={SettingsScreen}
        options={{ title: 'Pengaturan', drawerIcon: ({ color }) => <Ionicons name="settings" size={24} color={color} /> }}
      />
    </Drawer.Navigator>
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
      <Text>Halaman Home</Text>
      <Button title="Buka Drawer" onPress={() => navigation.toggleDrawer()} />
    </View>
  );
};

export default HomeScreen;
```

**Penjelasan:** Drawer dasar dengan icons (dari react-native-vector-icons); swipe left untuk open; custom width/bg. Rebuild app setelah link icons.

### B. Contoh Interaktif: Material Top Tabs Navigator

```jsx
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createMaterialTopTabNavigator } from '@react-navigation/material-top-tabs';
import Tab1Screen from './screens/Tab1Screen';
import Tab2Screen from './screens/Tab2Screen';

const Tab = createMaterialTopTabNavigator();

const App = () => (
  <NavigationContainer>
    <Tab.Navigator
      initialRouteName="Tab1"
      screenOptions={{
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: 'gray',
        tabBarIndicatorStyle: { backgroundColor: '#007AFF', height: 3 },
        lazy: true,
        swipeEnabled: true,
      }}
    >
      <Tab.Screen
        name="Tab1"
        component={Tab1Screen}
        options={{ tabBarLabel: 'Kategori 1' }}
      />
      <Tab.Screen
        name="Tab2"
        component={Tab2Screen}
        options={{ tabBarLabel: 'Kategori 2' }}
      />
    </Tab.Navigator>
  </NavigationContainer>
);

// screens/Tab1Screen.js
import React from 'react';
import { View, Text } from 'react-native';

const Tab1Screen = () => (
  <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
    <Text>Konten Tab 1</Text>
  </View>
);

export default Tab1Screen;
```

**Penjelasan:** Top tabs dengan swipe; custom indicator; lazy untuk perf. Link pager-view dan rebuild untuk CLI.

### C. Contoh Lanjutan: Hybrid Drawer + Nested Top Tabs

```jsx
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createDrawerNavigator } from '@react-navigation/drawer';
import { createMaterialTopTabNavigator } from '@react-navigation/material-top-tabs';
import Ionicons from '@react-native-vector-icons/ionicons'; // Setup vector icons
import HomeScreen from './screens/HomeScreen';
import ExploreScreen from './screens/ExploreScreen'; // Top Tabs di sini
import ProfileScreen from './screens/ProfileScreen';

const Drawer = createDrawerNavigator();
const Tab = createMaterialTopTabNavigator();

const ExploreTabs = () => (
  <Tab.Navigator>
    <Tab.Screen name="Tab1" component={HomeScreen} options={{ tabBarLabel: 'Feed' }} />
    <Tab.Screen name="Tab2" component={ProfileScreen} options={{ tabBarLabel: 'Favorites' }} />
  </Tab.Navigator>
);

const App = () => (
  <NavigationContainer>
    <Drawer.Navigator screenOptions={{ headerShown: false }}>
      <Drawer.Screen
        name="Home"
        component={HomeScreen}
        options={{
          title: 'Home',
          drawerIcon: ({ color }) => <Ionicons name="home" size={24} color={color} />,
        }}
      />
      <Drawer.Screen
        name="Explore"
        component={ExploreTabs}
        options={{
          title: 'Explore',
          drawerIcon: ({ color }) => <Ionicons name="search" size={24} color={color} />,
        }}
      />
      <Drawer.Screen
        name="Profile"
        component={ProfileScreen}
        options={{
          title: 'Profile',
          drawerIcon: ({ color }) => <Ionicons name="person" size={24} color={color} />,
        }}
      />
    </Drawer.Navigator>
  </NavigationContainer>
);
```

**Penjelasan:** Drawer root dengan nested Top Tabs di 'Explore'; swipe tabs dalam drawer. Rebuild setelah setup pager-view dan icons.

### D. Contoh Tambahan: Custom Drawer Component

Untuk kasus custom drawer (e.g., dengan user avatar, dividers, atau Reanimated animations), buat komponen yang menerima props dari Drawer.Navigator. Ini memungkinkan personalisasi seperti dynamic menu berdasarkan user role atau scrollable content dengan sections.

**Setup:** Di Drawer.Navigator, set `drawerContent={(props) => <CustomDrawer {...props} />}`. Komponen harus render `<DrawerContentScrollView {...props}>` dan handle `descriptors` untuk items. Untuk CLI, pastikan gesture-handler diindex.js.

```jsx
// components/CustomDrawer.js
import React from 'react';
import {
  View,
  TouchableOpacity,
  Text,
  StyleSheet,
  SafeAreaView,
  Image,
} from 'react-native';
import Ionicons from 'react-native-vector-icons/Ionicons'; // Setup vector icons
import { DrawerContentScrollView, DrawerItemList } from '@react-navigation/drawer';

const CustomDrawer = (props) => {
  const { state, navigation, descriptors } = props;
  const user = { name: 'John Doe', avatar: 'https://via.placeholder.com/80' }; // Simulasi user data

  const handleLogout = () => {
    // Logic logout
    navigation.closeDrawer();
  };

  return (
    <SafeAreaView style={styles.container}>
      <DrawerContentScrollView {...props} style={styles.scrollView}>
        {/* Custom Header */}
        <View style={styles.header}>
          <Image source={{ uri: user.avatar }} style={styles.avatar} />
          <View>
            <Text style={styles.userName}>{user.name}</Text>
            <Text style={styles.userEmail}>john@example.com</Text>
          </View>
        </View>
        {/* Custom Items */}
        {state.routes.map((route, index) => {
          const isFocused = state.index === index;
          const onPress = () => {
            const event = navigation.emit({
              type: 'drawerItemPress',
              target: route.key,
              canPreventDefault: true,
            });
            if (!isFocused && !event.defaultPrevented) navigation.navigate(route.name);
          };

          return (
            <TouchableOpacity
              key={route.key}
              style={[styles.customItem, isFocused && styles.focusedItem]}
              onPress={onPress}
              accessibilityRole="button"
            >
              <Ionicons name="home" size={24} color={isFocused ? '#007AFF' : '#666'} />
              <Text style={[styles.itemLabel, isFocused && styles.focusedLabel]}>
                {descriptors[route.key].options.title || route.name}
              </Text>
            </TouchableOpacity>
          );
        })}
        {/* Divider */}
        <View style={styles.divider} />
        {/* Logout */}
        <TouchableOpacity style={styles.logout} onPress={handleLogout}>
          <Ionicons name="log-out-outline" size={24} color="#ff4444" />
          <Text style={styles.logoutText}>Logout</Text>
        </TouchableOpacity>
      </DrawerContentScrollView>
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1 },
  scrollView: { flex: 1 },
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 20,
    borderBottomWidth: 1,
    borderBottomColor: '#e0e0e0',
  },
  avatar: { width: 50, height: 50, borderRadius: 25, marginRight: 15 },
  userName: { fontSize: 18, fontWeight: 'bold' },
  userEmail: { fontSize: 14, color: '#666' },
  customItem: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 16,
    marginHorizontal: 10,
  },
  focusedItem: { backgroundColor: '#e3f2fd' },
  itemLabel: { marginLeft: 16, fontSize: 16 },
  focusedLabel: { color: '#007AFF' },
  divider: { height: 1, backgroundColor: '#e0e0e0', marginVertical: 10 },
  logout: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 16,
    marginHorizontal: 10,
    borderTopWidth: 1,
    borderTopColor: '#e0e0e0',
  },
  logoutText: { marginLeft: 16, fontSize: 16, color: '#ff4444' },
});

export default CustomDrawer;
```

**Integrasi ke App.js (Modifikasi dari 3.1):**

```jsx
// Di Drawer.Navigator
<Drawer.Navigator
  drawerContent={(props) => <CustomDrawer {...props} />}
  // ... other options
>
  {/* Screens */}
</Drawer.Navigator>
```

**Penjelasan:** CustomDrawer gunakan DrawerContentScrollView base; custom header dengan avatar; items manual dari state/descriptors; logout handler. Ini memungkinkan fitur seperti role-based menu atau animations (tambah Reanimated). Test: Swipe open, tap items, accessibility dengan role/label. Rebuild setelah setup icons.

**Tips Debugging:** Gunakan `adb logcat` untuk stack traces; test deep links dengan `adb shell am start -W -a android.intent.action.VIEW -d "myapp://home" com.myapp` (Android) atau `xcrun simctl openurl booted "myapp://home"` (iOS).

## 4. Rangkuman

Hari ke-10 melengkapi navigasi dengan Drawer Navigator untuk menu samping swipeable (custom content, gestures, permanent mode) dan Material Top Tabs untuk segmented swipe tabs (lazy pagination, indicators, scroll sync). Kunci: Nested composability dengan Stack/Tabs; v7 layout props untuk augmentation; custom components untuk branding. Integrasi ini ciptakan alur kompleks yang gesture-native dan performant, siap untuk full-stack apps dengan auth atau web parity.

**Latihan Selanjutnya:** Integrasikan drawer dengan top tabs di explore; tambah custom animations pada swipe.

**Referensi:**

- [Drawer Navigator | React Navigation](https://reactnavigation.org/docs/drawer-navigator/)
- [Material Top Tabs Navigator | React Navigation](https://reactnavigation.org/docs/material-top-tab-navigator)
- [React Navigation 7.0 Release Notes](https://reactnavigation.org/blog/2024/11/06/react-navigation-7.0/)
- [Customizing Drawer Content | React Navigation](https://reactnavigation.org/docs/drawer-navigator#custom-drawer-content)
- [Advanced Top Tabs Usage | React Navigation](https://reactnavigation.org/docs/material-top-tab-navigator#advanced-features)

## 5. Evaluasi Harian: Soal Praktik

`Lanjutan project Mini E-Commerce`

### 1: Perancangan Menu Utama dengan Avatar Pengguna

**Tugas:** Buat navigasi utama aplikasi (sebagai *root*) menggunakan **Drawer Navigator**. Isi Drawer tersebut dengan komponen **kustom** yang menampilkan **gambar profil (avatar)** dan **nama pengguna** di bagian atas, serta memiliki item menu untuk `'Home'`, `'Settings'`, dan tombol `'Logout'` terpisah di bagian bawah. Pastikan menu samping ini bisa dibuka dengan **swipe** dari tepi layar.

---

### 2: Segmentasi Konten Halaman Utama

**Tugas:** Ganti konten layar `'Home'` dengan **Material Top Tabs Navigator** untuk mengelompokkan tampilan produk. Buat 3 tab yang bisa di-swipe, yaitu: **'Populer'**, **'Terbaru'**, dan **'Diskon'**. Kustomisasi tampilan bar tab tersebut agar **garis indikator** berwarna biru primer dan **label tab** tidak menggunakan huruf kapital semua (*lowercase* atau *title case*).

---

### 3: Optimasi Loading Tab yang Efisien

**Tugas:** Terapkan teknik **lazy loading** pada **Material Top Tabs** untuk memastikan konten setiap tab hanya dimuat saat diakses (fokus), namun tambahkan *preload* untuk tab yang berdekatan. Buktikan efisiensi ini dengan menambahkan *side effect* yang menampilkan pesan log ke konsol **hanya saat** tab **'Diskon'** benar-benar sedang **aktif** (di-*focus*) dan pesan pembersihan saat tab tersebut ditinggalkan.

---

### 4: Mode Kunci Navigasi

**Tugas:** Terapkan mekanisme penguncian (*locking*) pada **Drawer Navigator**. Secara *default*, atur agar Drawer **tidak dapat dibuka** melalui *swipe gesture*. Di dalam layar **'Settings'**, sediakan tombol atau *toggle* yang memungkinkan pengguna untuk **membuka kunci** *swipe gesture* Drawer secara programatik. Selain itu, sediakan tombol yang akan menavigasi kembali ke **'Home'** sekaligus **menutup** Drawer secara programatik.

---

### 5: Penanganan Banyak Kategori Produk

**Tugas:** Modifikasi **Material Top Tabs** untuk mengakomodasi total **8 kategori** produk (e.g., `'Populer'`, `'Terbaru'`, `'Elektronik'`, `'Pakaian'`, `'Makanan'`, `'Otomotif'`, `'Hiburan'`, `'Perlengkapan Bayi'`). Aktifkan fungsi **scroll horizontal** pada tab bar agar semua kategori dapat diakses tanpa meluap dari layar perangkat. Pastikan **indikator** yang aktif tetap terlihat rapi di bawah label.

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

---

**Mobile App Development With React Native*
