# Hari ke-8: Membuat Layout Flexible & Responsive di React Native

## Tujuan Pembelajaran

Setelah menyelesaikan materi pembelajaran hari ini, siswa diharapkan mampu:

- Mengaplikasikan kombinasi properti Flexbox lanjutan (seperti `flexWrap`, `alignContent`, dan `justifyContent` dengan variasi) untuk membangun layout yang otomatis menyesuaikan dengan konten dinamis, termasuk handling overflow dan distribusi ruang yang optimal di skenario multi-item.
- Memanfaatkan hook `useWindowDimensions` dan fungsi `PixelRatio` secara efisien untuk mendeteksi dan merespons perubahan ukuran layar/orientasi secara real-time, termasuk integrasi dengan state management untuk re-rendering yang smooth.
- Merancang sistem conditional styling menggunakan helper functions yang modular, sehingga kode tetap clean dan scalable untuk aplikasi kompleks.
- Mengintegrasikan `react-native-safe-area-context` dari setup hingga advanced usage, termasuk handling edge cases seperti modal atau nested navigators, serta memahami dampaknya terhadap performa aplikasi.
- Menguji dan mengoptimalkan layout di berbagai simulator/emulator (iOS/Android, portrait/landscape) untuk memastikan konsistensi UX, dengan metrik seperti waktu render dan memory usage sebagai benchmark.

Tujuan ini dirancang agar Anda tidak hanya paham "cara", tapi juga "mengapa" dan "kapan" menerapkannya, sehingga bisa adaptasi ke proyek real-world.

## Materi Pembelajaran

### 1. Teknik Merancang Layout Flexible dengan Flexbox Lanjutan

Flexbox di React Native adalah fondasi utama untuk layout fleksibel, tapi level lanjutan melibatkan strategi untuk menangani konten yang unpredictable (misalnya, teks panjang atau gambar variabel). Bayangkan Flexbox seperti sungai: alur utama (main axis) mengalir item, dan sumbu silang (cross axis) mengatur posisi vertikal/horizontal. Kunci fleksibilitas adalah membuat item "mengalir" tanpa memaksa ukuran tetap.

- **Menggunakan `flexWrap` untuk Layout Multi-Barang (Deep Dive)**:
  - `flexWrap: 'wrap'` (atau `'nowrap'` default) memungkinkan item melipat ke baris baru saat melebihi lebar container. Ini krusial untuk daftar dinamis, seperti galeri foto yang jumlahnya berubah berdasarkan API response.
  - **Step-by-Step Cara Kerja**:
    1. Container punya `flexDirection: 'row'` (horizontal flow).
    2. Setiap child punya `flex: 1` atau `width: 'auto'` agar ukuran alami.
    3. Saat total width child > container width, item terakhir pindah ke baris baru.
  - **Pitfalls & Solusi**: Jika wrap menyebabkan item terlalu sempit (misalnya teks overflow), tambahkan `minWidth` pada child. Contoh snippet:

    ```jsx
    <View style={{ flexDirection: 'row', flexWrap: 'wrap', width: '100%' }}>
      {items.map((item, index) => (
        <View key={index} style={{ flex: 1, minWidth: 150, margin: 5 }}> {/* minWidth cegah terlalu kecil */}
          <Text>{item.name}</Text>
        </View>
      ))}
    </View>
    ```

  - **Kapan Pakai**: Ideal untuk e-commerce (kartu produk) atau social feed. Test: Tambah/hapus item secara dinamis dengan `setState` untuk lihat adaptasi.

