# Hari ke-3: Core Component & Native Component (Membangun Struktur UI)

## 1. Tujuan Pembelajaran

Setelah menyelesaikan pembelajaran hari ini, siswa diharapkan mampu:

- Memahami peran masing-masing komponen inti (View, Text, Image, TextInput) dan native (ImageBackground, Modal, StatusBar, Switch) dalam membangun struktur UI React Native yang responsif dan native-feeling.
- Mengimplementasikan layout dasar menggunakan View, menangani teks dan gambar dengan styling yang konsisten, serta mengelola input pengguna melalui TextInput dan Switch.
- Mengintegrasikan elemen overlay seperti Modal dan background seperti ImageBackground untuk UI yang lebih interaktif, sambil menyesuaikan StatusBar untuk pengalaman pengguna yang optimal di berbagai platform.
- Membangun prototipe UI sederhana, seperti form login dengan input, toggle, dan modal konfirmasi, yang siap dikembangkan lebih lanjut.
- Menganalisis dan meng-debug isu umum, seperti overflow teks atau loading gambar lambat, untuk memastikan UI yang robust.

## 2. Materi Pembelajaran

Bagian ini memberikan penjelasan lengkap tentang beberapa komponen inti (core components) di React Native. Setiap komponen dibahas secara mendalam, mencakup tujuan (apa fungsinya), props utama (properti yang bisa disetel), dan pola penggunaan (cara menggunakannya dalam kode). Komponen-komponen ini saling melengkapi untuk membangun antarmuka pengguna (UI) yang solid: View sebagai dasar layout, Text dan Image untuk menampilkan konten visual, TextInput dan Switch untuk interaksi pengguna, serta Modal, ImageBackground, dan StatusBar untuk lapisan tambahan seperti overlay dan pengaturan sistem.

### A. View: Fondasi Layout dengan Flexbox

**Tujuan:** View adalah kontainer universal yang mendukung flexbox, styling, touch handling, dan aksesibilitas. Ini adalah blok bangunan utama untuk mengelompokkan elemen UI, mirip dengan <div> di web, tapi dioptimalkan untuk rendering native.

**Props Kunci:**

| Prop | Tipe | Deskripsi | Default | Catatan Platform |
|------|------|-----------|---------|------------------|
| `style` | StyleProp<ViewStyle> | Gaya layout (flexbox, dimensi, warna, border). Gunakan StyleSheet untuk performa. | {} | Dukung flexDirection: 'row'/'column'. |
| `children` | ReactNode | Elemen anak (Text, Image, dll.). | - | Nesting tak terbatas. |
| `pointerEvents` | 'auto'/'none'/'box-none'/'box-only' | Kontrol respons touch. | 'auto' | 'box-none' berguna untuk overlay. |
| `hitSlop` | Insets | Perluas area touch. | {} | Tingkatkan aksesibilitas touch target (min 44x44 pt). |
| `accessible` | boolean | Buat fokusabel untuk screen reader. | false | Wajib untuk elemen interaktif. |
| `accessibilityLabel` | string | Label untuk assistive tech. | Agregat dari anak Text. | Override untuk deskripsi custom. |
| `accessibilityRole` | AccessibilityRole | Role seperti 'button'/'image'. | undefined | Dukung ARIA-like props. |
| `onLayout` | LayoutChangeEvent => void | Callback saat layout berubah. | - | Dapatkan {x, y, width, height}. |
| `removeClippedSubviews` | boolean | Hapus subview offscreen untuk performa scroll. | false | Butuh overflow: 'hidden' di parent. |

**Pola Penggunaan:** Gunakan View untuk layout responsif dengan flexbox (flex: 1, justifyContent: 'center'). Nesting View menciptakan hierarki UI kompleks, seperti card atau grid. Integrasikan dengan TouchableOpacity untuk tombol custom. Pertimbangan performa: Hindari over-nesting; gunakan React.memo untuk subtree besar.

**Praktik Terbaik:** Selalu gunakan StyleSheet.create() untuk gaya statis. Aktifkan aksesibilitas untuk elemen non-tekstual. Pada Android, gunakan renderToHardwareTextureAndroid untuk animasi lancar, tapi nonaktifkan setelahnya untuk hemat memori.

### B. Text: Penanganan Teks yang Fleksibel

