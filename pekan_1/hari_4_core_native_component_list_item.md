# Hari ke-4: Core Component & Native Component (Mengelola List Item)

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

- Memahami peran ScrollView sebagai dasar scrolling sederhana, FlatList untuk daftar data yang di virtualisasikan, SectionList untuk daftar bersection dengan header/footer, dan RefreshControl untuk pull-to-refresh interaktif.
- Mengimplementasikan list efisien dengan data array, renderItem custom, dan optimasi seperti getItemLayout untuk scrolling lancar tanpa layout shift.
- Mengintegrasikan fitur lanjutan seperti sticky headers, separators, viewability callbacks, dan pull-to-refresh dengan state management untuk UX dinamis.
- Menerapkan praktik terbaik termasuk aksesibilitas (VoiceOver/TalkBack support), performa (removeClippedSubviews, windowSize), dan penanganan perbedaan platform (e.g., stickySectionHeadersEnabled di iOS vs Android).
- Membangun prototipe seperti daftar kontak bergrup dengan refresh, yang siap untuk integrasi API nyata.
- Menganalisis dan debug isu umum, seperti jank pada scroll atau missing items pada clipped views, untuk list yang robust dan scalable.

## 2. Materi Pembelajaran

Bagian ini menjelaskan setiap komponen secara mendalam, termasuk tujuan, props kunci (dalam tabel), pola penggunaan, dan pertimbangan platform. Komponen ini saling melengkapi: ScrollView untuk prototipe cepat, FlatList/SectionList untuk produksi, dan RefreshControl untuk interaksi.

### A. ScrollView: Scrolling Dasar untuk Konten Terbatas

**Tujuan:** ScrollView adalah wrapper sederhana untuk konten yang melebihi layar, mendukung horizontal/vertical scroll dengan bouncing (iOS) dan over-scroll (Android). Cocok untuk list pendek (<50 item); tidak virtualized, jadi render semua children sekaligus—hindari untuk data besar.

**Props Kunci:**
| Prop | Tipe | Deskripsi | Default | Catatan Platform |
|------|------|-----------|---------|------------------|
| `contentContainerStyle` | StyleProp<ViewStyle> | Gaya untuk wrapper children. | {} | Tambah padding untuk spacing. |
| `horizontal` | boolean | Scroll horizontal. | false | Nonaktifkan vertical jika true. |
| `refreshControl` | ReactElement<RefreshControl> | Integrasi pull-to-refresh. | null | Hanya vertical. |
| `removeClippedSubviews` | boolean | Hapus offscreen views untuk performa. | true (Android), false (iOS) | Bisa sebabkan bug missing content. |
| `scrollEnabled` | boolean | Aktifkan scrolling. | true | Programmatic scroll tetap jalan. |
| `onScroll` | (event) => void | Callback scroll dengan nativeEvent (contentOffset, velocity). | - | Throttle dengan scrollEventThrottle. |
| `pagingEnabled` | boolean | Snap ke halaman penuh. | false | Ideal untuk carousel. |
| `keyboardDismissMode` | 'none'/'on-drag'/'interactive' | Dismiss keyboard saat drag. | 'none' | 'interactive' iOS only. |
| `bounces` | boolean | Bouncing di edge (iOS). | true | - |
| `overScrollMode` | 'auto'/'always'/'never' | Over-scroll Android. | 'auto' | Android only. |
| `stickyHeaderIndices` | number[] | Index untuk sticky headers. | [] | Gabung dengan StickyHeaderComponent. |
| `maintainVisibleContentPosition` | {minIndexForVisible: number, autoscrollToTopThreshold: number} | Jaga posisi visible saat update. | null | Cegah jump di chat-like list. |

**Pola Penggunaan:** Nesting View/Text/Image di dalam untuk list sederhana. Gunakan onContentSizeChange untuk deteksi ukuran. Pola umum: Horizontal untuk gallery, vertical untuk form panjang.

**Praktik Terbaik:** Batasi children untuk performa; gunakan FlatList untuk >50 item. Aktifkan aksesibilitas dengan accessible={true}. Hindari inline styles; gunakan StyleSheet. Pada Android, set endFillColor untuk optimasi overdraw.

### B. FlatList: Daftar Datar Virtualized untuk Performa Tinggi

**Tujuan:** FlatList merender daftar datar (non-sectioned) dengan virtualisasi—hanya item visible + buffer yang dirender. Ideal untuk list panjang (ribuan item), dukung horizontal, multi-column, dan scroll-to-index.

