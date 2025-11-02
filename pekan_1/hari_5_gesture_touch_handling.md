# Hari ke-5: Gesture & Touch Handling

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

- Memahami peran masing-masing komponen touch handling: Button untuk tombol sederhana, Pressable untuk gesture fleksibel, TouchableOpacity/Highlight untuk feedback visual dasar, TouchableWithoutFeedback untuk interaksi tanpa feedback, dan TouchableNativeFeedback untuk ripple native Android.
- Mengimplementasikan deteksi gesture seperti onPress, onLongPress, onPressIn/Out, dan hitSlop untuk area touch yang lebih besar, sambil mengintegrasikan dengan state untuk UI responsif.
- Menerapkan praktik terbaik seperti menggunakan Pressable untuk kode baru, menambahkan hitSlop untuk touch target minimal 44x44 pt, dan menghindari over-nesting untuk performa.
- Membangun prototipe interaktif seperti tombol custom dengan long press menu, yang siap untuk integrasi dengan animasi atau navigation.
- Menganalisis dan debug isu umum, seperti missed taps atau konflik gesture responder, untuk aplikasi yang robust di berbagai device.

## 2. Materi Pembelajaran

Bagian ini menjelaskan setiap komponen secara mendalam, termasuk tujuan, props kunci (dalam tabel), pola penggunaan, dan pertimbangan platform. Komponen ini dibangun di atas Pressability API, yang mengelola lifecycle gesture (start, move, end). Pressable direkomendasikan sebagai pengganti modern untuk touchables legacy (per docs 2025), karena lebih fleksibel dan future-proof.

### A. Button: Tombol Sederhana Cross-Platform

**Tujuan:** Button adalah komponen dasar untuk interaksi tap-to-act, dengan styling platform default (rounded iOS, material Android). Cocok untuk tombol sederhana; gunakan Pressable untuk customisasi lanjutan.

**Props Kunci:**

| Prop | Tipe | Default | Deskripsi | Catatan Platform |
|------|------|---------|-----------|------------------|
| `onPress` | Function | required | Callback saat tap; terima {nativeEvent: PressEvent}. | Cross-platform. |
| `title` | string | required | Teks tombol; auto-upper di Android. | - |
| `color` | Color | iOS: '#007AFF', Android: '#2196F3' | Warna teks (iOS) atau background (Android). | Perbedaan signifikan. |
| `disabled` | boolean | false | Nonaktifkan interaksi. | - |
| `accessibilityLabel` | string | - | Label screen reader. | Wajib untuk aksesibilitas. |
| `touchSoundDisabled` | boolean | false | Matikan suara tap (Android only). | Android. |
| `hasTVPreferredFocus` | boolean | false | Prioritas focus TV. | TV only. |

**Pola Penggunaan:** Gunakan untuk tombol submit/form. Pola: Integrasikan dengan state untuk loading (disabled=true). Hindari nesting dalam touchable lain untuk hindari konflik responder.

**Praktik Terbaik:** Selalu tambah accessibilityLabel. Gunakan disabled dengan visual cue (e.g., opacity). Per 2025, docs sarankan Pressable untuk advanced touch.

**Pertimbangan Platform:** Android upper-case title dan background color; iOS text color. TV: Gunakan nextFocus* props untuk navigasi.

### B. Pressable: Gesture Handler Fleksibel Modern

**Tujuan:** Pressable mendeteksi tahap press (in/out/move/long) pada child apapun, dengan dukungan dynamic styling dan hitSlop untuk UX lebih baik. Direkomendasikan untuk semua touch baru (2025).

**Props Kunci:**