**Tujuan:** Text digunakan untuk menampilkan teks dengan dukungan nesting (teks bertingkat), styling, dan penanganan sentuhan. Ini memungkinkan pemformatan teks tanpa API native rumit.

**Props Kunci:**

| Prop | Tipe | Deskripsi | Default | Catatan |
|------|------|-----------|---------|---------|
| `style` | StyleProp<TextStyle> | Gaya font (size, weight, color). | {} | Warisi dari parent Text. |
| `numberOfLines` | number | Batasi baris; tambahkan ellipsis. | 0 (tak terbatas) | Gabung dengan ellipsizeMode: 'tail'. |
| `selectable` | boolean | Izinkan copy-paste. | false | Tingkatkan UX untuk teks panjang. |
| `accessibilityLabel` | string | Label screen reader. | Teks itu sendiri. | Agregat dari subtree. |
| `adjustsFontSizeToFit` | boolean | Skala font agar muat. | false | Berguna untuk teks dinamis. |
| `allowFontScaling` | boolean | Hormati pengaturan aksesibilitas font. | true | Nonaktifkan untuk UI konsisten. |
| `onPress` | GestureResponderHandler | Tangani tap. | - | Buat teks clickable. |
| `onTextLayout` | TextLayoutEvent => void | Callback layout per baris. | - | Ukur teks untuk animasi. |

**Pola Penggunaan:** Nesting Text untuk styling parsial (bold + italic). Gunakan sebagai anak View untuk blok teks. Pola umum: Custom Text component untuk font app-wide (e.g., const AppText = (props) => <Text style={{fontFamily: 'Custom'}} {...props} />).

**Praktik Terbaik:** Gunakan ellipsizeMode untuk truncation rapi. Pastikan accessible={true} untuk teks interaktif. Hindari inline styles berat; gunakan StyleSheet.

### C. Image: Menampilkan Gambar dengan Caching

**Tujuan:** Image menampilkan gambar dari URL jarak jauh, aset statis, atau file lokal. Ia mendukung loading progresif, caching, dan responsif.

**Props Kunci:**

| Prop | Tipe | Deskripsi | Default | Catatan |
|------|------|-----------|---------|---------|
| `source` | ImageSource | Sumber gambar (uri, require). | {} | Wajib dimensi untuk network image. |
| `style` | StyleProp<ImageStyle> | Dimensi, margin. | {} | Tentukan width/height eksplisit. |
| `resizeMode` | ImageResizeMode | Skala: 'cover'/'contain'/'stretch'. | 'cover' | 'contain' untuk aspek rasio asli. |
| `blurRadius` | number | Efek blur. | 0 | >5 untuk efek nyata di iOS. |
| `onLoad` | ImageLoadEvent => void | Sukses load. | - | Dapatkan dimensi nativeEvent. |
| `onError` | ErrorEvent => void | Gagal load. | - | Tangani fallback image. |
| `loadingIndicatorSource` | ImageSource | Spinner saat loading. | null | Gunakan GIF sederhana. |

**Pola Penggunaan:** Prefetch gambar penting dengan Image.prefetch(url). Query cache dengan Image.queryCache(urls) untuk optimasi offline. Dukung multiple sources untuk adaptive loading (e.g., 1x/2x).

**Praktik Terbaik:** Selalu tentukan dimensi untuk hindari layout shift. Gunakan cache: 'force-cache' di iOS untuk penghematan data. Pada Android, tambah dependensi Fresco untuk WebP/GIF animasi.

### D. TextInput: Input Pengguna yang Kuat

**Tujuan:** TextInput menangkap input teks dari pengguna melalui keyboard on-screen. Dukung single/multiline, auto-correct, dan event kustom.

**Props Kunci:**

| Prop | Tipe | Deskripsi | Default | Catatan |
|------|------|-----------|---------|---------|
| `value` | string | Nilai terkendali. | '' | Gabung dengan onChangeText. |
| `onChangeText` | (text) => void | Update saat teks berubah. | - | Gunakan state hook. |
| `placeholder` | string | Teks hint. | '' | Warnai dengan placeholderTextColor. |
| `keyboardType` | KeyboardType | 'numeric'/'email-address'. | 'default' | Prioritaskan inputMode di RN baru. |
| `secureTextEntry` | boolean | Sembunyikan untuk password. | false | Gabung textContentType='password'. |
| `maxLength` | number | Batasi karakter. | undefined | Enforce native untuk smooth. |
| `multiline` | boolean | Dukung multi-baris. | false | Nonaktifkan border samping di iOS. |
| `autoCapitalize` | AutoCapitalize | 'none'/'sentences'. | 'sentences' | Sesuaikan konteks input. |

