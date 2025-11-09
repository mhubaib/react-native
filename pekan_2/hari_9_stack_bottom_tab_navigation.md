# Hari ke-9 - Screen Navigation: Stack Navigation & Bottom Tab Navigation

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

- Memahami sistem navigasi pada aplikasi mobile secara umum, termasuk pola standar seperti bottom tabs dan stack flow, serta mengenali jenis-jenis layar umum seperti Home, Profile, dan Modal, untuk merancang alur aplikasi yang intuitif dan mudah digunakan.

- Menerapkan Stack Navigator untuk alur navigasi bertingkat (hierarkis), seperti push/pop screen, tampilan modal, custom header, dan gesture untuk menutup layar, dengan pengoptimalan performa seperti penggunaan detachInactiveScreens.

- Menerapkan Bottom Tab Navigator untuk navigasi utama aplikasi, termasuk lazy loading, ikon dan badge kustom, pengaturan tampilan tab bar, serta pengaturan perilaku tombol back, dan mengintegrasikannya dengan Stack Navigator untuk navigasi bertingkat (nested).

- Mengintegrasikan kedua jenis navigator (misalnya tab dengan stack di dalamnya) menggunakan NavigationContainer dan screenOptions agar tema dan gesture tetap konsisten, serta memanfaatkan hook seperti useNavigation dan useRoute untuk routing yang dinamis.

- Menggunakan praktik terbaik seperti pengaturan opsi layar (options) secara dinamis berdasarkan route params.

- Menangani masalah umum (troubleshooting) seperti penyimpanan state navigasi atau kebocoran memori pada stack yang dalam, agar navigasi tetap stabil dan andal.

- Membangun prototipe aplikasi sederhana dengan alur multi-screen (misalnya tab Home dengan stack detail) yang siap dikembangkan lebih lanjut ke fitur seperti drawer atau alur autentikasi.

## 2. Materi Pembelajaran

Materi hari ini menjelaskan navigasi screen secara mendalam, dimulai dengan perkenalan konteks mobile untuk pemahaman holistik, diikuti detail Stack dan Bottom Tab Navigator. Navigasi di RN menggunakan library React Navigation, yang membungkus native APIs untuk transisi lancar. Semua navigator dibungkus dalam `<NavigationContainer>` untuk state management global.

### A. Perkenalan Sistem Navigasi di Mobile dan Jenis Screen Umum/Standar

**Tujuan:** Sebelum mendalami implementasi, pahami konteks navigasi mobile untuk merancang aplikasi yang mengikuti pola standar, mengurangi cognitive load pengguna, dan meningkatkan retention (studi Nielsen Norman Group 2025 menunjukkan 70% drop-off karena navigasi buruk).

Navigasi di aplikasi mobile sangat berbeda dari desktop karena layar ponsel yang kecil, interaksi berbasis sentuhan (touch), dan pengguna sering menggunakannya saat sedang bergerak (on-the-go). Ini membuat desain navigasi harus sederhana, cepat diakses, dan intuitif agar pengguna tidak frustrasi. Berdasarkan panduan terbaru seperti **Material Design 3** (untuk Android) dan **Human Interface Guidelines 2025** (untuk iOS), berikut adalah pola navigasi utama yang direkomendasikan. Saya akan jelaskan masing-masing secara rinci, termasuk kelebihan, kekurangan, dan tips penggunaan.

#### 1. Bottom Tab Navigation (Navigasi Tab di Bagian Bawah)

- **Penjelasan**: Ini adalah bar tab yang ditempatkan di bagian bawah layar, berisi 3-5 ikon utama untuk destinasi primer seperti Home, Pencarian, atau Profil. Tab ini selalu terlihat (persistent) dan mudah dijangkau dengan ibu jari pengguna saat memegang ponsel dalam posisi alami.
- **Kelebihan**:
  - Sangat intuitif karena mirip dengan kebiasaan pengguna sehari-hari.
  - Mudah diakses tanpa mengubah posisi tangan.
- **Kekurangan**:
  - Tidak cocok jika ada lebih dari 5 tab, karena layar jadi penuh—gunakan hanya ikon dengan label singkat.
