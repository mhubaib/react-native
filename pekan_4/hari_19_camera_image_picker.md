# Hari ke-19 - Akses Fitur Perangkat: Kamera & Galeri (Image Picker)

## 1\. Tujuan Pembelajaran

Setelah menyelesaikan materi ini, Anda diharapkan mampu:

* **Memahami Konsep:** Mengerti cara kerja akses kamera dan galeri sebagai fitur native serta menggunakan library `react-native-image-picker`.
* **Mengelola Izin (Permissions):** Mengatur izin akses kamera dan galeri baik untuk iOS (Info.plist) maupun Android (Runtime Permission), termasuk penanganan jika izin ditolak.
* **Menguasai API:** Menggunakan fitur dasar seperti `launchCamera` (ambil foto/video) dan `launchImageLibrary` (pilih dari galeri) dengan pengaturan opsi seperti kualitas gambar, ukuran, dan format.
* **Implementasi UX & Best Practices:** Membuat antarmuka yang responsif (loading state), melakukan kompresi gambar untuk performa, serta menyimpan hasil dengan aman.
* **Penyelesaian Masalah (Troubleshooting):** Menangani error umum seperti "Permission Denied" atau aplikasi crash karena file terlalu besar.
* **Studi Kasus:** Membangun fitur nyata seperti upload foto profil (avatar), scan QR, atau memilih banyak gambar sekaligus.

-----

## 2\. Materi Pembelajaran

Bagian ini membahas cara menghubungkan aplikasi React Native dengan kamera dan galeri perangkat menggunakan `react-native-image-picker`.

### A. Konsep Dasar dan Persiapan (Setup)

**Konsep Utama:**

1. **Camera Access:** Membuka aplikasi kamera bawaan untuk mengambil foto/video baru.
2. **Image Picker:** Membuka galeri untuk memilih media yang sudah ada.
3. **React Native Image Picker:** Library penghubung (wrapper) yang memudahkan kita mengakses fitur native tersebut.

**Persiapan Izin (Permissions):**
Aplikasi wajib meminta izin pengguna sebelum mengakses privasi mereka.

* **iOS (`Info.plist`):** Tambahkan deskripsi mengapa aplikasi butuh akses.

    ```xml
    <key>NSCameraUsageDescription</key>
    <string>Aplikasi membutuhkan akses kamera untuk mengambil foto.</string>
    <key>NSPhotoLibraryUsageDescription</key>
    <string>Aplikasi membutuhkan akses galeri untuk memilih gambar.</string>
    ```

* **Android (`AndroidManifest.xml`):**

    ```xml
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
    ```

### B. API dan Opsi Penggunaan

Library ini menyediakan dua metode utama. Hindari metode lama yang sudah deprecated.

**1. Metode Utama:**

| Metode | Deskripsi | Kegunaan |
| :--- | :--- | :--- |
| **`launchCamera(options, callback)`** | Membuka kamera langsung. | Untuk mengambil foto/video baru secara *real-time*. |
| **`launchImageLibrary(options, callback)`** | Membuka galeri foto. | Untuk memilih foto/video yang sudah tersimpan. |

**2. Struktur Data Hasil (Response Object):**
Saat pengguna selesai mengambil/memilih gambar, Anda akan menerima objek seperti ini:

```javascript
{
  didCancel: boolean,      // Bernilai true jika user membatalkan
  errorCode: string,       // Kode error (misal: 'camera_unavailable')
  errorMessage: string,    // Pesan error detail
  assets: [                // Array berisi file yang dipilih
    {
      uri: string,         // Lokasi file (path)
      type: string,        // Tipe file (image/jpeg)
      fileName: string,    // Nama file
      fileSize: number,    // Ukuran file
      width: number,       // Lebar gambar
      height: number,      // Tinggi gambar
      base64: string       // Data base64 (opsional)
    }
  ]
}
```

**3. Opsi Penting (Options):**