**Pola Penggunaan:** Controlled mode dengan useState untuk validasi real-time. Ref untuk .focus()/.blur(). Pola: Form dengan onSubmitEditing untuk navigasi antar field.

**Praktik Terbaik:** Gunakan clearButtonMode (iOS) untuk UX cepat. Validasi di onEndEditing; tampilkan error via state. Pada Android, atur underlineColorAndroid='transparent' untuk custom style.

### E. ImageBackground: Latar Belakang Gambar

**Tujuan:** ImageBackground menjadikan gambar sebagai latar belakang untuk elemen lain, mirip CSS background-image. Ia merender Image di dalamnya.

**Props Kunci:** Mewarisi dari Image (source, style, resizeMode). Tambahan: `imageRef` untuk akses node Image internal.

**Pola Penggunaan:** Tentukan style={{width: '100%', height: '100%'}}; overlay View dengan opacity untuk teks readable.

**Praktik Terbaik:** Hindari kompleksitas tinggi; lihat source code untuk custom jika perlu.

### F. Modal: Overlay Konten

**Tujuan:** Modal menampilkan konten di atas layar utama, seperti dialog atau full-screen overlay, dengan animasi masuk.

**Props Kunci:**

| Prop | Tipe | Deskripsi | Default | Catatan |
|------|------|-----------|---------|---------|
| `visible` | boolean | Tampilkan/sembunyikan. | true | Kontrol via state. |
| `animationType` | 'slide'/'fade'/'none' | Animasi masuk. | 'none' | 'slide' untuk bottom sheet. |
| `transparent` | boolean | Background transparan. | false | Wajib untuk overlay. |
| `onRequestClose` | () => void | Tangani back button (wajib Android). | - | Integrasikan dismiss logic. |
| `presentationStyle` | 'fullScreen'/'pageSheet' (iOS) | Gaya tampilan iPad. | 'fullScreen' | 'pageSheet' untuk sheet. |

**Pola Penggunaan:** Gunakan dengan Portal-like pattern untuk avoid z-index issues. Swipe dismiss di iOS dengan allowSwipeDismissal.

**Praktik Terbaik:** Tambah backdropColor untuk semi-transparan. Test orientation change di iOS.

### G. StatusBar: Kustomisasi Bar Atas

**Tujuan:** StatusBar mengontrol tampilan dan visibilitas bilah status sistem (waktu, baterai, dll.) di bagian atas layar.

**Props Kunci:**

| Prop | Tipe | Deskripsi | Default | Catatan |
|------|------|-----------|---------|---------|
| `barStyle` | 'default'/'light-content'/'dark-content' | Warna teks ikon. | 'default' | 'light-content' untuk dark BG. |
| `backgroundColor` | color (Android) | Warna latar. | Sistem | Deprecated di Android 15. |
| `hidden` | boolean | Sembunyikan bar. | false | Animasi dengan showHideTransition (iOS). |
| `translucent` | boolean (Android) | Gambar di bawah bar. | false | Deprecated di Android 15. |

**Pola Penggunaan:** Multiple StatusBar untuk stack navigation. Gunakan imperative API seperti StatusBar.setHidden(true) untuk transient changes.

**Praktik Terbaik:** Sesuaikan barStyle berdasarkan tema app. Pada iOS, atur networkActivityIndicatorVisible untuk loading.

### H. Switch: Toggle Boolean

**Tujuan:** Switch adalah tombol toggle boolean (ya/tidak) yang dikontrol oleh state. Ia memerlukan callback untuk update nilai.

**Props Kunci:**