- **Kontrol Distribusi dengan `justifyContent` dan `alignContent` (Eksplorasi Mendalam)**:
  - `justifyContent`: Mengatur spacing sepanjang main axis. Variasi lanjutan:
    - `space-evenly`: Ruang merata termasuk tepi (lebih uniform daripada `space-between`).
    - `space-around`: Ruang di sekitar item, tapi lebih di tengah.
    - Gunakan dengan `alignItems: 'stretch'` untuk item yang memenuhi cross axis penuh.
  - `alignContent`: Hanya aktif saat `flexWrap` on dan ada multi-baris. Ini seperti "justifyContent untuk baris-baris secara keseluruhan".
    - Contoh: `alignContent: 'center'` untuk pusatkan grup baris di container tinggi.
  - **Ilustrasi Konseptual**: Bayangkan 6 kotak di grid 2x3. `justifyContent: 'space-between'` bagi ruang horizontal per baris; `alignContent: 'flex-start'` tumpuk baris ke atas.
  - **Pitfalls & Solusi**: Di landscape mode, distribusi bisa bergeser—gabungkan dengan `useWindowDimensions` (lihat bagian 2). Snippet lanjutan:

    ```jsx
    <View style={{
      flexDirection: 'row',
      flexWrap: 'wrap',
      justifyContent: 'space-evenly', // Ruang merata per baris
      alignContent: 'center', // Pusatkan baris-baris di vertikal
      height: 300, // Butuh height eksplisit untuk alignContent
    }}>
      {/* 6 child View di sini */}
    </View>
    ```

  - **Tips Praktis**: Gunakan React DevTools untuk inspect layout tree dan lihat computed styles saat resize emulator.

- **Basis Data (Basis Item) Lanjutan**: Selalu prioritaskan `flexGrow: 1` untuk child yang ekspansif, dan `flexShrink: 1` untuk yang bisa menyusut. Ini mencegah layout "pecah" saat konten bertambah.

### 2. Teknik Responsif dengan API React Native

Responsif berarti layout berubah berdasarkan konteks perangkat, bukan hard-code. API React Native memungkinkan deteksi dinamis, tapi kuncinya adalah integrasi dengan lifecycle component untuk update efisien.

- **Hook `useWindowDimensions` (Breakdown Lengkap)**:
  - Mengembalikan `{ width, height, scale, fontScale }` yang update otomatis saat orientasi berubah (portrait ke landscape).
  - **Step-by-Step Implementasi**:
    1. Import: `import { useWindowDimensions } from 'react-native';`.
    2. Di functional component: `const { width, height } = useWindowDimensions();`.
    3. Gunakan di conditional: `const isLandscape = height < width;`.
    4. Wrap dengan `useEffect` jika perlu side-effect: `useEffect(() => { setLayout(isLandscape ? 'row' : 'column'); }, [width, height]);`.
  - **Use Cases Lanjutan**:
    - Grid vs List: Jika `width > 768` (tablet), render sebagai grid dengan `flexWrap`.
    - Orientasi: Di landscape, buat sidebar + content; portrait, stack vertikal.
  - **Pitfalls & Solusi**: Initial render bisa salah jika app load di landscape—gunakan `useState` untuk cache nilai sebelumnya. Snippet:

    ```jsx
    import { useState, useEffect } from 'react';
    // ...
    const [orientation, setOrientation] = useState('portrait');
    const { width, height } = useWindowDimensions();
    useEffect(() => {
      setOrientation(height > width ? 'portrait' : 'landscape');
    }, [width, height]);
    // Gunakan orientation di style
    ```

- **Fungsi `PixelRatio` (Detail Teknis)**:
  - `PixelRatio.get()`: Faktor skala (1-4x), `getFontScale()`: Skala font user (untuk accessibility).
  - **Cara Kerja**: Hitung ukuran adaptif, misalnya border: `1 / PixelRatio.get()` untuk garis tipis konsisten di high-DPI.
  - **Use Cases**: Font responsif: `fontSize: Math.min(18, 16 * PixelRatio.getFontScale())` agar teks tidak terlalu besar di tablet.
  - **Pitfalls**: Di webview hybrid, nilai bisa beda—test di device real. Snippet:

    ```jsx
    import { PixelRatio } from 'react-native';
    const responsiveFont = (baseSize) => baseSize * PixelRatio.getFontScale();
    // Di style: fontSize: responsiveFont(16)
    ```

- **Conditional Styling dengan Helper Functions (Modular Approach)**:
  - Buat file `utils/responsive.js` dengan fungsi seperti:

    ```js
    export const getLayoutStyle = (width) => ({
      flexDirection: width > 500 ? 'row' : 'column',
      alignItems: width > 500 ? 'flex-start' : 'center',
    });
    ```

  - **Mengapa Modular?**: Hindari inline conditional yang bikin kode berantakan; mudah test dan reuse.
  - **Tips Debugging**: Gunakan `console.log(dimensions)` di render untuk track perubahan, dan Flipper tool untuk profil performa.