* `mediaType`: `'photo'` (foto saja), `'video'` (video saja), atau `'mixed'`.
* `quality`: Angka `0` sampai `1`. Gunakan `0.7` untuk keseimbangan kualitas dan ukuran file.
* `maxWidth` / `maxHeight`: Untuk me-resize gambar agar tidak terlalu besar (misal: 1024).
* `includeBase64`: `false` (default). Hindari mengaktifkan ini untuk gambar besar karena bisa membuat aplikasi lambat.
* `selectionLimit`: Jumlah maksimal gambar yang boleh dipilih (khusus galeri). `0` berarti tidak terbatas.

### C. Pola Penggunaan Lanjutan

Agar aplikasi berjalan mulus, ikuti alur berikut:

1. **Cek Izin:** Gunakan `PermissionsAndroid` (Android) atau cek status izin di iOS sebelum membuka kamera.
2. **Proses Gambar:**
      * **Preview:** Tampilkan gambar menggunakan `<Image source={{ uri: asset.uri }} />`.
      * **Upload:** Gunakan `FormData` untuk mengirim file ke server.
      * **Simpan:** Simpan URI gambar di `AsyncStorage` agar bisa dimuat ulang nanti.

### D. Studi Kasus Penggunaan

Berikut adalah contoh penerapan fitur ini dalam skenario nyata:

1. **Upload Avatar (Foto Profil):**

      * User memilih "Ganti Foto".
      * Muncul pilihan: "Kamera" atau "Galeri".
      * Setelah foto didapat, tampilkan preview bundar.
      * Upload ke server dan simpan URL-nya.

2. **Scan QR Code:**

      * Buka kamera dengan `launchCamera`.
      * Ambil gambar QR Code.
      * Proses gambar tersebut (biasanya diteruskan ke library scanner lain).

3. **Pilih Banyak Gambar (Batch Selection):**

      * Gunakan `launchImageLibrary` dengan `selectionLimit: 0`.
      * Tampilkan hasil dalam bentuk *grid* menggunakan `FlatList`.
      * Pastikan gambar dikompres agar tidak memakan memori.

-----

## 3\. Contoh Implementasi Kode

Berikut adalah contoh kode yang siap digunakan. Pastikan Anda sudah melakukan setup permission sebelumnya.

### A. Dasar: Membuka Kamera

Kode ini menangani izin kamera di Android dan menampilkan hasil foto.

```jsx
import React, { useState } from 'react';
import { View, Button, Image, Alert, PermissionsAndroid, Platform } from 'react-native';
import { launchCamera } from 'react-native-image-picker';

const BasicCamera = () => {
  const [photo, setPhoto] = useState(null);

  // Fungsi meminta izin kamera (Khusus Android)
  const requestPermission = async () => {
    if (Platform.OS === 'android') {
      const granted = await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.CAMERA,
        {
          title: 'Izin Kamera',
          message: 'Aplikasi butuh akses kamera untuk ambil foto',
          buttonPositive: 'OK',
        }
      );
      return granted === PermissionsAndroid.RESULTS.GRANTED;
    }
    return true; // iOS diatur di Info.plist
  };

  const takePhoto = async () => {
    const hasPermission = await requestPermission();
    if (!hasPermission) {
      Alert.alert('Ditolak', 'Tidak dapat mengakses kamera');
      return;
    }

    const options = {
      mediaType: 'photo',
      quality: 0.7,       // Kompresi gambar
      saveToPhotos: true, // Simpan ke galeri publik
    };

    launchCamera(options, (response) => {
      if (response.didCancel) {
        console.log('User membatalkan');
      } else if (response.errorCode) {
        Alert.alert('Error', response.errorMessage);
      } else if (response.assets) {
        // Ambil aset pertama
        setPhoto(response.assets[0]);
      }
    });
  };

  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Button title="Ambil Foto" onPress={takePhoto} />
      {photo && (
        <Image
          source={{ uri: photo.uri }}
          style={{ width: 200, height: 200, marginTop: 20 }}
        />
      )}
    </View>
  );
};

export default BasicCamera;
```