| Prop | Tipe | Deskripsi | Default | Catatan |
|------|------|-----------|---------|---------|
| `value` | boolean | Status on/off. | false | Controlled; update via state. |
| `onValueChange` | (value) => void | Callback perubahan. | - | Wajib untuk respons. |
| `disabled` | boolean | Nonaktifkan toggle. | false | Gray out visual. |
| `trackColor` | {false?: color, true?: color} | Warna track. | {} | Custom untuk state. |
| `thumbColor` | color | Warna grip. | - | Hilang shadow di iOS jika set. |

**Pola Penggunaan:** Integrasikan dengan Formik atau state untuk persistensi. Gunakan onChange untuk event alternatif.

**Praktik Terbaik:** Tambah accessibilityRole='switch' untuk screen reader. Test haptic feedback di iOS.

**Integrasi Keseluruhan:** Komponen ini membentuk struktur UI: View + flexbox untuk skeleton, Text/Image untuk konten, TextInput/Switch untuk input, Modal untuk modals, ImageBackground untuk visual, StatusBar untuk polish.

## 3. Contoh Implementasi

Bagian ini menyediakan kode lengkap yang dapat dijalankan di Expo atau simulator. Setiap contoh mengintegrasikan beberapa komponen untuk simulasi app nyata, seperti profil user dengan form, toggle, dan modal.

### A. Contoh Dasar: Card Profil dengan View, Text, Image, dan StatusBar

```jsx
import React from 'react';
import { View, Text, Image, StyleSheet, StatusBar } from 'react-native';

const ProfilCard = () => (
  <>
    <StatusBar barStyle="dark-content" backgroundColor="#f0f0f0" />
    <View style={styles.container}>
      <Image source={{ uri: 'https://reactnative.dev/img/tiny_logo.png' }} style={styles.avatar} />
      <Text style={styles.nama}>John Doe</Text>
      <Text style={styles.bio} numberOfLines={2}>Developer React Native enthusiast.</Text>
    </View>
  </>
);

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center', padding: 20 },
  avatar: { width: 100, height: 100, borderRadius: 50, marginBottom: 10 },
  nama: { fontSize: 24, fontWeight: 'bold', marginBottom: 5 },
  bio: { fontSize: 16, textAlign: 'center' },
});

export default ProfilCard;
```

**Penjelasan:** View sebagai kontainer flex, Image dengan resizeMode implisit, Text dengan truncation. StatusBar disesuaikan untuk light theme.

### B. Contoh Interaktif: Form Login dengan TextInput, Switch, dan Modal

```jsx
import React, { useState } from 'react';
import { View, Text, TextInput, Switch, Modal, Button, StyleSheet, Alert } from 'react-native';

const LoginForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isRemember, setIsRemember] = useState(false);
  const [modalVisible, setModalVisible] = useState(false);

  const handleLogin = () => {
    if (email && password) {
      setModalVisible(true);
    } else {
      Alert.alert('Error', 'Lengkapi form!');
    }
  };

  return (
    <View style={styles.container}>
      <TextInput
        style={styles.input}
        placeholder="Email"
        value={email}
        onChangeText={setEmail}
        keyboardType="email-address"
        autoCapitalize="none"
      />
      <TextInput
        style={styles.input}
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
        maxLength={20}
      />
      <View style={styles.switchContainer}>
        <Text>Ingat Saya</Text>
        <Switch value={isRemember} onValueChange={setIsRemember} trackColor={{ true: '#007AFF' }} />
      </View>
      <Button title="Login" onPress={handleLogin} />
      <Modal visible={modalVisible} transparent animationType="fade">
        <View style={styles.modalOverlay}>
          <View style={styles.modalContent}>
            <Text>Login Berhasil!</Text>
            <Button title="Tutup" onPress={() => setModalVisible(false)} />
          </View>
        </View>
      </Modal>
    </View>
  );
};

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20, justifyContent: 'center' },
  input: { borderWidth: 1, borderColor: '#ccc', padding: 10, marginBottom: 10, borderRadius: 5 },
  switchContainer: { flexDirection: 'row', justifyContent: 'space-between', marginBottom: 20 },
  modalOverlay: { flex: 1, backgroundColor: 'rgba(0,0,0,0.5)', justifyContent: 'center', alignItems: 'center' },
  modalContent: { backgroundColor: 'white', padding: 20, borderRadius: 10, width: '80%' },
});

export default LoginForm;
```

**Penjelasan:** TextInput controlled dengan state, Switch untuk toggle, Modal transparan dengan fade. Integrasi onSubmitEditing bisa ditambahkan untuk flow form.