| Prop | Tipe | Default | Deskripsi | Catatan Platform |
|------|------|---------|-----------|------------------|
| `onPress` | Function | - | Callback tap sukses. | Cross-platform. |
| `onLongPress` | Function | - | Long press setelah delayLongPress. | delayLongPress: 500ms. |
| `onPressIn/Out` | Function | - | Mulai/akhir touch. | - |
| `onPressMove` | Function | - | Gerak selama press. | Untuk drag detection. |
| `style` | Style or Function({pressed}) | - | Gaya dinamis berdasarkan pressed state. | - |
| `hitSlop` | Rect/number | - | Perluas area touch (e.g., {top:20}). | Min 44pt untuk aksesibilitas. |
| `pressRetentionOffset` | Rect/number | {top:20, etc.} | Toleransi gerak sebelum cancel. | Cegah missed taps. |
| `disabled` | boolean | false | Nonaktifkan. | - |
| `android_ripple` | RippleConfig | - | Ripple effect (color, borderless). | Android only. |
| `android_disableSound` | boolean | false | Matikan suara. | Android. |

**Pola Penggunaan:** Wrap View/Text untuk custom button. Pola: Gunakan style function untuk opacity change; hitSlop untuk tombol kecil. Integrasikan dengan Gesture Responder untuk multi-gesture.

**Praktik Terbaik:** Gunakan untuk new code; tambah visual feedback via style({pressed}). Test dengan accessibility tools. Per 2025, unstable_pressDelay untuk delay custom, tapi hati-hati.

**Pertimbangan Platform:** Android: Ripple native; iOS: No sound prop. Hover props untuk web/desktop.

### C. TouchableOpacity: Feedback Opacity Dasar

**Tujuan:** Wrapper yang kurangi opacity child saat press, ideal untuk tombol dengan dimming effect. Dibangun di atas TouchableWithoutFeedback + Animated.View.

**Props Kunci:**

| Prop | Tipe | Default | Deskripsi | Catatan Platform |
|------|------|---------|-----------|------------------|
| `activeOpacity` | number | 0.2 | Opacity saat active (0-1). | - |
| `onPress` | Function | - | Callback tap. | Inherited from TouchableWithoutFeedback. |
| `disabled` | boolean | false | Nonaktifkan. | - |
| `hasTVPreferredFocus` | boolean | false | Focus TV. | TV. |

**Pola Penggunaan:** Wrap Image/Text untuk icon button. Pola: Gabung dengan onPressIn untuk animasi custom.

**Praktik Terbaik:** Hindari layout shift dari Animated.View; gunakan Pressable sebagai alternatif. Per 2025, docs sarankan migrasi ke Pressable untuk kontrol lebih.

**Pertimbangan Platform:** Konsisten, tapi TV props untuk navigasi.

**Perbandingan dengan Pressable:** TouchableOpacity hanya opacity; Pressable tambah long press, move, dll.

### D. TouchableHighlight: Feedback Underlay Color

**Tujuan:** Tampilkan underlay color di bawah child saat press, dengan opacity reduction. Cocok untuk tombol dengan highlight visual.

**Props Kunci:**

| Prop | Tipe | Default | Deskripsi | Catatan Platform |
|------|------|---------|-----------|------------------|
| `activeOpacity` | number | 0.85 | Opacity saat active. | Butuh underlayColor. |
| `underlayColor` | Color | - | Warna underlay. | - |
| `onShowUnderlay/Hide` | Function | - | Callback show/hide underlay. | - |
| `onPress` | Function | - | Tap callback. | Inherited. |

**Pola Penggunaan:** Wrap View dengan backgroundColor eksplisit. Pola: Gunakan untuk list item.

**Praktik Terbaik:** Set backgroundColor child untuk hindari artifact. Migrasi ke Pressable per 2025.

**Pertimbangan Platform:** TV focus props.

**Perbandingan:** Mirip Opacity tapi dengan underlay; kurang fleksibel dari Pressable.

### E. TouchableWithoutFeedback: Interaksi Tanpa Feedback Visual

**Tujuan:** Tangkap press events tanpa ubah visual; base untuk touchables lain. Gunakan saat butuh custom feedback.

**Props Kunci:**