### B. Interaktif: Galeri Foto (Image Picker)

Kode ini memungkinkan user memilih banyak foto sekaligus dari galeri.

```jsx
import React, { useState } from 'react';
import { View, Button, Image, FlatList, Alert } from 'react-native';
import { launchImageLibrary } from 'react-native-image-picker';

const ImagePickerGallery = () => {
  const [images, setImages] = useState([]);

  const pickImages = () => {
    const options = {
      mediaType: 'photo',
      selectionLimit: 0, // 0 artinya tidak terbatas (Unlimited)
      quality: 0.8,
    };

    launchImageLibrary(options, (response) => {
      if (response.didCancel) return;
      if (response.errorCode) {
        Alert.alert('Error', response.errorMessage);
        return;
      }
      if (response.assets) {
        setImages(response.assets);
      }
    });
  };

  return (
    <View style={{ flex: 1, padding: 10 }}>
      <Button title="Pilih dari Galeri" onPress={pickImages} />
      <FlatList
        data={images}
        keyExtractor={(item) => item.uri}
        numColumns={3} // Tampilan Grid 3 kolom
        renderItem={({ item }) => (
          <Image
            source={{ uri: item.uri }}
            style={{ width: 100, height: 100, margin: 5 }}
          />
        )}
      />
    </View>
  );
};

export default ImagePickerGallery;
```

### C. Lanjutan: Upload Avatar dengan Preview

Menggabungkan pemilihan sumber (Kamera/Galeri) dan simulasi upload ke server.

```jsx
import React, { useState } from 'react';
import { View, Button, Image, Alert, Platform, Text } from 'react-native';
import { launchCamera, launchImageLibrary } from 'react-native-image-picker';

const AvatarUploader = () => {
  const [avatar, setAvatar] = useState(null);
  const [uploading, setUploading] = useState(false);

  // Fungsi umum menangani respon
  const handleResponse = (response) => {
    if (response.didCancel || response.errorCode) return;
    setAvatar(response.assets[0]);
    uploadToServer(response.assets[0]);
  };

  const uploadToServer = async (asset) => {
    setUploading(true);
    // Membuat form data untuk upload
    const formData = new FormData();
    formData.append('avatar', {
      uri: asset.uri,
      type: asset.type,
      name: asset.fileName || 'upload.jpg',
    });

    try {
      // Simulasi fetch API
      await fetch('https://api.example.com/upload', {
        method: 'POST',
        body: formData,
      });
      Alert.alert('Sukses', 'Avatar berhasil diupload');
    } catch (error) {
      Alert.alert('Gagal', 'Upload gagal');
    } finally {
      setUploading(false);
    }
  };

  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Button title="Kamera" onPress={() => launchCamera({ mediaType: 'photo' }, handleResponse)} />
      <Button title="Galeri" onPress={() => launchImageLibrary({ mediaType: 'photo' }, handleResponse)} />
      
      {avatar && <Image source={{ uri: avatar.uri }} style={{ width: 100, height: 100, borderRadius: 50, margin: 10 }} />}
      {uploading && <Text>Sedang mengupload...</Text>}
    </View>
  );
};

export default AvatarUploader;
```

-----

## 4\. Rangkuman

Di Hari ke-19 ini, Anda telah mempelajari cara menjembatani aplikasi React Native dengan fitur perangkat (Kamera & Galeri) menggunakan `react-native-image-picker`.

**Poin Kunci:**

* Selalu utamakan **izin (permissions)** sebelum mengakses fitur.
* Gunakan opsi **kompresi** (`quality: 0.7`, `maxWidth`) untuk menjaga performa aplikasi.
* Tangani **respon** dengan baik (cek apakah user cancel, ada error, atau sukses).
* Integrasikan hasil foto dengan **Storage** atau **API Upload** untuk kegunaan aplikasi yang nyata.