### C. Contoh Lanjutan: Hero Section dengan ImageBackground dan Integrasi Penuh

```jsx
import React, { useState } from 'react';
import { View, Text, ImageBackground, Switch, StatusBar, StyleSheet } from 'react-native';

const HeroSection = () => {
  const [darkMode, setDarkMode] = useState(false);

  return (
    <>
      <StatusBar barStyle={darkMode ? 'light-content' : 'dark-content'} />
      <ImageBackground
        source={{ uri: 'https://reactnative.dev/img/tiny_logo.png' }}
        style={styles.background}
        imageStyle={{ opacity: 0.5 }}
      >
        <View style={styles.overlay}>
          <Text style={styles.title}>Selamat Datang!</Text>
          <View style={styles.switchContainer}>
            <Text>Dark Mode</Text>
            <Switch value={darkMode} onValueChange={setDarkMode} />
          </View>
        </View>
      </ImageBackground>
    </>
  );
};

const styles = StyleSheet.create({
  background: { flex: 1, resizeMode: 'cover' },
  overlay: { flex: 1, justifyContent: 'center', alignItems: 'center', backgroundColor: 'rgba(0,0,0,0.3)' },
  title: { fontSize: 32, color: 'white', marginBottom: 20 },
  switchContainer: { flexDirection: 'row', alignItems: 'center' },
});

export default HeroSection;
```

**Penjelasan:** ImageBackground sebagai latar, overlay View dengan Switch yang update StatusBar. Tambah onLoad untuk prefetch.

**Tips Debugging:** Gunakan React Native Debugger untuk inspect layout. Tes di device untuk touch events.

## 4. Ringkasan Materi

Pada Hari ke-3, kita mempelajari dasar-dasar UI (User Interface) di React Native melalui komponen-komponen inti dan native. Komponen ini membentuk fondasi untuk membangun tampilan aplikasi mobile yang sederhana namun kuat. Berikut adalah penjelasan singkat mengenai masing-masing komponen:

### Komponen Inti dan Native

- **View**: Digunakan untuk membangun struktur layout dengan sistem flexbox, seperti container utama untuk elemen-elemen lain.
- **Text**: Menampilkan konten teks statis atau dinamis, seperti label, judul, atau paragraf.
- **Image**: Menampilkan gambar statis dari sumber lokal atau jaringan, untuk menambahkan elemen visual.
- **TextInput**: Memungkinkan input teks dinamis dari pengguna, seperti form login atau pencarian.
- **ImageBackground**: Menjadikan gambar sebagai latar belakang kreatif untuk elemen lain, seperti header atau card.
- **Modal**: Membuat jendela pop-up untuk interupsi sementara, seperti konfirmasi atau detail tambahan.
- **StatusBar**: Mengintegrasikan elemen sistem seperti status bar ponsel untuk tampilan yang lebih native.
- **Switch**: Komponen toggle sederhana untuk pilihan on/off, seperti pengaturan notifikasi.

Dengan mengintegrasikan komponen-komponen ini, Anda dapat menciptakan UI mobile yang terasa native, responsif terhadap berbagai perangkat, dan ramah pengguna. Latihan lebih lanjut akan membantu menguasai kombinasi antar-komponen ini.

**Referensi:**

- [React Native Documentation: View](https://reactnative.dev/docs/view)
- [React Native Documentation: Text](https://reactnative.dev/docs/text)
- [React Native Documentation: Image](https://reactnative.dev/docs/image)
- [React Native Documentation: TextInput](https://reactnative.dev/docs/textinput)
- [React Native Documentation: ImageBackground](https://reactnative.dev/docs/imagebackground)
- [React Native Documentation: Modal](https://reactnative.dev/docs/modal)
- [React Native Documentation: StatusBar](https://reactnative.dev/docs/statusbar)
- [React Native Documentation: Switch](https://reactnative.dev/docs/switch)
- [Core Components Tutorial - React Native](https://reactnative.dev/docs/intro-react-native-components)

## 5. Evaluasi Harian: Soal Praktik

- **Buat contoh penggunaan semua core component yang telah dipelajari diatas, sertakan contoh penggunaan semua props defaultnya!**

### Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  