- **Tips**: Ideal untuk aplikasi sederhana seperti media sosial. Contoh: Instagram menggunakan ini untuk beralih antar feed, reels, dan profil.

#### 2. Stack/Drill-Down Navigation (Navigasi Bertingkat atau Tumpukan Layar)

- **Penjelasan**: Navigasi ini seperti tumpukan kartu, di mana pengguna "mendorong" (push) layar baru dari yang sebelumnya, misalnya dari Home ke Detail Produk, lalu ke Edit. Setiap layar baru ditumpuk di atas yang lama, dan pengguna bisa kembali dengan tombol back atau gerakan geser.
- **Kelebihan**:
  - Cocok untuk alur linear, seperti membaca artikel lalu membagikannya.
  - Memberi rasa kemajuan yang jelas.
- **Kekurangan**:
  - Jika terlalu dalam (lebih dari 5 tingkat), pengguna bisa bingung dan tersesat—seperti labirin.
- **Tips**: Batasi kedalaman maksimal 4-5 level. Contoh: Aplikasi e-commerce seperti Tokopedia, di mana Anda mulai dari katalog, lalu detail barang, keranjang, dan checkout.

#### 3. Hamburger/Drawer Navigation (Menu Samping atau Drawer)

- **Penjelasan**: Menu tersembunyi yang muncul dengan geseran dari tepi layar (swipe dari kiri atau kanan), sering diwakili ikon hamburger (tiga garis). Digunakan untuk opsi sekunder seperti pengaturan atau bantuan.
- **Kelebihan**:
  - Menghemat ruang layar utama, sehingga konten utama lebih luas.
- **Kekurangan**:
  - Karena tersembunyi, pengguna mungkin lupa ada menu itu—kurang mudah ditemukan (kurang discoverable).
- **Tips**: Jangan gunakan untuk navigasi utama; simpan untuk fitur tambahan. Hindari jika aplikasi Anda butuh akses cepat. Contoh: Gmail menggunakan drawer untuk label email.

#### 4. Modal/Sheets Navigation (Jendela Pop-up atau Lembar Geser)

- **Penjelasan**: Ini adalah lapisan overlay yang muncul di atas konten utama untuk tugas sementara, seperti notifikasi, formulir login, atau konfirmasi pembelian. Pengguna bisa menutupnya dengan geser ke bawah (swipe-to-dismiss).
- **Kelebihan**:
  - Membuat pengguna fokus pada satu tugas tanpa gangguan.
  - Cepat dan tidak mengubah alur utama.
- **Kekurangan**:
  - Menutupi seluruh konten di belakangnya, yang bisa mengganggu jika terlalu sering digunakan.
- **Tips**: Gunakan hanya untuk interupsi penting, dan pastikan ada tombol tutup yang jelas. Contoh: Aplikasi banking seperti BCA Mobile, yang menggunakan sheet untuk verifikasi transaksi.

#### 5. Gestures & Tabs Hybrid (Kombinasi Gerakan dan Tab)

- **Penjelasan**: Menggabungkan tab dasar dengan gerakan sentuhan lanjutan, seperti geser antar tab atau navigasi suara/gerak di aplikasi AR/VR. Tren di 2025: Integrasi AI untuk prediksi gerakan, membuat navigasi lebih alami.
- **Kelebihan**:
  - Fleksibel dan modern, cocok untuk aplikasi kompleks.
  - Mengurangi ketergantungan pada tombol, lebih cepat untuk pengguna berpengalaman.
- **Kekurangan**:
  - Bisa membingungkan pemula jika gerakan tidak konsisten.
- **Tips**: Uji dengan pengguna untuk memastikan intuitif. Contoh: Aplikasi fitness seperti Nike Training Club, yang gabungkan tab dengan swipe untuk rutinitas latihan, dan tren baru di VR seperti navigasi suara di Apple Vision Pro.