### 3. Package `react-native-safe-area-context`

Package ini seperti "penjaga gerbang" untuk area layar yang aman, mencegah konten "terjebak" di zona sistem. Di perangkat modern (iPhone 14+, Samsung Galaxy S series), safe area bisa 20-50px di top/bottom, dan tanpa handling, layout jadi rusak.

- **Fungsi Utama (Dengan Contoh Dampak)**:
  - Akses insets secara akurat dan real-time, termasuk saat keyboard muncul (via listener opsional).
  - **Mengapa Penting?**: Tanpa ini, header app bisa overlap status bar, atau footer tombol hilang di gesture navigation—penyebab 30% bug UX di app mobile.

- **Props dan Komponen Utama (Detail Props)**:
  - **SafeAreaProvider**: Root wrapper. Props lengkap:
    - `style`: Style tambahan untuk provider (jarang dipakai).
    - `initialMetrics`: Object `{ frame: { x, y, width, height }, insets: { top, bottom, left, right } }` untuk mock di testing (gunakan Jest dengan react-native-testing-library).
    - `children`: App content.
  - **SafeAreaView**: Auto-padding. Props:
    - `style`: Override default.
    - `edges`: `['top', 'bottom', 'left', 'right']` atau subset, e.g., `edges={['top']}` hanya safe top untuk header.
    - `mode`: `'padding' | 'margin'` (default padding).
  - **useSafeAreaInsets()**: Hook simple, return insets object. Bisa dipanggil di mana saja di tree.

- **Methods dan Hooks (Advanced Usage)**:
  - **useSafeAreaInsets()**: Core hook. Update otomatis, tapi untuk manual control: `const insets = useSafeAreaInsets();`.
  - **useInitialWindowMetrics()**: Hook untuk metrics awal (frame + insets) saat app cold start—berguna untuk splash screen atau loading state.
  - **Listener**: Tambahkan `insets.addEventListener('change', callback)` untuk custom event (jarang, tapi untuk animasi kompleks).
  - **Nested Context**: Di modal atau drawer, wrap ulang dengan provider jika konflik.

- **Masalah yang Diselesaikan (Dengan Solusi Alternatif)**:
  - **Overflow di Notch/Home Indicator**: Solusi: Padding insets langsung ke root View. Alternatif manual: Gunakan `StatusBar.currentHeight` (Android only, kurang akurat).
  - **Inkonsistensi Cross-Platform**: Satu API untuk semua; tanpa ini, butuh `Platform.OS` checks yang verbose.
  - **Performa dan Battery**: Compute sekali di init ( <1ms), update lazy. Pitfall: Di app besar, hindari panggil hook di setiap render—cache dengan `useMemo`.
  - **Edge Cases Lanjutan**:
    - Landscape: Insets berubah (bottom lebih besar)—test dengan rotate emulator.
    - iPad Split-View: Left/right insets aktif; gunakan `edges` untuk adaptasi.
    - Keyboard: Integrasikan dengan `KeyboardAvoidingView` untuk bottom inset dinamis.
    - Troubleshooting: Jika insets 0, cek `pod install` (iOS) atau rebuild (Android). Log: `console.log(insets)` untuk debug.

Instalasi tetap sama, tapi tambahan: Untuk testing, mock insets di Jest dengan `jest.mock('react-native-safe-area-context')`.

## Contoh Implementasi

### 1: Product Grid Responsive

