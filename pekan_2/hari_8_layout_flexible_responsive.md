# Hari ke-8 - Membuat Layout Flexible & Responsive di React Native

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

- Menerapkan teknik conditional sizing dan percentage-based layouts untuk membuat UI yang otomatis menyesuaikan dengan screen width/height, menghindari fixed pixels yang menyebabkan overflow atau white space berlebih di berbagai perangkat.
- Mengimplementasikan orientation dan foldable handling menggunakan Dimensions API (useWindowDimensions hook dan event listeners) untuk switch layout secara dinamis, seperti dari single-column portrait ke multi-column landscape.
- Mengintegrasikan react-native-safe-area-context untuk dynamic safe area insets, memastikan konten tidak ter-clip oleh notch, status bar, atau gesture navigation, sambil menjaga Flexbox layout tetap fleksibel (e.g., padding adaptif).
- Menerapkan praktik terbaik seperti gap utilities dalam Flexbox untuk spacing responsif, media query-like logic via width thresholds, dan memoization untuk menghindari re-calculation berlebih saat rotasi atau unfold.
- Mengelola platform-specific behaviors, seperti iOS Dynamic Island insets vs Android gesture nav, untuk UI edge-to-edge yang konsisten tanpa layout shift.
- Membangun prototipe responsif seperti e-commerce grid yang beradaptasi dari 1-kolom di phone portrait ke 4-kolom di tablet landscape, dengan safe areas terintegrasi.
- Menganalisis dan debug isu seperti misalignment saat orientation change atau clipping di foldables, untuk layout yang performant (60fps) dan user-centric.

## 2. Materi Pembelajaran

Materi hari ini menjelaskan teknik design flexible & responsive secara mendalam, dengan fokus pada strategi adaptasi daripada konsep dasar Flexbox. Teknik melibatkan kombinasi Dimensions untuk awareness, conditional logic untuk variasi, dan react-native-safe-area-context untuk insets dinamis. Per 2025, best practices menekankan relative units (%/vw-like via width) dan hooks untuk zero-config re-renders. Integrasikan dengan StyleSheet dari Hari 7 untuk performance.

### A. Teknik Conditional Sizing dan Percentage-Based Layouts: Adaptasi Berdasarkan Screen Size

**Tujuan:** Teknik ini memungkinkan layout yang "self-adjusting" tanpa fixed dimensi, menggunakan percentage widths/heights dan thresholds untuk simulasi CSS media queries, sehingga UI scale di phone (e.g., 320px) ke tablet (e.g., 1024px).

**Props dan Hooks Kunci (untuk Integrasi Flexbox):**

| Teknik/Prop | Tipe | Deskripsi | Default | Catatan & Pola Penggunaan |
|-------------|------|-----------|---------|---------------------------|
| `width`/`height` | string ('%') atau number | Relative sizing: '50%' dari parent atau screen width. | 'auto' | Pola: width: `${100 / numCols}%` di Flexbox row; hindari px—gunakan untuk cards yang shrink di small screens. |
| `flexBasis` | string/number ('%') | Initial size flex item; kombinasikan dengan flexWrap untuk wrap responsif. | 'auto' | Pola: flexBasis: width > 600 ? '25%' : '50%' untuk grid; auto-adjust kolom berdasarkan available space. |
| Conditional via `useWindowDimensions()` | Hook: {width, height} | Threshold logic: if (width < 400) single layout else multi. | - | Pola: const isTablet = width > 768; style: { flexDirection: isTablet ? 'row' : 'column' }; re-render otomatis saat resize. |
| `gap`/`rowGap`/`columnGap` | number/string ('%') | Spacing responsif di Flexbox; scale dengan screen untuk proportional gutters. | 0 | Pola: gap: width * 0.02 (2% screen) untuk adaptive padding; RN 0.75+: Optimized untuk wrapped layouts, cegah margin collapse. |

**Pola Penggunaan:** Gunakan useWindowDimensions() di root component untuk global thresholds; apply conditional styles di child Flexbox (e.g., numColumns di FlatList = Math.floor(width / 200)). Pola hybrid: Percentage + flexGrow untuk fill space tanpa overflow—ideal untuk hero sections yang stretch di landscape.