Dengan mengikuti pola-pola ini, aplikasi mobile Anda akan lebih user-friendly dan sesuai standar terkini. Jika aplikasi Anda kompleks, kombinasikan beberapa pola (misalnya, bottom tabs dengan stack). Selalu uji dengan pengguna nyata untuk memastikan navigasi terasa alami!

**Jenis Screen Umum/Standar di Aplikasi:** Aplikasi mobile biasanya punya 5-10 screen types, disusun hierarkis untuk onboarding → core → support. Berikut tabel standar berdasarkan pola e-commerce/social apps (sumber: UXPin & Justinmind 2025):

| Jenis Screen | Deskripsi & Tujuan | Contoh Penggunaan | Pola Navigasi Umum |
|--------------|--------------------|-------------------|---------------------|
| **Splash/Onboarding** | Layar awal loading/pertama kali; intro features. | Logo + progress bar; swipe tutorial. | Auto-redirect ke login/home setelah 3-5s. |
| **Auth/Login/Signup** | Form autentikasi; social login options. | Email/password, OAuth buttons. | Stack: Splash → Login → Verification; modal untuk forgot password. |
| **Home/Dashboard** | Overview utama; quick actions/cards. | Feed, search bar, recent items. | Bottom tab: Home (default); nested stack untuk detail. |
| **Profile/Settings** | User info, preferences, account management. | Avatar, edit form, logout. | Bottom tab: Profile; stack ke sub-screens seperti Privacy. |
| **Detail/Product** | Info mendalam satu item; actions (buy/share). | Images, desc, reviews. | Stack push dari home/search; back ke parent. |
| **Search/Explore** | Query input, results list. | Filter, infinite scroll. | Bottom tab: Search; stack ke detail results. |
| **Cart/Checkout** | Shopping flow; payment integration. | Items list, total, confirm. | Stack: Cart → Address → Payment → Success; modal untuk promo. |
| **Modals/Alerts** | Interruptive tasks; confirmation dialogs. | Error messages, share sheets. | Modal presentation; swipe dismiss. |
| **Notifications** | In-app alerts; push handling. | List unread, mark read. | Stack dari home; tab jika persistent. |

**Pola Umum Aplikasi Standar:** Onboarding → Auth → Home (tab root) dengan nested stacks untuk flows (e.g., Home tab → Product Detail stack).

### B. Stack Navigation: Alur Hierarkis untuk Flows Linear

**Tujuan:** Stack Navigator mengelola "tumpukan" screens, ideal untuk drill-down (e.g., Home → Detail → Edit), dengan transisi native dan gesture support. Di v7, fokus pada custom animations dan memory optimization.

**Setup Dasar:** Instal `@react-navigation/native-stack`; wrap di `<NavigationContainer>`; gunakan `createNativeStackNavigator()` untuk native performance.

**Props Kunci (Stack.Navigator & Screen, v7.x):**

