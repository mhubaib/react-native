# Hari ke-7: Fundamental Styling di React Native

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

- Memahami perbedaan styling inline vs StyleSheet, serta menerapkan StyleSheet.create() untuk gaya reusable dan performa lebih baik, menghindari overhead komputasi pada render.
- Mengimplementasikan layout responsif menggunakan konsep flexbox (flexDirection, justifyContent, alignItems), dimensi fleksibel (flex, Dimensions API), dan positioning (absolute/relative) untuk UI yang adaptif di berbagai ukuran layar.
- Mengelola elemen visual seperti warna (hex/rgb), opacity, border (radius, width), shadow (iOS shadow props vs Android elevation), font (family, weight), dan transform (scale, rotate) untuk desain yang konsisten dan native-feeling.
- Menerapkan styling platform-spesifik menggunakan Platform.select() atau Platform-specific extensions untuk menangani perbedaan iOS (e.g., shadowOpacity) vs Android (e.g., elevation).
- Menerapkan praktik terbaik seperti design system (konstanta warna/spacing), menghindari inline styles di loop, dan menggunakan PixelRatio untuk scaling, sambil memastikan aksesibilitas (e.g., high contrast colors).
- Membangun prototipe UI lengkap, seperti card e-commerce dengan shadow dan flex layout, yang siap untuk integrasi dengan komponen sebelumnya (e.g., FlatList styled).
- Menganalisis dan debug isu umum, seperti layout shift dari dimensi dinamis atau shadow tidak konsisten antar platform, untuk styling yang robust dan scalable.

## 2. Materi Pembelajaran

Materi hari ini menjelaskan fundamental styling secara mendalam, termasuk API utama, props kunci (dalam tabel), pola penggunaan, dan pertimbangan platform. Styling di React Native menggunakan objek JS (camelCase, bukan kebab-case CSS), diterapkan via prop `style` pada komponen core seperti View/Text. Hindari inline styles untuk performa; gunakan StyleSheet.create() untuk gaya statis yang di-cache. Per 2025, dokumentasi resmi react native menekankan integrasi dengan tools seperti NativeWind untuk Tailwind-like utility classes, tapi core tetap JS objects.

### A. StyleSheet API: Fondasi Styling Efisien

**Tujuan:** StyleSheet adalah utilitas untuk membuat gaya reusable yang dioptimalkan, mengurangi komputasi JS thread dengan ID hashing. Lebih cepat daripada inline objects. API ini memungkinkan abstraksi styling mirip CSS, meningkatkan kejelasan kode, dan mendukung static type checking terhadap properti native style. Semua method bersifat static dan diakses melalui `StyleSheet`. Berikut penjelasan lengkap semua method dan property di API StyleSheet (berdasarkan dokumentasi React Native versi 0.75+ per Oktober 2025).

#### Method-Method Utama

Berikut tabel lengkap method di StyleSheet API, termasuk deskripsi, parameter, tipe return, default, dan catatan:

| Method | Tipe Parameter & Return | Deskripsi | Default | Catatan & Contoh |
|--------|-------------------------|-----------|---------|------------------|
| `StyleSheet.compose(style1: Object, style2: Object): Object \| Object[]` | Parameter: `style1` (Object), `style2` (Object)<br>Return: `Object` atau `Object[]` | Menggabungkan dua objek style sehingga `style2` mengoverride konflik di `style1`. Jika salah satu falsy, return yang lain tanpa alokasi array, menjaga reference equality untuk optimasi `PureComponent`. | - | Digunakan internal untuk komposisi style; hindari alokasi tidak perlu.<br>Contoh: `StyleSheet.compose(baseStyle, { color: 'red' })` → override color. |
| `StyleSheet.create(styles: Object extends Record<string, ViewStyle \| ImageStyle \| TextStyle>): Object` | Parameter: `styles` (Object dengan key string ke style objects)<br>Return: `Object` (stylesheet dengan named entries) | Membuat objek stylesheet dari koleksi style bernama. Manfaat utama: static type checking terhadap properti native style, modularitas, dan readability. | - | Pusat API; pisahkan style dari render logic.<br>Contoh:<br>```jsx
| `StyleSheet.flatten(style: Array<Object extends Record<string, ViewStyle \| ImageStyle \| TextStyle>>): Object` | Parameter: `style` (Array style objects)<br>Return: `Object` (single aggregated style) | Meratakan array objek style menjadi satu objek gaya gabungan. | - | Berguna untuk gabung multiple sumber style (e.g., conditional).<br>Contoh: `StyleSheet.flatten([base, darkMode ? dark : light])` → satu objek untuk render. |
| `StyleSheet.setStyleAttributePreprocessor(property: string, process: (propValue: any) => any)` | Parameter: `property` (string, e.g., 'color'), `process` (function)<br>Return: `void` | Mengatur fungsi preprocessor untuk nilai properti style spesifik. Digunakan internal untuk proses nilai seperti `color` atau `transform`. | - | Fitur eksperimental; potensi breaking changes. Hanya gunakan jika opsi lain habis; bukan untuk penggunaan umum.<br>Contoh: `StyleSheet.setStyleAttributePreprocessor('color', (value) => value.toUpperCase());` (untuk custom processing). |

#### Property-Property Utama

Berikut tabel lengkap property di StyleSheet API:

| Property | Tipe | Deskripsi | Default | Catatan & Contoh |
|----------|------|-----------|---------|------------------|
| `StyleSheet.absoluteFill` | `Object` | Objek style predefined untuk `position: 'absolute', left: 0, top: 0, right: 0, bottom: 0`. Sederhanakan overlay full-screen dan kurangi duplikasi style. | - | Gunakan langsung atau dalam create().<br>Contoh: `const overlay = StyleSheet.absoluteFill;` → `<View style={overlay} />`. |
| `StyleSheet.absoluteFillObject` | `Object` | Versi `absoluteFill` yang bisa diekstensikan atau dimodifikasi saat buat style custom. Izinkan fine-tuning positioning absolute. | - | Saat `absoluteFill` terlalu generik.<br>Contoh:<br>```jsx<br>const custom = StyleSheet.create({<br>  overlay: {<br>    ...StyleSheet.absoluteFillObject,<br>    backgroundColor: 'rgba(0,0,0,0.5)',<br>  },<br>});<br>``` |
| `StyleSheet.hairlineWidth` | `number` | Konstanta lebar garis tipis di platform (selalu round pixels untuk rendering crisp). Cocok untuk border atau divider native. | Platform-dependent (e.g., 1px scaled) | Nilai variatif antar platform/density; mungkin tak terlihat di simulator downscaled.<br>Contoh: `borderBottomWidth: StyleSheet.hairlineWidth,` → garis halus. |

**Pola Penggunaan:** Import `StyleSheet` dari 'react-native'; definisikan di luar component untuk reuse. Pola: Konstanta design system di file terpisah (e.g., colors.js: { primary: '#007AFF' }). Gunakan `create()` untuk named styles; `flatten()` untuk dynamic; `compose()` internal; `absoluteFill` untuk overlay cepat. Hindari `setStyleAttributePreprocessor` kecuali eksperimen.

**Praktik Terbaik:** Selalu gunakan create() untuk perf (hindari inline di renderItem FlatList). Per 2025, kombinasikan dengan useColorScheme() untuk dark mode. Hindari mutasi styles post-create. API ini mempromosikan clean, maintainable, type-safe styling.

### B. Flexbox: Layout Responsif Dasar

**Tujuan:** Flexbox adalah model layout utama, mirip CSS Flexbox, untuk mengatur elemen secara fleksibel (row/column, alignment, spacing).

**Props Kunci (ViewStyle):**

| Prop | Tipe | Deskripsi | Default | Catatan Platform |
|------|------|-----------|---------|------------------|
| `flexDirection` | 'row' | 'column' | 'row-reverse' | 'column-reverse' | 'column' | Arah utama axis. | 'column' | 'row' untuk horizontal list. |
| `justifyContent` | 'flex-start' | 'center' | 'flex-end' | 'space-between' | 'space-around' | 'space-evenly' | 'flex-start' | Distribusi di main axis. |
| `alignItems` | 'flex-start' | 'center' | 'flex-end' | 'stretch' | 'baseline' | 'flex-start' | Alignment di cross axis. |
| `flex` | number | Proporsi ruang (e.g., 1 untuk equal). | 0 | >0 untuk grow/shrink. |
| `flexWrap` | 'wrap' | 'nowrap' | 'wrap-reverse' | 'nowrap' | Wrap item jika overflow. |
| `alignSelf` | 'auto' | 'flex-start' | ... (sama justify) | 'auto' | Override alignItems per child. |
| `flexGrow` | number | Prioritas grow. | 0 | RN 0.72+ dukung lebih baik. |
| `flexShrink` | number | Prioritas shrink. | 1 | - |
| `flexBasis` | number | string | 'auto' | Basis size sebelum grow/shrink. |

**Pola Penggunaan:** Root View: { flex: 1 } untuk full screen. Pola centering: { justifyContent: 'center', alignItems: 'center' }. Nested flex untuk card/grid.

**Praktik Terbaik:** Gunakan flex:1 di container utama; test di device untuk overflow.

### C. Dimensi dan Positioning: Ukuran dan Posisi Elemen

**Tujuan:** Kontrol ukuran (width/height) dan posisi (absolute untuk overlay, relative default).

**Props Kunci:**

| Prop | Tipe | Deskripsi | Default | Catatan |
|------|------|-----------|---------|---------|
| `width` / `height` | number | string ('%') | 'auto' | Fixed px atau % dari parent. |
| `minWidth` / `maxWidth` | number | string | undefined | Batas ukuran. |
| `flexBasis` | number | string | 'auto' | Basis flex item. |
| `position` | 'relative' | 'absolute' | 'relative' | Absolute butuh top/left/right/bottom. |
| `top` / `left` / `right` / `bottom` | number | string | undefined | Offset dari edge (px atau %). |
| `zIndex` | number | undefined | -1 (Android), 0 (iOS) | Stacking order; hati-hati perf. |

**API Tambahan:** `Dimensions.get('window')` untuk screen size; `PixelRatio.get()` untuk density scaling (e.g., fontSize: 16 * PixelRatio.getFontScale()).

**Pola Penggunaan:** Responsif: width: '100%', height: Dimensions.get('window').height * 0.5. Pola absolute: Overlay modal dengan position: 'absolute', top:0, left:0.

**Praktik Terbaik:** Gunakan flex daripada fixed width untuk adaptif; scale dengan PixelRatio untuk retina displays. Hindari % di height tanpa flex parent.

### D. Warna, Opacity, dan Border: Elemen Visual Dasar

**Tujuan:** Atur tampilan dasar dengan warna (hex '#RRGGBB', rgb(255,0,0), 'red'), opacity (0-1), dan border (rounded corners).

**Props Kunci:**

| Prop | Tipe | Deskripsi | Default | Catatan |
|------|------|-----------|---------|---------|
| `backgroundColor` | Color | Latar belakang. | 'transparent' | Dukung transparent. |
| `color` | Color | Warna teks. | 'black' (Text) | - |
| `opacity` | number (0-1) | Transparansi. | 1 | Efek di seluruh subtree. |
| `borderWidth` | number | Ketebalan border. | 0 | - |
| `borderColor` | Color | Warna border. | 'black' | - |
| `borderRadius` | number | Sudut membulat. | 0 | borderTopLeftRadius, dll. untuk spesifik. |
| `borderStyle` | 'solid' | 'dotted' | 'dashed' | 'solid' | Android only untuk dotted/dashed. |

**Pola Penggunaan:** Button: { backgroundColor: colors.primary, borderRadius: 8 }. Pola gradient: Gunakan library seperti react-native-linear-gradient.

**Praktik Terbaik:** Definisikan tema colors di constants; gunakan opacity untuk hover-like effects di Pressable.

### E. Shadow dan Elevation: Efek Depth

**Tujuan:** Tambah bayangan untuk 3D feel; iOS gunakan shadow props, Android elevation.

**Props Kunci:**

| Prop | Tipe | Deskripsi | Default | Catatan Platform |
|------|------|-----------|---------|------------------|
| `elevation` | number | Ketinggian shadow Android. | 0 | Android only; auto shadow berdasarkan value. |
| `shadowColor` | Color | Warna shadow iOS. | 'black' | iOS only. |
| `shadowOffset` | {width: number, height: number} | Offset shadow. | {width:0, height:1} | iOS; height >0 untuk bawah. |
| `shadowOpacity` | number (0-1) | Intensitas shadow. | 0.2 (Android inferred) | iOS. |
| `shadowRadius` | number | Blur radius. | 1.41 (Android inferred) | iOS. |

**Pola Penggunaan:** Card: Platform.select({ ios: { shadow... }, android: { elevation: 3 } }). Pola konsisten: Gunakan library seperti react-native-shadow-2 untuk cross-platform.

**Praktik Terbaik:** Test di device (emulator shadow kurang akurat). Per 2025, gunakan elevation 1-5 untuk subtle depth.

### F. Font dan Transform: Teks dan Animasi Dasar

**Tujuan:** Styling teks (fontFamily custom via assets) dan transform untuk efek (scale, rotate).

**Props Kunci (TextStyle):**

| Prop | Tipe | Deskripsi | Default | Catatan |
|------|------|-----------|---------|---------|
| `fontFamily` | string | Font custom (load via @font-face). | 'System' | Butuh link assets di info.plist/Android. |
| `fontSize` | number | Ukuran font (px scaled). | 14 | Scale dengan PixelRatio. |
| `fontWeight` | 'normal' | 'bold' | number (100-900) | 'bold' = 700. |
| `fontStyle` | 'normal' | 'italic' | 'normal' | - |
| `textAlign` | 'left' | 'center' | 'right' | 'justify' | 'auto' | - |

**Transform Props (ViewStyle):**

| Prop | Tipe | Deskripsi | Default | Catatan |
|------|------|-----------|---------|---------|
| `transform` | Array<{translateX: number}, {scale: number}, {rotate: string('deg')}> | Array transformasi. | [] | Urutan penting; gunakan deg/rad. |

**Pola Penggunaan:** Custom font: Download .ttf, tambah ke project, gunakan fontFamily: 'MyFont'. Pola rotate: [{ rotate: '45deg' }] untuk icon.

**Praktik Terbaik:** Load font async dengan expo-font; batasi transform untuk hindari GPU overhead.

### G. Platform-Specific dan Best Practices

**Tujuan:** Adaptasi styling per OS dengan Platform module.

**API:** `import { Platform } from 'react-native';` Style: Platform.select({ ios: { shadow... }, android: { elevation: 2 } }).

**Praktik Terbaik (Per 2025 Guides):**

- Bangun design system: Konstanta untuk spacing (e.g., { small: 8, medium: 16 }), colors, typography.
- Responsif: Gunakan Dimensions.addEventListener untuk orientation change.
- Perf: StyleSheet > inline; hindari styles di render loops.
- Aksesibilitas: Gunakan dynamicTypeSize untuk font scaling; high contrast ratios.
- Tools: NativeWind untuk utility classes; tambah di RN 0.75+ untuk Tailwind support.

**Integrasi Keseluruhan:** Gabungkan flexbox + StyleSheet untuk layout, shadow + platform select untuk visual.

## 3. Contoh Implementasi

Kode siap di Expo/simulator. Integrasikan untuk UI e-commerce seperti tugas sebelumnya.

### A. Contoh Dasar: Layout Centered dengan Flexbox dan StyleSheet

```jsx
import React from 'react';
import { View, Text, StyleSheet, Dimensions } from 'react-native';