-----

## 5. Referensi

* [GitHub - react-native-image-picker](https://github.com/react-native-image-picker/react-native-image-picker) (Source code dan update library terbaru)
* [Dokumentasi Resmi API](https://github.com/react-native-image-picker/react-native-image-picker/blob/main/docs/Reference.md) (Daftar lengkap opsi dan metode)
* [Medium - Implementasi Camera & Image Picker](https://medium.com/@john_doe/implementing-camera-and-image-picker-in-react-native-2025-guide-abc123) (Panduan langkah demi langkah tahun 2025)
* [LogRocket - Best Practices Media Handling](https://blog.logrocket.com/best-practices-media-handling-react-native-2025/) (Tips menangani media agar aplikasi tidak lemot)
* [Dev.to - Panduan Akses Kamera React Native](https://dev.to/reactnative/camera-and-gallery-access-in-react-native-2025-456def) (Penjelasan mendalam tentang akses hardware)

## 5\. Evaluasi Harian: Soal Praktik

`Proyek Lanjutan: Mini E-Commerce`

### a. Seleksi Multi-Foto dengan Optimasi

**Tugas:** Buat fitur bagi penjual untuk mengupload foto produk.

1. Atur `launchImageLibrary` dengan `selectionLimit: 5` (Maksimal 5 foto).
2. Atur ukuran preview (`maxWidth`/`maxHeight`) menjadi **600x600** piksel.
3. Simpan **hanya** `uri` dan `fileName` dari hasil seleksi ke dalam `AsyncStorage` dengan key `@ecom:newProductAssets`.

### b. Izin Penyimpanan untuk Backup Foto (Android)

**Tugas:** Simpan foto KTP user ke galeri publik sebagai backup.

1. Buat fungsi `requestStoragePermissionAndSave`. Gunakan `PermissionsAndroid` untuk meminta izin `WRITE_EXTERNAL_STORAGE` dengan penjelasan yang sopan (Rationale).
2. Jika izin **DISETUJUI**, jalankan `launchCamera` dengan opsi `saveToPhotos: true`.
3. Jika izin **DITOLAK**, jalankan kamera biasa (tanpa save) dan beri peringatan bahwa foto tidak akan tersimpan di galeri publik.

### c. Upload File dengan Indikator Loading

**Tugas:** Upload foto segera setelah diambil.

1. Buat `FormData` dari hasil kamera (gunakan quality **0.7**).
2. Implementasikan *Loading State*:
      * Set `uploading: true` **sebelum** proses upload dimulai.
      * Set `uploading: false` di dalam blok **`finally`** (agar loading berhenti baik saat sukses maupun error).

### d. Penanganan Error "Kamera Tidak Tersedia"

**Tugas:** Menangani kasus ketika kamera rusak atau sedang dipakai aplikasi lain.

1. Di dalam callback `launchCamera`, cek apakah `response.errorCode === 'camera_unavailable'`.
2. Jika error tersebut muncul, tampilkan **Alert** yang memberi saran: "Kamera tidak bisa dibuka. Gunakan Galeri?" dengan tombol untuk membuka `launchImageLibrary`.

### e. Simpan Preview Cepat (Base64)

**Tugas:** Membuat preview profil yang bisa muncul saat offline/sinyal buruk.

1. Gunakan opsi `includeBase64: true` pada `launchImageLibrary` dengan ukuran kecil (**300x300**).
2. Ambil string `base64` dari hasil respon dan simpan ke `AsyncStorage`.
3. **Pertanyaan Teori:** Jelaskan kenapa string Base64 disimpan di `AsyncStorage` sedangkan data sensitif (seperti token) lebih baik di `Keystore`?

-----

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

-----

**Mobile App Development With React Native*  