| Prop | Tipe | Deskripsi | Default | Catatan Platform & Best Practices |
|------|------|-----------|---------|----------------------------------|
| `initialRouteName` | string | Route awal stack. | First screen | Set untuk deep linking; gunakan dengan params untuk personalized start. |
| `screenOptions` | object/function | Global config (header, gestures). | {} | Function: `({ route }) => ({ title: route.params?.name })`; override per screen. |
| `detachInactiveScreens` | boolean | Hapus inactive screens dari hierarchy. | true | Optimasi memori di deep stacks; false untuk transparent modals. |
| `headerMode` | 'float'/'screen' | Header animasi: float (independent) atau screen (with card). | Platform default | iOS: 'float' untuk smooth; Android: 'screen' untuk material. |
| `mode` | 'card'/'modal'/'transparentModal' | Presentation type. | 'card' | 'transparentModal': Keep previous visible; gunakan untuk overlays. |
| `gestureEnabled` | boolean | Swipe-to-dismiss. | true (iOS)/false (Android) | Custom: `gestureDirection: 'horizontal-inverted'`; disable di forms. |
| `presentation` | 'card'/'modal'/'transparentModal' (per screen) | Screen-specific mode. | Inherit | 'modal': Full-sheet iOS; kombinasikan dengan cardOverlay untuk blur. |
| `title` | string | Header title. | Screen name | Dynamic: Function dengan route.params untuk personalized headers. |
| `headerShown` | boolean | Tampilkan/tidak header. | true | false untuk full-bleed screens; custom dengan header prop. |
| `headerStyle` | ViewStyle | Header background/elevation. | {} | { backgroundColor: '#007AFF', elevation: 2 }; platform select untuk shadow. |
| `headerTintColor` | Color | Icon/teks color. | Platform default | '#fff' untuk dark themes; accessibility: Min contrast 4.5:1. |
| `headerBackTitleVisible` | boolean | Tampilkan back title. | true (iOS)/false (Android) | false untuk clean UI; custom backImage untuk icons. |
| `animation` | 'default'/'fade'/'slide_from_right'/etc. | Transisi preset. | 'default' | 'fade' untuk quick switches; custom transitionSpec untuk spring/timing. |
| `transitionSpec` | {open/close: TimingConfig/SpringConfig} | Custom anim timing. | Default | { open: { animation: 'spring', config: { stiffness: 1000 } } }; RN 0.75+: GPU-accelerated. |
| `cardStyleInterpolator` | function | Custom card anim (opacity/translate). | Default | ({ current }) => ({ cardStyle: { opacity: current.progress } }); untuk fade. |
| `listeners` | object | Event handlers (transitionStart, gestureEnd). | {} | useEffect di screen: navigation.addListener('focus', onFocus); untuk analytics. |

**Pola Penggunaan:** Push: `navigation.navigate('Detail', { id: 123 })`; Pop: `navigation.goBack()`; Replace: `navigation.replace('Edit')`. Nested: Tabs dengan stack di tab (e.g., HomeStack di Home tab). Pola modal: `presentation: 'modal'` untuk bottom sheets.

**Praktik Terbaik (2025):** Gunakan native-stack untuk performance (vs @stack); dynamic options dengan route.params; enable freezeOnBlur via react-native-screens.enableFreeze(); test gestures di device. Hindari deep stacks (>5); gunakan popToTop() untuk reset flows.

**Pertimbangan Platform:** iOS: Native slide + swipe; Android: Fade/reveal + back button. v7: Better foldable support dengan per-window stacks.

### C. Bottom Tab Navigation: Navigasi Primer untuk Multi-Destinasi

**Tujuan:** Bottom Tab Navigator menyediakan persistent tabs bawah untuk akses cepat ke 3-5 sections utama, dengan lazy loading untuk efisiensi. Di v7, dukung custom bars dan shifting animations.

**Setup Dasar:** Instal `@react-navigation/bottom-tabs`; gunakan `createBottomTabNavigator()`.

**Props Kunci (Tab.Navigator & Screen, v7.x):**