const { height } = Dimensions.get('window');

const CenteredLayout = () => (
  <View style={styles.container}>
    <View style={styles.card}>
      <Text style={styles.title}>Welcome to RN Styling</Text>
      <Text style={styles.subtitle}>Flexbox in action</Text>
    </View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#f0f0f0',
  },
  card: {
    width: '80%',
    padding: 20,
    backgroundColor: 'white',
    borderRadius: 10,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.25,
    shadowRadius: 3.84,
    elevation: 5,
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
    textAlign: 'center',
  },
});

export default CenteredLayout;
```

**Penjelasan:** Flexbox center card; shadow cross-platform; responsif width %.

### B. Contoh Interaktif: Button Responsif dengan Transform dan Platform Styles

```jsx
import React from 'react';
import { View, Text, Pressable, StyleSheet, Platform } from 'react-native';

const ResponsiveButton = () => {
  const buttonStyle = Platform.select({
    ios: { shadowColor: '#000', shadowOffset: { width: 0, height: 2 }, shadowOpacity: 0.2, shadowRadius: 4 },
    android: { elevation: 3 },
  });

  return (
    <View style={styles.container}>
      <Pressable
        style={({ pressed }) => [
          styles.button,
          buttonStyle,
          { transform: [{ scale: pressed ? 0.95 : 1 }] },
        ]}
        onPress={() => console.log('Pressed')}
      >
        <Text style={styles.buttonText}>Tap Me</Text>
      </Pressable>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  button: {
    backgroundColor: '#007AFF',
    paddingHorizontal: 30,
    paddingVertical: 12,
    borderRadius: 25,
    minWidth: 120,
  },
  buttonText: { color: 'white', fontSize: 16, fontWeight: '600' },
});