| Prop | Tipe | Default | Deskripsi | Catatan Platform |
|------|------|---------|-----------|------------------|
| `onPress` | Function | - | Tap sukses. | - |
| `onPressIn/Out` | Function | - | Mulai/akhir touch. | delayPressIn/Out: 0ms. |
| `onLongPress` | Function | - | Long press (500ms). | - |
| `disabled` | boolean | false | Nonaktifkan. | - |
| `hitSlop` | Rect/number | - | Perluas area. | - |
| `touchSoundDisabled` | boolean | false | Matikan suara (Android). | Android. |

**Pola Penggunaan:** Wrap custom View; tambah manual opacity via Animated. Pola: Untuk invisible tap areas.

**Praktik Terbaik:** Selalu tambah visual cue manual. Gunakan Pressable untuk new code; docs 2025 bilang "hanya jika alasan kuat".

**Pertimbangan Platform:** Android: touchSound; iOS: accessibilityIgnoresInvertColors.

### F. TouchableNativeFeedback: Ripple Native Android

**Tujuan:** Gunakan drawable native Android untuk ripple feedback; Android-only untuk pengalaman autentik.

**Props Kunci:**

| Prop | Tipe | Default | Deskripsi | Catatan Platform |
|------|------|---------|-----------|------------------|
| `background` | Object | - | Konfig ripple (e.g., Ripple(color, borderless)). | Android API 21+. |
| `useForeground` | boolean | - | Ripple di foreground (API 23+). | Cek canUseNativeForeground(). |
| `onPress` | Function | - | Tap callback. | Inherited. |

**Pola Penggunaan:** Gunakan static Ripple('#color', true). Pola: Untuk material buttons.

**Praktik Terbaik:** Cek kompatibilitas; fallback ke Pressable di iOS. Per 2025, Pressable dengan android_ripple sebagai alternatif cross-platform.

**Pertimbangan Platform:** Android only; TV focus.

**Integrasi Keseluruhan:** Mulai dengan Pressable; gunakan touchables legacy jika legacy code. Integrasikan Gesture Responder System untuk konflik multi-touch.

## 3. Contoh Implementasi

Kode siap di Expo/simulator. Integrasikan untuk simulasi app interaktif.

### A. Contoh Dasar: Tombol Sederhana dengan Button dan Pressable

```jsx
import React from 'react';
import { View, Text, Button, Pressable, StyleSheet, Alert } from 'react-native';

const SimpleButtons = () => {
  const handlePress = () => Alert.alert('Pressed!');

  return (
    <View style={styles.container}>
      <Button title="Button Sederhana" onPress={handlePress} color="#007AFF" />
      <Pressable
        onPress={handlePress}
        style={({ pressed }) => [styles.pressable, { opacity: pressed ? 0.7 : 1 }]}
        hitSlop={{ top: 10, bottom: 10, left: 10, right: 10 }}
      >
        <Text style={styles.text}>Pressable Custom</Text>
      </Pressable>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center', padding: 20 },
  pressable: { backgroundColor: '#007AFF', padding: 15, borderRadius: 5 },
  text: { color: 'white', fontSize: 16 },
});

export default SimpleButtons;
```

**Penjelasan:** Button untuk default; Pressable dengan dynamic opacity dan hitSlop.

### B. Contoh Interaktif: Feedback Visual dengan TouchableOpacity dan Highlight

```jsx
import React from 'react';
import { View, Text, TouchableOpacity, TouchableHighlight, StyleSheet, Alert } from 'react-native';

const FeedbackButtons = () => {
  const handlePress = () => Alert.alert('Pressed!');

  return (
    <View style={styles.container}>
      <TouchableOpacity
        style={styles.opacityBtn}
        activeOpacity={0.5}
        onPress={handlePress}
        disabled={false}
      >
        <Text style={styles.text}>Opacity Feedback</Text>
      </TouchableOpacity>
      <TouchableHighlight
        style={styles.highlightBtn}
        activeOpacity={0.6}
        underlayColor="#DDDDDD"
        onPress={handlePress}
        onShowUnderlay={() => console.log('Show underlay')}
      >
        <Text style={styles.text}>Highlight Underlay</Text>
      </TouchableHighlight>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center', padding: 20 },
  opacityBtn: { backgroundColor: '#007AFF', padding: 15, borderRadius: 5, marginBottom: 10 },
  highlightBtn: { backgroundColor: '#007AFF', padding: 15, borderRadius: 5 },
  text: { color: 'white', fontSize: 16 },
});

export default FeedbackButtons;
```