**Praktik Terbaik (2025):** Definisikan breakpoints di constants (e.g., { phone: 320, tablet: 768 }); gunakan useMemo untuk conditional styles agar hindari re-calc di setiap render. Test di device: Percentage cegah zoom issues di accessibility mode. Integrasi Flexbox: Wrap dengan gap untuk clean, scalable grids tanpa manual margins.

**Pertimbangan Platform:** Android: Percentage heights butuh explicit parent height; iOS: Smooth scaling di split-view iPad. Foldables: Thresholds handle unfold (width jumps dari 360px ke 800px).

### B. Orientation dan Foldable Handling: Dynamic Updates dengan Event Listeners

**Tujuan:** Teknik ini memastikan UI berubah mulus saat rotasi atau unfold foldable, menggunakan listeners untuk trigger re-layout tanpa manual intervention.

**Methods dan Hooks Kunci:**

| Teknik/Method | Tipe | Deskripsi | Default | Catatan & Pola Penggunaan |
|---------------|------|-----------|---------|---------------------------|
| `useWindowDimensions()` | Hook | Auto re-render saat orientation change; return {width, height, scale}. | - | Pola: Gunakan height > width ? 'portrait' : 'landscape' untuk switch justifyContent; preferred > manual listeners untuk simplicity. |
| `Dimensions.addEventListener('change', handler)` | Event: {window: {width, height}} | Manual subscribe ke resize events (rotasi, keyboard open, fold). | - | Pola: useEffect untuk setState(layoutType); cleanup subscription di return; kombinasikan dengan Flexbox flexDirection conditional. |
| Foldable Detection | width threshold (e.g., >600) | Deteksi unfold via width spike; apply multi-pane layout. | - | Pola: if (width > 800) { splitView: true } di Flexbox row; RN 0.75+: Event 'change' fire saat hinge angle ubah. |

**Pola Penggunaan:** Hook di component utama: const orientation = height > width ? 'portrait' : 'landscape'; style: { flexWrap: orientation === 'landscape' ? 'wrap' : 'nowrap' }. Pola foldable: Listener update state untuk side-by-side panels di unfolded mode.

**Praktik Terbaik (2025):** Gunakan hook untuk auto-handling; debounce listeners jika kompleks (e.g., via lodash). Test rotasi: Lock orientation di simulator, unlock untuk verify no jank. Integrasi Flexbox: Gunakan alignContent 'stretch' untuk fill new space post-rotasi.

**Pertimbangan Platform:** iOS: Event fire saat split-view resize; Android: Foldable API di 15+ (use WindowManager untuk hinge); keduanya dukung keyboard-induced changes.

### C. react-native-safe-area-context: Dynamic Safe Area Insets untuk Edge-to-Edge Design

**Tujuan:** Package ini (instalasi: `npm i react-native-safe-area-context`) menyediakan hooks dan components untuk dynamic insets, menggantikan deprecated SafeAreaView, memastikan layout fleksibel tanpa clipping di notched/gesture devices.

**Instalasi dan Setup:** Tambah ke native code (iOS: Pod install; Android: Auto-link). Wrap app di <SafeAreaProvider>.

**Hooks dan Components Kunci:**

| Hook/Component | Tipe | Deskripsi | Default | Catatan & Pola Penggunaan |
|----------------|------|-----------|---------|---------------------------|
| `useSafeAreaInsets()` | Hook: {top, bottom, left, right, frame} | Return current insets; re-render saat change (e.g., orientation). | {0} | Pola: const insets = useSafeAreaInsets(); style: { paddingTop: insets.top, paddingBottom: insets.bottom }; integrasi Flexbox untuk margin adaptif. |
| `<SafeAreaView>` (from package) | Component | Wrapper auto-pad children dengan insets. | - | Pola: <SafeAreaView style={{ flex: 1 }} edges={['top', 'bottom']}>; edges prop limit (e.g., no left/right di landscape). |
| `initialWindowMetrics` | Prop di Provider | Initial metrics untuk SSR/offline. | - | Pola: Gunakan di Expo untuk consistent boot-up; kombinasikan dengan Dimensions untuk full-screen calc. |
| `edges` (prop SafeAreaView) | ['top'/'bottom'/'left'/'right'] | Spesifik edges untuk apply insets. | All | Pola: edges={['top']} untuk status bar only; fleksibel untuk modal yang ignore bottom inset. |