| Prop | Tipe | Deskripsi | Default | Catatan Platform & Best Practices |
|------|------|-----------|---------|----------------------------------|
| `initialRouteName` | string | Tab awal. | First tab | Set berdasarkan user prefs (e.g., last visited). |
| `screenOptions` | object/function | Global tab/header config. | {} | Function: `({ route }) => ({ tabBarIcon: getIcon(route.name) })`; theme-aware. |
| `tabBar` | Component | Custom tab bar renderer. | Default | (props) => <CustomBar {...props} />; gunakan untuk FAB integration. |
| `backBehavior` | 'firstRoute'/'initialRoute'/'order'/'history'/'none' | Back button action. | 'history' | 'initialRoute' untuk reset ke tab pertama; 'none' di root. |
| `detachInactiveScreens` | boolean | Hapus inactive tabs. | true | Optimasi; false untuk persistent state di tabs. |
| `lazy` | boolean | Mount tab saat first focus. | true | false untuk pre-load heavy tabs (e.g., maps). |
| `freezeOnBlur` | boolean | Stop re-render inactive tabs. | false | true untuk battery saving; auto via react-native-screens. |
| `popToTopOnBlur` | boolean | Reset nested stack saat leave tab. | false | true untuk clean flows; cegah stack buildup. |
| `tabBarStyle` | ViewStyle | Bar container (height, bg). | Platform default | { position: 'absolute', height: 60 }; paddingBottom untuk safe area. |
| `tabBarActiveTintColor` | Color | Active icon/label color. | Primary | '#007AFF'; accessibility: High contrast. |
| `tabBarInactiveTintColor` | Color | Inactive color. | Muted | '#8E8E93'; gunakan opacity 0.5 untuk subtle. |
| `tabBarHideOnKeyboard` | boolean | Sembunyikan saat keyboard muncul. | false | true untuk form-heavy tabs; iOS default behavior. |
| `tabBarBadge` | string/number | Badge count. | null | Dynamic: unreadMessages; style dengan tabBarBadgeStyle. |
| `tabBarIcon` | function ({focused, color, size}) | Icon renderer. | null | <Ionicons name={focused ? 'home' : 'home-outline'} size={size} color={color} />; Expo Vector Icons. |
| `tabBarLabel` | string/Component | Label text/node. | Route title | Function untuk dynamic (e.g., ({ focused }) => focused ? 'Home' : 'H'); fontSize 12. |
| `tabBarLabelPosition` | 'below-icon'/'beside-icon'/'above-icon' | Label posisi. | 'below-icon' | 'beside-icon' untuk compact; tren 2025: Icon-only di landscape. |
| `animation` | 'none'/'fade'/'shift' | Tab switch transisi. | 'none' | 'shift': Icon move; 'fade' untuk smooth content change. |
| `transitionSpec` | TransitionSpec | Custom timing. | Default | { open: { duration: 200 } }; kombinasikan dengan sceneStyleInterpolator. |

**Pola Penggunaan:** Nested: <Tab.Screen name="Home" component={HomeStack} />; Badge: options={{ tabBarBadge: state.unread }}. Custom bar: Emit 'tabPress' events untuk analytics.

**Praktik Terbaik (2025):** Batasi 4-5 tabs; gunakan icons + labels; lazy=true untuk perf; shifting=false untuk non-iOS. Integrasi Stack: Nested untuk detail flows. Test back: 'history' untuk intuitive.

**Pertimbangan Platform:** iOS: UIKit-style dengan labels; Android: Material icons. v7: Sidebar tabs di foldables (tabBarPosition: 'left').

**Integrasi Keseluruhan:** Gunakan Stack di dalam Tabs untuk hybrid (e.g., Home tab → Stack detail); NavigationContainer untuk theme persistence.

## 3. Contoh Implementasi

### A. Contoh Dasar: Stack Navigator Sederhana

```jsx
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import HomeScreen from './screens/HomeScreen';
import DetailScreen from './screens/DetailScreen';

const Stack = createNativeStackNavigator();

const App = () => (
  <NavigationContainer>
    <Stack.Navigator
      initialRouteName="Home"
      screenOptions={{
        headerStyle: { backgroundColor: '#007AFF' },
        headerTintColor: '#fff',
        gestureEnabled: true,
      }}
    >
      <Stack.Screen
        name="Home"
        component={HomeScreen}
        options={{ title: 'Beranda' }}
      />
      <Stack.Screen
        name="Detail"
        component={DetailScreen}
        options={({ route }) => ({
          title: route.params?.name || 'Detail',
          presentation: route.params?.modal ? 'modal' : 'card',
        })}
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
      <Text>Halaman Home</Text>
      <Button
        title="Buka Detail"
        onPress={() => navigation.navigate('Detail', { name: 'Produk A' })}
      />
      <Button
        title="Buka Modal"
        onPress={() => navigation.navigate('Detail', { name: 'Modal', modal: true })}
      />
    </View>
  );
};

export default HomeScreen;
```

**Penjelasan:** Stack dasar dengan dynamic title/modal; gesture swipe back.

### B. Contoh Interaktif: Bottom Tab Navigator