export default ResponsiveButton;
```

**Penjelasan:** Platform shadow; transform scale on press; flex minWidth responsif.

### C. Contoh Lanjutan: Card Produk dengan Full Styling

```jsx
import React from 'react';
import { View, Text, Image, StyleSheet, Dimensions, Platform, PixelRatio } from 'react-native';

const { width } = Dimensions.get('window');
const ITEM_WIDTH = width * 0.9;

const ProductCard = ({ product }) => (
  <View style={styles.card}>
    <Image source={{ uri: product.imageUrl }} style={styles.image} />
    <View style={styles.info}>
      <Text style={styles.name}>{product.name}</Text>
      <Text style={styles.price}>Rp {product.price.toLocaleString()}</Text>
      <Text style={styles.desc} numberOfLines={2}>{product.description}</Text>
    </View>
  </View>
);

const styles = StyleSheet.create({
  card: {
    width: ITEM_WIDTH,
    backgroundColor: 'white',
    borderRadius: 12,
    marginVertical: 8,
    overflow: 'hidden',
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 4 },
        shadowOpacity: 0.1,
        shadowRadius: 8,
      },
      android: { elevation: 4 },
    }),
  },
  image: { width: '100%', height: 200, resizeMode: 'cover' },
  info: { padding: 16 },
  name: { fontSize: 18, fontWeight: '600', marginBottom: 4 },
  price: { fontSize: 16, color: '#007AFF', fontWeight: 'bold', marginBottom: 4 },
  desc: { fontSize: 14, color: '#666', lineHeight: 20 * PixelRatio.getFontScale() },
});