**Pola Penggunaan:** Di root: <SafeAreaProvider><MyApp /></SafeAreaProvider>; di layout: const insets = useSafeAreaInsets(); <View style={{ flex: 1, paddingTop: insets.top, paddingBottom: Math.max(insets.bottom, 20) }}> (fallback untuk keyboard). Pola responsif: insets.right/left di landscape untuk side margins di foldables.

**Praktik Terbaik (2025):** Selalu wrap app di Provider; gunakan hook > component untuk granular control. Test di notched devices: Inset top ~44pt iPhone 14; bottom ~34pt dengan home indicator. Integrasi Flexbox: Apply insets sebagai padding di container, biarkan children flex-grow.

**Pertimbangan Platform:** iOS: Dukung Dynamic Island (insets update saat expand); Android: Gesture insets di 10+; package handle foldables dengan per-window metrics.

### D. Best Practices Design Flexible & Responsive (2025)

**Tujuan:** Strategi holistik untuk layout adaptif.

- **Relative Units & Breakpoints:** Gunakan % di flexBasis; define breakpoints (phone: <480px, tablet: 768px+) untuk conditional Flexbox props.
- **Gap untuk Spacing:** rowGap/columnGap di wrapped Flexbox untuk proportional gutters; scale dengan width * 0.01.
- **Memoization & Perf:** useMemo(conditionalStyle, [width]); hindari heavy calc di render.
- **Libraries Tambahan:** react-native-responsive-screen untuk scale (wp(10) = 10% width); kombinasikan dengan safe-area-context.
- **Testing:** Device lab (e.g., BrowserStack) untuk sizes/orientations; accessibility: Respect fontScale di Dimensions.

**Integrasi Keseluruhan:** Conditional % sizing + insets padding di Flexbox container; pola: Dynamic grid dengan safe top/bottom.

## 3. Contoh Implementasi

Kode siap di Expo (instal react-native-safe-area-context terlebih dahulu). Fokus adaptasi: Rotasi/unfold ubah layout.

### A. Contoh Dasar: Grid Adaptif dengan Conditional % Sizing

```jsx
import React from 'react';
import { View, Text, StyleSheet, useWindowDimensions } from 'react-native';

const AdaptiveGrid = () => {
  const { width } = useWindowDimensions();
  const colWidth = width > 768 ? '25%' : width > 480 ? '50%' : '100%';

  return (
    <View style={styles.container}>
      {Array.from({ length: 6 }, (_, i) => (
        <View key={i} style={[styles.item, { width: colWidth }]}>
          <Text>Item {i + 1}</Text>
        </View>
      ))}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'row',
    flexWrap: 'wrap',
    justifyContent: 'space-evenly',
    rowGap: 10,
    columnGap: 10,
    padding: 10,
  },
  item: {
    height: 100,
    backgroundColor: '#007AFF',
    alignItems: 'center',
    justifyContent: 'center',
  },
});

export default AdaptiveGrid;
```

**Penjelasan:** % width conditional; gap untuk spacing proporsional—1 kolom phone, 4 tablet.

### B. Contoh Interaktif: Layout Orientation dengan SafeArea Context

```jsx
import React, { useState, useEffect } from 'react';
import { View, Text, StyleSheet, Dimensions } from 'react-native';
import { SafeAreaProvider, useSafeAreaInsets, SafeAreaView } from 'react-native-safe-area-context';

const DynamicLayout = () => {
  const [dimensions, setDimensions] = useState(Dimensions.get('window'));
  const insets = useSafeAreaInsets();

  useEffect(() => {
    const subscription = Dimensions.addEventListener('change', ({ window }) => setDimensions(window));
    return () => subscription?.remove();
  }, []);

  const isLandscape = dimensions.width > dimensions.height;

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.safe} edges={['top', 'bottom']}>
        <View style={[styles.container, { flexDirection: isLandscape ? 'row' : 'column', paddingTop: insets.top, paddingBottom: insets.bottom }]}>
          <View style={styles.box1}><Text>Section 1</Text></View>
          <View style={styles.box2}><Text>Section 2</Text></View>
        </View>
        <Text style={styles.debug}>Orientation: {isLandscape ? 'Landscape' : 'Portrait'}</Text>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  safe: { flex: 1 },
  container: { flex: 1, justifyContent: 'space-between', gap: 10, paddingHorizontal: 20 },
  box1: { flex: 1, backgroundColor: 'red', justifyContent: 'center', alignItems: 'center' },
  box2: { flex: 1, backgroundColor: 'blue', justifyContent: 'center', alignItems: 'center' },
  debug: { textAlign: 'center', padding: 10 },
});

export default DynamicLayout;
```