```tsx

// ============================================================================
// KOMPONEN 1: ProductGrid.js
// Grid responsif produk dengan flexWrap - untuk e-commerce, gallery
// Implementasi: flexWrap, justifyContent, minWidth, useWindowDimensions
// ============================================================================

import React from 'react';
import {
    View,
    Text,
    StyleSheet,
    ScrollView,
    TouchableOpacity,
    useWindowDimensions
} from 'react-native';
import { useSafeAreaInsets } from 'react-native-safe-area-context';

export default function ProductGrid() {
    const { width } = useWindowDimensions();
    const insets = useSafeAreaInsets();

    // Data produk contoh - ganti dengan data dari API/props
    const products = [
        { id: 1, name: 'Laptop Gaming', price: 'Rp 15.000.000', image: '💻' },
        { id: 2, name: 'Smartphone', price: 'Rp 8.000.000', image: '📱' },
        { id: 3, name: 'Headphone', price: 'Rp 2.500.000', image: '🎧' },
        { id: 4, name: 'Keyboard Mech', price: 'Rp 1.200.000', image: '⌨️' },
        { id: 5, name: 'Mouse Wireless', price: 'Rp 500.000', image: '🖱️' },
        { id: 6, name: 'Monitor 4K', price: 'Rp 6.000.000', image: '🖥️' },
    ];

    // Hitung kolom berdasarkan lebar layar
    const isTablet = width > 600;
    const isLandscape = width > 700;
    const cardWidth = isLandscape ? '30%' : (isTablet ? '45%' : '100%');

    return (
        <ScrollView
            style={[styles.container, { paddingTop: insets.top }]}
            contentContainerStyle={{ paddingBottom: insets.bottom + 40, paddingHorizontal: insets.left + 16 }}
        >
            <Text style={styles.title}>Daftar Produk</Text>
            <Text style={styles.subtitle}>
                Mode: {isLandscape ? 'Landscape (3 kolom)' : (isTablet ? 'Tablet (2 kolom)' : 'Mobile (1 kolom)')}
            </Text>

            <View style={styles.gridContainer}>
                {products.map((product) => (
                    <TouchableOpacity
                        key={product.id}
                        style={[styles.productCard, { width: cardWidth }]}
                        activeOpacity={0.7}
                        onPress={() => console.log('Product pressed:', product.id)}
                    >
                        <View style={styles.productImage}>
                            <Text style={styles.productEmoji}>{product.image}</Text>
                        </View>
                        <Text style={styles.productName}>{product.name}</Text>
                        <Text style={styles.productPrice}>{product.price}</Text>
                        <TouchableOpacity style={styles.addButton}>
                            <Text style={styles.addButtonText}>+ Keranjang</Text>
                        </TouchableOpacity>
                    </TouchableOpacity>
                ))}
            </View>
        </ScrollView>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#f5f5f5',
    },
    title: {
        fontSize: 24,
        fontWeight: 'bold',
        padding: 16,
        paddingBottom: 8,
        color: '#333',
    },
    subtitle: {
        fontSize: 14,
        color: '#666',
        paddingHorizontal: 16,
        paddingBottom: 16,
    },
    gridContainer: {
        flexDirection: 'row',
        flexWrap: 'wrap', // KEY: Wrap ke baris baru
        justifyContent: 'space-evenly', // Distribusi merata
        alignContent: 'flex-start', // Mulai dari atas
        padding: 8,
        gap: 16,
    },
    productCard: {
        backgroundColor: '#fff',
        minWidth: 150, 
        borderRadius: 12,
        padding: 16,
        elevation: 3,
    },
    productImage: {
        height: 100,
        backgroundColor: '#f0f0f0',
        borderRadius: 8,
        justifyContent: 'center',
        alignItems: 'center',
        marginBottom: 12,
    },
    productEmoji: {
        fontSize: 48,
    },
    productName: {
        fontSize: 16,
        fontWeight: '600',
        color: '#333',
        marginBottom: 4,
    },
    productPrice: {
        fontSize: 14,
        color: '#4CAF50',
        fontWeight: '700',
        marginBottom: 12,
    },
    addButton: {
        backgroundColor: '#2196F3',
        padding: 10,
        borderRadius: 6,
        alignItems: 'center',
    },
    addButtonText: {
        color: '#fff',
        fontWeight: '600',
        fontSize: 14,
    },
});
```

### 2: Adabtive Form