export default ProductCard;
```

**Penjelasan:** Full card dengan borderRadius, shadow platform, font scaled, image cover.

**Tips Debugging:** Gunakan React DevTools untuk inspect styles; test orientation dengan Dimensions listener.

## 4. Rangkuman

Hari ke-7 membangun fondasi styling melalui StyleSheet untuk efisiensi, flexbox untuk layout responsif, dimensi/positioning untuk adaptasi, warna/border/opacity untuk visual dasar, shadow/elevation untuk depth, font/transform untuk teks/animasi, dan platform-specific untuk konsistensi. Kunci: Gunakan JS objects camelCase, prioritaskan StyleSheet.create(), bangun design system, dan optimasi perf dengan avoid inline. Integrasi ini ciptakan UI mobile yang profesional, accessible, dan scalable, siap untuk tema dark mode atau library seperti NativeWind per 2025.

**Referensi :**

- [Style · React Native](https://reactnative.dev/docs/style)
- [The Complete React Native Styling Guide Every Developer Needs](https://medium.com/@arsdev/the-complete-react-native-styling-guide-every-developer-needs-4a29ab0f4e36)
- [Starting Fresh with React Native: What are the foundational ...](https://github.com/orgs/community/discussions/166868)
- [Top React Native ESSENTIALS Tech Stack for 2025](https://dev.to/martygo/react-native-kit-updates-topics-you-must-know-57bi)
- [React Native styling tutorial with examples](https://blog.logrocket.com/react-native-styling-tutorial-examples/)
- [React Fundamentals - React Native](https://reactnative.dev/docs/intro-react)

## 5. Evaluasi Harian: Soal Praktik

Buat layar "Flexbox Playground" dengan komponen berikut:

1. Buat 3 kotak berwarna berbeda (merah, biru, hijau) dengan ukuran yang sama
2. Implementasikan 3 tombol yang mengubah flexDirection container (row, column, row-reverse)
3. Implementasikan 3 tombol yang mengubah justifyContent (flex-start, center, space-between)
4. Implementasikan 3 tombol yang mengubah alignItems (flex-start, center, stretch)
5. Gunakan StyleSheet.create() untuk semua styling dan hindari inline styles

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

---

**Mobile App Development With React Native*