**Penjelasan:** Opacity untuk dimming; Highlight untuk underlay. Tambah disabled untuk state.

### C. Contoh Lanjutan: Long Press dan Native Feedback dengan TouchableWithoutFeedback dan NativeFeedback

```jsx
import React from 'react';
import { View, Text, TouchableWithoutFeedback, TouchableNativeFeedback, Platform, StyleSheet, Alert } from 'react-native';

const AdvancedTouches = () => {
  const handleLongPress = () => Alert.alert('Long Press!');
  const background = TouchableNativeFeedback.Ripple('#2196F3', true);

  return (
    <View style={styles.container}>
      <TouchableWithoutFeedback
        onPress={() => Alert.alert('Pressed!')}
        onLongPress={handleLongPress}
        delayLongPress={1000}
        hitSlop={{ top: 20 }}
      >
        <View style={styles.noFeedback}>
          <Text>No Visual Feedback</Text>
        </View>
      </TouchableWithoutFeedback>
      {Platform.OS === 'android' && (
        <TouchableNativeFeedback
          background={background}
          onPress={() => Alert.alert('Native Ripple!')}
          useForeground={TouchableNativeFeedback.canUseNativeForeground()}
        >
          <View style={styles.nativeBtn}>
            <Text style={styles.text}>Android Native</Text>
          </View>
        </TouchableNativeFeedback>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center', padding: 20 },
  noFeedback: { backgroundColor: 'gray', padding: 15, borderRadius: 5, marginBottom: 10 },
  nativeBtn: { backgroundColor: 'white', padding: 15, borderRadius: 5 },
  text: { color: 'black' },
});

export default AdvancedTouches;
```

**Penjelasan:** WithoutFeedback untuk custom long press; NativeFeedback untuk ripple Android.

**Tips Debugging:** Gunakan onPressIn/Out untuk log gesture. Test di device untuk touch accuracy.

## 4. Rangkuman

Hari ke-5 membangun kemampuan gesture handling melalui Button untuk sederhana, Pressable untuk modern fleksibel, dan touchables lain untuk feedback spesifik. Kunci: Gunakan Pressable untuk new code (2025 rec), tambah hitSlop/pressRetentionOffset untuk UX, prioritaskan aksesibilitas, dan integrasikan Gesture Responder untuk advanced. Ini ciptakan interaksi mobile yang responsif, inklusif, dan performa tinggi.

**Referensi:**

- [Button · React Native](https://reactnative.dev/docs/button)
- [Pressable · React Native](https://reactnative.dev/docs/pressable)
- [TouchableOpacity · React Native](https://reactnative.dev/docs/touchableopacity)
- [TouchableHighlight · React Native](https://reactnative.dev/docs/touchablehighlight)
- [TouchableWithoutFeedback · React Native](https://reactnative.dev/docs/touchablewithoutfeedback)
- [TouchableNativeFeedback · React Native](https://reactnative.dev/docs/touchablenativefeedback)
- [Gesture Responder System - React Native](https://reactnative.dev/docs/gesture-responder-system)
- [Handling Touches - React Native](https://reactnative.dev/docs/handling-touches)

## 5. Evaluasi Harian: Soal Praktik

- **Buat contoh penggunaan semua core component yang telah dipelajari pada materi kali ini, sertakan contoh penggunaan semua props defaultnya!**

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  