**Props Kunci:**
| Prop | Tipe | Deskripsi | Default | Catatan Platform |
|------|------|-----------|---------|------------------|
| `data` | Array<any> | Array item untuk render. | required | Update memicu re-render. |
| `renderItem` | ({item, index, separators}) => Element | Renderer per item. | required | Gunakan React.memo untuk item. |
| `keyExtractor` | (item, index) => string | Key unik untuk React. | item.key atau index | Stabil untuk hindari re-render. |
| `initialNumToRender` | number | Item awal dirender. | 10 | Sesuaikan dengan screen size. |
| `getItemLayout` | (data, index) => {length, offset, index} | Layout statis untuk optimasi. | null | Wajib untuk scrollToIndex lancar. |
| `refreshing` | boolean | Status pull-to-refresh. | false | Gabung dengan onRefresh. |
| `onRefresh` | () => void | Callback refresh. | - | Integrasi RefreshControl. |
| `numColumns` | number | Kolom grid (vertical only). | 1 | Item harus height sama. |
| `removeClippedSubviews` | boolean | Hapus offscreen views. | true (Android), false (iOS) | Tingkatkan performa, tapi test bug. |
| `viewabilityConfig` | ViewabilityConfig | Threshold viewable item. | {itemVisiblePercentThreshold: 10} | Untuk onViewableItemsChanged. |
| `extraData` | any | Force re-render jika berubah. | null | Untuk state eksternal seperti selection. |
| `ListHeaderComponent` | Element | Header sebelum item pertama. | null | - |
| `ListFooterComponent` | Element | Footer setelah item terakhir. | null | - |
| `ItemSeparatorComponent` | Element | Separator antar item. | null | Terima highlighted prop. |

**Pola Penggunaan:** Gunakan untuk feed sosial atau to-do list. Integrasikan onEndReached untuk infinite scroll. Pola: Horizontal untuk carousel, numColumns untuk grid.

**Praktik Terbaik:** Selalu sediakan keyExtractor stabil. Gunakan getItemLayout jika height fixed untuk scrollToEnd mulus. Batasi initialNumToRender untuk load cepat. Memoize renderItem dengan useCallback. Pada iOS, set stickyHeaderHiddenOnScroll jika pakai header.

### C. SectionList: Daftar Bergrup dengan Header/Footer

**Tujuan:** SectionList mirip FlatList tapi untuk data bersection (e.g., kontak A-Z), dengan renderSectionHeader/Footer dan sticky headers. Virtualized, dukung override per-section untuk renderItem custom.

**Props Kunci:**
| Prop | Tipe | Deskripsi | Default | Catatan Platform |
|------|------|-----------|---------|------------------|
| `sections` | Section[] | Array section {data, key?, renderItem?, ...}. | required | Data per grup. |
| `renderItem` | ({item, index, section}) => Element | Renderer item default. | required | Bisa override per section. |
| `renderSectionHeader` | ({section}) => Element | Header per section. | null | - |
| `renderSectionFooter` | ({section}) => Element | Footer per section. | null | - |
| `keyExtractor` | (item, index) => string | Key unik. | index | - |
| `stickySectionHeadersEnabled` | boolean | Sticky header ke atas. | false (iOS), true (Android) | iOS butuh explicit true. |
| `SectionSeparatorComponent` | Element | Separator antar section. | null | Di atas/bawah section. |
| `ItemSeparatorComponent` | Element | Separator antar item. | null | Dalam section saja. |
| `refreshing` | boolean | Status refresh. | false | - |
| `onRefresh` | () => void | Callback refresh. | - | - |
| `getItemLayout` | function | Layout statis. | null | Untuk performa. |
| `viewabilityConfig` | object | Config viewable. | default | Untuk callback changed. |
| `ListEmptyComponent` | Element | UI jika kosong. | null | - |
| `initialNumToRender` | number | Item awal. | 10 | - |

**Pola Penggunaan:** Gunakan untuk daftar alfabetis atau timeline berbulan. Pola: renderSectionHeader untuk judul grup, sticky untuk navigasi mudah.

**Praktik Terbaik:** Sediakan key per section untuk reordering. Gunakan per-section override jika data heterogen. Aktifkan removeClippedSubviews hati-hati. Pada Android, sticky default true—test konsistensi iOS.