```jsx
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import Ionicons from '@expo/vector-icons/Ionicons';
import HomeScreen from './screens/HomeScreen';
import ProfileScreen from './screens/ProfileScreen';

const Tab = createBottomTabNavigator();

const App = () => (
  <NavigationContainer>
    <Tab.Navigator
      initialRouteName="Home"
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          let iconName;
          if (route.name === 'Home') iconName = focused ? 'home' : 'home-outline';
          else if (route.name === 'Profile') iconName = focused ? 'person' : 'person-outline';
          return <Ionicons name={iconName} size={size} color={color} />;
        },
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: 'gray',
        headerShown: false, // Hide header di tabs
        lazy: true,
      })}
    >
      <Tab.Screen name="Home" component={HomeScreen} options={{ tabBarBadge: 3 }} />
      <Tab.Screen name="Profile" component={ProfileScreen} />
    </Tab.Navigator>
  </NavigationContainer>
);
```

**Penjelasan:** Tabs dengan icons dynamic; badge di Home; lazy loading.

### C. Contoh Lanjutan: Hybrid Tabs + Nested Stack

```jsx
// App.js
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import Ionicons from '@expo/vector-icons/Ionicons';
import HomeScreen from './screens/HomeScreen';
import DetailScreen from './screens/DetailScreen';
import ProfileScreen from './screens/ProfileScreen';

const Tab = createBottomTabNavigator();
const Stack = createNativeStackNavigator();

const HomeStack = () => (
  <Stack.Navigator>
    <Stack.Screen name="Home" component={HomeScreen} options={{ title: 'Home' }} />
    <Stack.Screen name="Detail" component={DetailScreen} options={{ title: 'Detail' }} />
  </Stack.Navigator>
);

const App = () => (
  <NavigationContainer>
    <Tab.Navigator screenOptions={{ headerShown: false }}>
      <Tab.Screen
        name="HomeTab"
        component={HomeStack}
        options={{
          tabBarLabel: 'Home',
          tabBarIcon: ({ focused }) => <Ionicons name={focused ? 'home' : 'home-outline'} size={24} />,
        }}
      />
      <Tab.Screen
        name="Profile"
        component={ProfileScreen}
        options={{
          tabBarLabel: 'Profile',
          tabBarIcon: ({ focused }) => <Ionicons name={focused ? 'person' : 'person-outline'} size={24} />,
        }}
      />
    </Tab.Navigator>
  </NavigationContainer>
);

// screens/HomeScreen.js (dari contoh 3.1, tambah navigate ke Detail)
```

**Penjelasan:** Tabs root dengan nested Stack di Home; navigasi seamless antar flows.

### D. Contoh Tambahan: Custom Tab Bar Component

Untuk kasus di mana tab bar default tidak cukup (e.g., menambahkan FAB atau animasi custom), buat komponen custom yang menerima props dari Tab.Navigator. Ini memungkinkan kontrol penuh atas styling, gestures, dan integrasi (e.g., dengan Reanimated untuk smooth transitions).

**Setup:** Di Tab.Navigator, set `tabBar={(props) => <CustomTabBar {...props} />}`. Komponen harus handle `state`, `descriptors`, `navigation`, dan emit events seperti `onTabPress`.