```tsx
// ============================================================================
// KOMPONEN 2: AdaptiveForm.js
// Form yang layout-nya berubah antara portrait/landscape
// Implementasi: useWindowDimensions, KeyboardAvoidingView, conditional flexDirection
// ============================================================================

import React, { useState } from 'react';
import {
    View,
    Text,
    StyleSheet,
    ScrollView,
    TextInput,
    TouchableOpacity,
    Switch,
    KeyboardAvoidingView,
    Platform,
    useWindowDimensions
} from 'react-native';
import { useSafeAreaInsets } from 'react-native-safe-area-context';

export default function AdaptiveForm() {
    const { width, height } = useWindowDimensions();
    const insets = useSafeAreaInsets();
    const [formData, setFormData] = useState({
        name: '',
        email: '',
        phone: '',
        address: '',
        newsletter: false
    });

    const isLandscape = height < width;
    const isTablet = width > 600;

    // Helper untuk update form
    const updateField = (field, value) => {
        setFormData(prev => ({ ...prev, [field]: value }));
    };

    const handleSubmit = () => {
        console.log('Form submitted:', formData);
        // Tambahkan logika submit Anda di sini
    };

    return (
        <KeyboardAvoidingView
            style={{ flex: 1 }}
            behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
        >
            <ScrollView
                style={[styles.container, { paddingTop: insets.top }]}
                contentContainerStyle={{ paddingBottom: insets.bottom + 20, paddingHorizontal: insets.left }}
                keyboardShouldPersistTaps="handled"
            >
                <Text style={styles.title}>Form Pendaftaran</Text>
                <Text style={styles.subtitle}>
                    {isLandscape && isTablet ? '↔️ Layout 2 Kolom' : '↕️ Layout 1 Kolom'}
                </Text>

                <View style={[
                    styles.formContainer,
                    {
                        flexDirection: isLandscape && isTablet ? 'row' : 'column', // KEY: Conditional layout
                        flexWrap: 'wrap',
                        justifyContent: 'space-between'
                    }
                ]}>
                    {/* Input Nama */}
                    <View style={[styles.inputGroup, { width: isLandscape && isTablet ? '48%' : '100%' }]}>
                        <Text style={styles.label}>Nama Lengkap *</Text>
                        <TextInput
                            style={styles.input}
                            placeholder="Masukkan nama"
                            value={formData.name}
                            onChangeText={(text) => updateField('name', text)}
                        />
                    </View>

                    {/* Input Email */}
                    <View style={[styles.inputGroup, { width: isLandscape && isTablet ? '48%' : '100%' }]}>
                        <Text style={styles.label}>Email *</Text>
                        <TextInput
                            style={styles.input}
                            placeholder="email@example.com"
                            value={formData.email}
                            onChangeText={(text) => updateField('email', text)}
                            keyboardType="email-address"
                            autoCapitalize="none"
                        />
                    </View>

                    {/* Input Phone */}
                    <View style={[styles.inputGroup, { width: isLandscape && isTablet ? '48%' : '100%' }]}>
                        <Text style={styles.label}>Nomor Telepon *</Text>
                        <TextInput
                            style={styles.input}
                            placeholder="08xxxxxxxxxx"
                            value={formData.phone}
                            onChangeText={(text) => updateField('phone', text)}
                            keyboardType="phone-pad"
                        />
                    </View>

                    {/* Input Address */}
                    <View style={[styles.inputGroup, { width: isLandscape && isTablet ? '48%' : '100%' }]}>
                        <Text style={styles.label}>Alamat</Text>
                        <TextInput
                            style={[styles.input, styles.textArea]}
                            placeholder="Alamat lengkap"
                            value={formData.address}
                            onChangeText={(text) => updateField('address', text)}
                            multiline
                            numberOfLines={3}
                        />
                    </View>

                    {/* Switch Newsletter */}
                    <View style={[styles.inputGroup, styles.switchGroup, { width: '100%' }]}>
                        <Text style={styles.label}>Subscribe Newsletter</Text>
                        <Switch
                            value={formData.newsletter}
                            onValueChange={(value) => updateField('newsletter', value)}
                            trackColor={{ false: '#ccc', true: '#4CAF50' }}
                            thumbColor={formData.newsletter ? '#fff' : '#f4f3f4'}
                        />
                    </View>

                    {/* Submit Button - Full width */}
                    <TouchableOpacity
                        style={[styles.submitButton, { width: '100%' }]}
                        onPress={handleSubmit}
                    >
                        <Text style={styles.submitButtonText}>💾 Simpan Data</Text>
                    </TouchableOpacity>
                </View>
            </ScrollView>
        </KeyboardAvoidingView>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#f5f5f5',
    },
    title: {
        fontSize: 24,
        fontWeight: 'bold',
        padding: 16,
        paddingBottom: 8,
        color: '#333',
    },
    subtitle: {
        fontSize: 14,
        color: '#666',
        paddingHorizontal: 16,
        paddingBottom: 16,
        fontStyle: 'italic',
    },
    formContainer: {
        padding: 16,
    },
    inputGroup: {
        marginBottom: 16,
    },
    label: {
        fontSize: 14,
        fontWeight: '600',
        color: '#333',
        marginBottom: 8,
    },
    input: {
        backgroundColor: '#fff',
        borderWidth: 1,
        borderColor: '#ddd',
        borderRadius: 8,
        padding: 12,
        fontSize: 16,
    },
    textArea: {
        height: 80,
        textAlignVertical: 'top',
    },
    switchGroup: {
        flexDirection: 'row',
        justifyContent: 'space-between',
        alignItems: 'center',
        backgroundColor: '#fff',
        padding: 12,
        borderRadius: 8,
        borderWidth: 1,
        borderColor: '#ddd',
    },
    submitButton: {
        backgroundColor: '#4CAF50',
        padding: 16,
        borderRadius: 8,
        alignItems: 'center',
        marginTop: 8,
    },
    submitButtonText: {
        color: '#fff',
        fontSize: 16,
        fontWeight: '700',
    },
});
```