### D. RefreshControl: Pull-to-Refresh Interaktif

**Tujuan:** RefreshControl menambahkan gesture pull-down untuk refresh di ScrollView/FlatList/SectionList, dengan spinner native dan state refreshing untuk feedback.

**Props Kunci:**
| Prop | Tipe | Deskripsi | Default | Catatan Platform |
|------|------|-----------|---------|------------------|
| `refreshing` | boolean | Tampilkan spinner. | required | Set true saat fetch, false setelah. |
| `onRefresh` | () => void | Callback pull. | required | Async fetch di sini. |
| `progressViewOffset` | number | Offset spinner dari atas. | 0 | Berguna dengan header. |
| `tintColor` | Color | Warna spinner (iOS). | null | iOS only. |
| `title` | string | Teks di bawah spinner (iOS). | null | iOS only, e.g., "Loading...". |
| `titleColor` | Color | Warna teks (iOS). | null | iOS only. |
| `colors` | Color[] | Warna spinner (Android). | null | Android only, minimal satu. |
| `progressBackgroundColor` | Color | Latar spinner (Android). | null | Android only. |
| `size` | 'default'/'large' | Ukuran spinner (Android). | 'default' | Android only. |
| `enabled` | boolean | Aktifkan gesture (Android). | true | Android only. |

**Pola Penggunaan:** Nest di refreshControl prop parent (e.g., <FlatList refreshControl={<RefreshControl ... />} />). Pola: Gunakan useState untuk refreshing, async di onRefresh.

**Praktik Terbaik:** Selalu sync refreshing state untuk hindari ghost spinner. Custom title/tint untuk branding. Test gesture di device—iOS lebih smooth. Tambah accessibilityLabel jika custom UI. Hindari multiple di satu scroll.

**Integrasi Keseluruhan:** Gabungkan untuk list lengkap: FlatList/SectionList sebagai base, ScrollView untuk sederhana, RefreshControl untuk update.

## 3. Contoh Implementasi

### A. Contoh Dasar: Daftar Sederhana dengan ScrollView dan RefreshControl

```jsx
import React, { useState } from 'react';
import { ScrollView, RefreshControl, Text, View, StyleSheet } from 'react-native';

const SimpleList = () => {
  const [refreshing, setRefreshing] = useState(false);
  const [data, setData] = useState(['Item 1', 'Item 2', 'Item 3', 'Item 4', 'Item 5']);

  const onRefresh = () => {
    setRefreshing(true);
    setTimeout(() => {
      setData([...data, `New Item ${data.length + 1}`]);
      setRefreshing(false);
    }, 1000);
  };

  return (
    <ScrollView
      style={styles.container}
      refreshControl={<RefreshControl refreshing={refreshing} onRefresh={onRefresh} />}
      contentContainerStyle={styles.content}
    >
      {data.map((item, index) => (
        <View key={index} style={styles.item}>
          <Text>{item}</Text>
        </View>
      ))}
    </ScrollView>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1 },
  content: { padding: 20 },
  item: { padding: 16, borderBottomWidth: 1, borderColor: '#ccc' },
});

export default SimpleList;
```

**Penjelasan:** ScrollView untuk render full list, RefreshControl tambah item simulasi. Tambah horizontal={true} untuk galeri.

### B. Contoh Interaktif: Daftar Datar dengan FlatList dan Viewability

```jsx
import React, { useState, useRef } from 'react';
import { FlatList, Text, View, StyleSheet, Alert } from 'react-native';

const DATA = Array.from({ length: 100 }, (_, i) => ({ id: i.toString(), title: `Item ${i}` }));

const FlatListExample = () => {
  const [refreshing, setRefreshing] = useState(false);
  const viewabilityConfig = { itemVisiblePercentThreshold: 50 };
  const onViewableItemsChanged = useRef(({ viewableItems }) => {
    if (viewableItems.length > 0) Alert.alert('Visible', `Item ${viewableItems[0].item.title} visible`);
  }).current;

  const onRefresh = () => {
    setRefreshing(true);
    setTimeout(() => setRefreshing(false), 1500);
  };

  const getItemLayout = (data, index) => ({ length: 50, offset: 50 * index, index });

  return (
    <FlatList
      data={DATA}
      keyExtractor={item => item.id}
      renderItem={({ item }) => (
        <View style={styles.item}>
          <Text>{item.title}</Text>
        </View>
      )}
      refreshControl={<RefreshControl refreshing={refreshing} onRefresh={onRefresh} />}
      getItemLayout={getItemLayout}
      initialNumToRender={10}
      viewabilityConfig={viewabilityConfig}
      onViewableItemsChanged={onViewableItemsChanged}
      ListHeaderComponent={<Text style={styles.header}>Daftar Item</Text>}
      ListFooterComponent={<Text style={styles.footer}>Akhir Daftar</Text>}
    />
  );
};

const styles = StyleSheet.create({
  item: { padding: 16, borderBottomWidth: 1, borderColor: '#ddd' },
  header: { padding: 16, fontWeight: 'bold' },
  footer: { padding: 16, textAlign: 'center', color: 'gray' },
});

export default FlatListExample;
```