**Penjelasan:** Listener + hook insets; direction conditional, padding adaptif—hindari clip saat rotasi.

### C. Contoh Lanjutan: E-Commerce Dashboard dengan Full Responsif

```jsx
import React from 'react';
import { View, Text, FlatList, StyleSheet, useWindowDimensions } from 'react-native';
import { SafeAreaProvider, useSafeAreaInsets } from 'react-native-safe-area-context';

const ResponsiveDashboard = () => {
  const { width } = useWindowDimensions();
  const insets = useSafeAreaInsets();
  const data = Array.from({ length: 12 }, (_, i) => ({ id: i.toString(), title: `Product ${i + 1}` }));
  const numCols = width > 800 ? 4 : width > 500 ? 2 : 1;
  const itemWidth = (width - (numCols - 1) * 10 - insets.left - insets.right) / numCols; // Gap + insets adjust

  return (
    <SafeAreaProvider>
      <View style={[styles.container, { paddingTop: insets.top, paddingBottom: insets.bottom }]}>
        <FlatList
          data={data}
          numColumns={numCols}
          renderItem={({ item }) => (
            <View style={[styles.product, { width: itemWidth }]}>
              <Text>{item.title}</Text>
            </View>
          )}
          keyExtractor={item => item.id}
          contentContainerStyle={styles.list}
          columnWrapperStyle={{ justifyContent: 'space-between', gap: 10 }}
        />
      </View>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, paddingHorizontal: 10 },
  list: { paddingBottom: 20 },
  product: { backgroundColor: '#f0f0f0', padding: 20, alignItems: 'center', borderRadius: 8 },
});

export default ResponsiveDashboard;
```

**Penjelasan:** Inset-adjusted width; numCols conditional + gap—adaptif dengan safe areas.

**Tips Debugging:** Gunakan Flipper Layout; test foldable emulator untuk unfold events.

## 4. Rangkuman

Hari ke-8 fokus pada teknik design flexible & responsive: Conditional % sizing dan thresholds untuk adaptasi screen, orientation/foldable handling via Dimensions hooks/listeners untuk dynamic switches, serta react-native-safe-area-context untuk insets granular yang cegah clipping di edge-to-edge UI. Kunci: Relative units + conditional Flexbox props; Provider/hook integrasi untuk zero-config; test multi-scenario untuk no-shift layouts. Ini ciptakan UI yang inclusive dan performant, siap untuk advanced scaling libraries.

**Referensi:**

- [react-native-safe-area-context · GitHub](https://github.com/th3rdwave/react-native-safe-area-context)
- [Dimensions · React Native](https://reactnative.dev/docs/dimensions)
- [Layout Props · React Native](https://reactnative.dev/docs/layout-props)
- [Creating Responsive Designs in React Native with react-native-responsive-screen](https://medium.com/@abdullahsevmez/creating-responsive-designs-in-react-native-with-react-native-responsive-screen-709e2f636867)
- [Essential Tips for Responsive Design in React Native](https://iamolchavan.medium.com/essential-tips-for-responsive-design-in-react-native-69327b0f8b4c)
- [5 Effective Ways to Create Responsive Design in React Native](https://medium.com/@devnexPro/5-effective-ways-to-create-responsive-design-in-react-native-4af63c8f23f9)

## 5. Evaluasi Harian: Soal Praktik

`Lanjutan project Mini E-Commerce`

Upgrade style project Mini E-Commerce yang sudah pernah dibuat ketika evaluasi kemarin, Pastikan:

1. Layout tetap responsif dan flexible di berbagai jenis layar dan di layar dengan orientasi yang berbeda.
2. Implementasikan pengetahuan yang sudah anda dapatkan dari materi pembelajaran kali ini
3. Pastikan konten tidak terhalang oleh area system android.

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

---

**Mobile App Development With React Native*