### 3: Dashboard

```tsx
// ============================================================================
// KOMPONEN 3: DashboardCards.js
// Dashboard dengan cards yang adaptif berdasarkan ukuran layar
// Implementasi: flexWrap, alignContent, responsive card sizing, PixelRatio-friendly
// ============================================================================

import React from 'react';
import {
    View,
    Text,
    StyleSheet,
    ScrollView,
    useWindowDimensions,
    PixelRatio
} from 'react-native';
import { useSafeAreaInsets } from 'react-native-safe-area-context';

export default function DashboardCards() {
    const { width } = useWindowDimensions();
    const insets = useSafeAreaInsets();
    const fontScale = PixelRatio.getFontScale();

    const stats = [
        { id: 1, label: 'Total Penjualan', value: 'Rp 125M', icon: '💰', color: '#4CAF50' },
        { id: 2, label: 'Pengguna Aktif', value: '12.5K', icon: '👥', color: '#2196F3' },
        { id: 3, label: 'Produk Terjual', value: '8.9K', icon: '📦', color: '#FF9800' },
        { id: 4, label: 'Rating Rata-rata', value: '4.8/5', icon: '⭐', color: '#FFC107' },
        { id: 5, label: 'Transaksi Hari Ini', value: '234', icon: '📊', color: '#9C27B0' },
        { id: 6, label: 'Pending Orders', value: '45', icon: '⏳', color: '#F44336' },
    ];

    // Responsive card sizing dengan multiple breakpoints
    const isLarge = width > 768;
    const isMedium = width > 500;
    const cardWidth = isLarge ? '30%' : (isMedium ? '45%' : '100%');

    // Responsive font sizes
    const titleSize = Math.min(24, 22 * fontScale);
    const subtitleSize = Math.min(14, 12 * fontScale);

    return (
        <ScrollView
            style={[styles.container, { paddingTop: insets.top }]}
            contentContainerStyle={{ paddingBottom: insets.bottom + 350, paddingHorizontal: insets.left + 16 }}
        >
            <Text style={[styles.title, { fontSize: titleSize }]}>Dashboard Analytics</Text>
            <Text style={[styles.subtitle, { fontSize: subtitleSize }]}>
                Ukuran: {isLarge ? 'Desktop' : (isMedium ? 'Tablet' : 'Mobile')} ({width.toFixed(0)}px)
            </Text>

            {/* Stats Cards */}
            <View style={styles.dashboardGrid}>
                {stats.map((stat) => (
                    <View
                        key={stat.id}
                        style={[
                            styles.statCard,
                            {
                                width: cardWidth,
                                borderLeftColor: stat.color,
                                borderLeftWidth: 4,
                            }
                        ]}
                    >
                        <View style={styles.statHeader}>
                            <Text style={styles.statIcon}>{stat.icon}</Text>
                            <Text style={styles.statLabel}>{stat.label}</Text>
                        </View>
                        <Text style={[styles.statValue, { color: stat.color }]}>{stat.value}</Text>
                    </View>
                ))}
            </View>

            {/* Chart Section - Responsive layout */}
            <View style={styles.chartContainer}>
                <Text style={styles.chartTitle}>📈 Grafik Penjualan (4 Bulan Terakhir)</Text>
                <View style={[styles.chartPlaceholder, {
                    flexDirection: isLarge ? 'row' : 'column', // KEY: Conditional chart layout
                    alignItems: isLarge ? 'flex-end' : 'stretch',
                    justifyContent: 'space-around'
                }]}>
                    {[
                        { label: 'Jan', height: '60%', color: '#4CAF50' },
                        { label: 'Feb', height: '80%', color: '#2196F3' },
                        { label: 'Mar', height: '70%', color: '#FF9800' },
                        { label: 'Apr', height: '90%', color: '#9C27B0' },
                    ].map((bar, index) => (
                        <View key={index} style={styles.chartBar}>
                            <View style={[styles.bar, { height: bar.height, backgroundColor: bar.color }]} />
                            <Text style={styles.barLabel}>{bar.label}</Text>
                        </View>
                    ))}
                </View>
            </View>
        </ScrollView>
    );
};

const styles = StyleSheet.create({
    container: {
        flex: 1,
        backgroundColor: '#f5f5f5',
    },
    title: {
        fontSize: 24,
        fontWeight: 'bold',
        padding: 16,
        paddingBottom: 8,
        color: '#333',
    },
    subtitle: {
        fontSize: 14,
        color: '#666',
        paddingHorizontal: 16,
        paddingBottom: 16,
    },
    dashboardGrid: {
        flexDirection: 'row',
        flexWrap: 'wrap', // KEY: Wrap cards
        justifyContent: 'space-evenly',
        alignContent: 'flex-start', // KEY: Align rows to top
        gap: 16,
    },
    statCard: {
        backgroundColor: '#fff',
        minWidth: 150, // KEY: Prevent too narrow
        borderRadius: 12,
        padding: 16,
        elevation: 3,
    },
    statHeader: {
        flexDirection: 'row',
        alignItems: 'center',
        marginBottom: 12,
    },
    statIcon: {
        fontSize: 24,
        marginRight: 8,
    },
    statLabel: {
        fontSize: 12,
        color: '#666',
        flex: 1,
    },
    statValue: {
        fontSize: 24,
        fontWeight: 'bold',
    },
    chartContainer: {
        backgroundColor: '#fff',
        margin: 16,
        padding: 16,
        borderRadius: 12,
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.1,
        shadowRadius: 4,
        elevation: 3,
    },
    chartTitle: {
        fontSize: 16,
        fontWeight: '600',
        marginBottom: 16,
        color: '#333',
        textAlign: 'center',
    },
    chartPlaceholder: {
        minHeight: 200,
        padding: 16,
    },
    chartBar: {
        flex: 1,
        alignItems: 'center',
        justifyContent: 'flex-end',
        marginHorizontal: 4,
        minHeight: 150,
    },
    bar: {
        width: 40,
        backgroundColor: '#2196F3',
        borderRadius: 4,
        marginBottom: 8,
        minHeight: 20,
    },
    barLabel: {
        fontSize: 12,
        color: '#666',
        fontWeight: '600',
    },
});

```

## Rangkuman

Hari ke-8 ini telah kita dalami secara mendalam: Dari Flexbox lanjutan yang membuat layout "mengalir" seperti air dengan `flexWrap` dan distribusi pintar, hingga API responsif seperti `useWindowDimensions` yang seperti "mata" app untuk lihat perubahan layar, ditambah `PixelRatio` untuk presisi visual. Package `react-native-safe-area-context` menjadi pahlawan tak terlihat, menyelesaikan overflow cross-platform dengan insets yang akurat, hooks sederhana, dan handling edge cases yang jarang dibahas.

Intinya, fleksibel + responsif = app yang "hidup" dan user-centric: Gunakan dinamis > statis, modular > inline, dan selalu test iteratif. Praktikkan dengan modifikasi contoh—misalnya, tambah animasi dengan Reanimated untuk transisi layout. Ini fondasi kuat untuk hari-hari mendatang. Selamat eksplorasi, dan keep coding! Jika butuh kode lengkap atau variasi lain, saya siap bantu.

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