**Penjelasan:** FlatList virtualized 100 item, getItemLayout untuk optimasi, viewability untuk track visible. Tambah numColumns={2} untuk grid.

### C. Contoh Lanjutan: Daftar Bergrup dengan SectionList dan Sticky Headers

```jsx
import React, { useState } from 'react';
import { SectionList, RefreshControl, Text, View, StyleSheet } from 'react-native';

const SECTIONS = [
  { title: 'A', data: ['Apple', 'Apricot'] },
  { title: 'B', data: ['Banana', 'Blueberry'] },
  { title: 'C', data: ['Cherry', 'Coconut'] },
];

const SectionListExample = () => {
  const [refreshing, setRefreshing] = useState(false);
  const [sections, setSections] = useState(SECTIONS);

  const onRefresh = () => {
    setRefreshing(true);
    setTimeout(() => {
      setSections([{ title: 'New', data: ['New Item'] }, ...sections]);
      setRefreshing(false);
    }, 1000);
  };

  return (
    <SectionList
      sections={sections}
      keyExtractor={(item, index) => item + index}
      renderItem={({ item }) => (
        <View style={styles.item}>
          <Text>{item}</Text>
        </View>
      )}
      renderSectionHeader={({ section: { title } }) => (
        <View style={styles.header}>
          <Text style={styles.headerText}>{title}</Text>
        </View>
      )}
      ItemSeparatorComponent={() => <View style={styles.separator} />}
      SectionSeparatorComponent={() => <View style={styles.sectionSeparator} />}
      refreshControl={<RefreshControl refreshing={refreshing} onRefresh={onRefresh} />}
      stickySectionHeadersEnabled={true}
      ListEmptyComponent={<Text style={styles.empty}>No Data</Text>}
    />
  );
};

const styles = StyleSheet.create({
  header: { backgroundColor: '#f0f0f0', padding: 8 },
  headerText: { fontWeight: 'bold' },
  item: { padding: 16 },
  separator: { height: 1, backgroundColor: '#ccc' },
  sectionSeparator: { height: 10, backgroundColor: 'transparent' },
  empty: { padding: 20, textAlign: 'center' },
});

export default SectionListExample;
```

**Penjelasan:** SectionList dengan grup, sticky headers, separators. Refresh tambah section baru.

**Tips Debugging:** Gunakan Flipper atau console.log di onScroll untuk monitor performa. Tes clipped views dengan removeClippedSubviews.

## 4. Rangkuman

Hari ke-4 membangun kemampuan mengelola list item melalui ScrollView untuk dasar, FlatList untuk efisiensi datar, SectionList untuk struktur bergrup, dan RefreshControl untuk interaksi refresh. Kunci: Virtualisasi kurangi lag, props seperti renderItem dan keyExtractor untuk customisasi, optimasi getItemLayout/windowSize untuk smooth scroll. Integrasi ini ciptakan list mobile yang scalable, accessible, dan native-feeling.

**Referensi:**

- [React Native Documentation: ScrollView](https://reactnative.dev/docs/scrollview)
- [React Native Documentation: FlatList](https://reactnative.dev/docs/flatlist)
- [React Native Documentation: SectionList](https://reactnative.dev/docs/sectionlist)
- [React Native Documentation: RefreshControl](https://reactnative.dev/docs/refreshcontrol)
- [VirtualizedList Reference - React Native](https://reactnative.dev/docs/virtualizedlist)

## 5. Evaluasi Harian: Soal Praktik

- **Buat contoh penggunaan semua core component yang telah dipelajari diatas, sertakan contoh penggunaan semua props defaultnya!**

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

---

**Mobile App Development With React Native*