```tsx
// components/CustomTabBar.js
import React from 'react';
import { View, TouchableOpacity, Text, StyleSheet } from 'react-native';
import Ionicons from '@expo/vector-icons/Ionicons';

const CustomTabBar = ({ state, descriptors, navigation }) => (
  <View style={styles.tabBar}>
    {state.routes.map((route, index) => {
      const { options } = descriptors[route.key];
      const label = options.tabBarLabel || route.name;
      const iconName = route.name === 'Home' ? 'home' : 'person'; // Custom logic

      const isFocused = state.index === index;

      const onPress = () => {
        const event = navigation.emit({
          type: 'tabPress',
          target: route.key,
          canPreventDefault: true,
        });

        if (!isFocused && !event.defaultPrevented) {
          navigation.navigate(route.name);
        }
      };

      const onLongPress = () => {
        navigation.emit({
          type: 'tabLongPress',
          target: route.key,
        });
      };

      return (
        <TouchableOpacity
          key={index}
          accessibilityRole="button"
          accessibilityState={isFocused ? { selected: true } : {}}
          accessibilityLabel={options.tabBarAccessibilityLabel}
          testID={options.tabBarTestID}
          onPress={onPress}
          onLongPress={onLongPress}
          style={[styles.tabItem, { backgroundColor: isFocused ? '#007AFF' : 'transparent' }]}
        >
          <Ionicons name={iconName} size={24} color={isFocused ? '#fff' : '#666'} />
          <Text style={[styles.tabLabel, { color: isFocused ? '#fff' : '#666' }]}>
            {label}
          </Text>
        </TouchableOpacity>
      );
    })}
  </View>
);

const styles = StyleSheet.create({
  tabBar: {
    flexDirection: 'row',
    backgroundColor: '#f8f8f8',
    borderTopWidth: 1,
    borderTopColor: '#e0e0e0',
    height: 70,
    justifyContent: 'space-around',
    paddingBottom: 10,
  },
  tabItem: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    paddingVertical: 10,
    borderRadius: 10,
    marginHorizontal: 5,
  },
  tabLabel: {
    fontSize: 12,
    marginTop: 4,
    fontWeight: '600',
  },
});

export default CustomTabBar;
```

**Integrasi ke App.js (Modifikasi dari 3.2):**

```jsx
// Di Tab.Navigator
<Tab.Navigator
  tabBar={(props) => <CustomTabBar {...props} />}
  screenOptions={{
    headerShown: false,
    lazy: true,
  }}
>
  <Tab.Screen name="Home" component={HomeScreen} />
  <Tab.Screen name="Profile" component={ProfileScreen} />
</Tab.Navigator>
```

**Penjelasan:** CustomTabBar menerima props navigator; handle press/longPress dengan emit events; styling kondisional (bg color saat focused). Ini memungkinkan ekstensi seperti animasi Reanimated atau FAB di tengah tabs. Test: Long press untuk debug; pastikan accessibilityRole untuk screen readers.

**Tips Debugging:** Gunakan `adb logcat` untuk stack traces; test deep links dengan `npx uri-scheme open exp://... -- /home`.

## 4. Rangkuman

Hari ke-9 memperkenalkan navigasi screen melalui perkenalan pola mobile (bottom tabs, stacks, modals) dan jenis screen standar (splash, home, detail, etc.), diikuti Stack Navigator untuk hierarki linear (push/pop, modals, gestures) dan Bottom Tab untuk akses primer (lazy tabs, icons, badges). Kunci: Declarative setup dengan NavigationContainer; dynamic options/hooks untuk personalization; nested patterns untuk kompleksitas. Integrasi ini ciptakan alur aplikasi yang intuitif, performant (detach/freeze), dan native, siap untuk auth/deep links di modul selanjutnya.

**Latihan Selanjutnya:** Bangun app dengan tabs (Home + Profile) dan nested stack di Home untuk detail; tambah modal confirmation.

**Referensi:**

- [Stack Navigator | React Navigation](https://reactnavigation.org/docs/stack-navigator/)
- [Bottom Tab Navigator | React Navigation](https://reactnavigation.org/docs/bottom-tab-navigator/)
- [Mobile Navigation Patterns: Pros and Cons - UXPin](https://www.uxpin.com/studio/blog/mobile-navigation-patterns-pros-and-cons/)
- [Mobile navigation: patterns and examples - Justinmind](https://www.justinmind.com/blog/mobile-navigation/)
- [Modern iOS Navigation Patterns - Frank Rausch](https://frankrausch.com/ios-navigation/)

## 5. Evaluasi Harian: Soal Praktik

`Lanjutan project Mini E-Commerce`

**Bangun struktur dasar aplikasi dengan navigasi(stack & bottom tab), sesuai dengan kriteria berikut:**

- On boarding screen (min: 2 screen) menggunakan stack navigasi untuk membantu pengguna memahami cara kerja aplikasi.
- Tempatkan screen product catalog sebagai screen utama di tab navigasi.
- Dan buat screen profile di tab navigasi sebagai screen tambahannya.

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```

---

**Mobile App Development With React Native*
