# Hari ke-21 - Akses Fitur Perangkat: Geolocation (Lokasi)

## 1\. Tujuan Pembelajaran

Setelah menyelesaikan materi hari ini, siswa diharapkan mampu:

* **Memahami Konsep Dasar:** Mengerti cara kerja Geolocation untuk mendapatkan koordinat GPS (latitude/longitude) menggunakan library `@react-native-community/geolocation`.
* **Mengelola Izin (Permissions):** Mengatur izin lokasi di iOS (Info.plist) dan Android (Runtime Permission), serta menangani situasi jika izin ditolak oleh pengguna.
* **Menguasai API:** Menggunakan metode `getCurrentPosition` (sekali ambil) dan `watchPosition` (pelacakan terus-menerus) dengan pengaturan akurasi dan efisiensi baterai.
* **Implementasi Best Practices:** Menerapkan teknik hemat baterai, penanganan error yang baik, dan integrasi dengan peta (`react-native-maps`).
* **Penyelesaian Masalah:** Mampu melakukan debugging masalah umum seperti lokasi tidak akurat atau izin yang bermasalah.
* **Studi Kasus:** Membangun fitur nyata seperti Navigasi Real-time dan Geofencing (deteksi area).

-----

## 2\. Materi Pembelajaran

Bagian ini membahas cara mengakses lokasi pengguna menggunakan library standar `@react-native-community/geolocation`. Library ini menghubungkan kode JavaScript Anda dengan fitur GPS asli di HP (CoreLocation di iOS dan Fused Location di Android).

### A. Pengenalan Geolocation: Konsep dan Persiapan

**Konsep Kunci:**

1. **Geolocation API:** Jembatan untuk mendapatkan koordinat (Garis Lintang & Bujur).
2. **Library:** Kita menggunakan `@react-native-community/geolocation` karena mendukung fitur pelacakan (`watchID`) dan penanganan error yang lebih baik dibanding API bawaan browser.
3. **Akurasi vs Baterai:**
      * *High Accuracy (GPS):* Sangat akurat, tapi boros baterai.
      * *Low Accuracy (Network/WiFi):* Kurang akurat, tapi hemat baterai.

**Persiapan Izin (Setup):**
Aplikasi wajib meminta izin sebelum mengakses lokasi pengguna.

* **iOS (`Info.plist`):**
    Tambahkan penjelasan kenapa aplikasi butuh lokasi.

    ```xml
    <key>NSLocationWhenInUseUsageDescription</key>
    <string>Aplikasi membutuhkan lokasi untuk navigasi.</string>
    <key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
    <string>Aplikasi membutuhkan lokasi di latar belakang.</string>
    ```

* **Android (`AndroidManifest.xml`):**

    ````xml
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" /> ```

    ````

* **Import Library:**
    `import Geolocation from '@react-native-community/geolocation';`

### B. API & Metode Penggunaan

Berikut adalah fungsi-fungsi utama yang akan sering Anda gunakan:

| Metode | Fungsi | Penggunaan | Catatan Penting |
| :--- | :--- | :--- | :--- |
| **`getCurrentPosition`** | Mengambil lokasi saat ini **satu kali** saja. | Cek lokasi saat buka aplikasi atau saat tekan tombol check-in. | Memberikan hasil berupa objek posisi atau kode error. |
| **`watchPosition`** | Melacak perpindahan lokasi secara **terus-menerus**. | Navigasi (Maps) atau pelari yang merekam rute. | Mengembalikan `watchID` yang harus disimpan untuk dihentikan nanti. |
| **`clearWatch`** | Menghentikan pelacakan lokasi. | Dipanggil saat pengguna keluar dari halaman peta. | **Wajib** dipanggil untuk mencegah baterai cepat habis. |

**Opsi Pengaturan (Options):**

* `enableHighAccuracy`: `true` (Gunakan GPS, boros baterai) atau `false` (Gunakan WiFi/Seluler, hemat baterai).
* `timeout`: Batas waktu (ms) menunggu lokasi didapatkan sebelum error (Misal: 15000ms atau 15 detik).
* `maximumAge`: Menggunakan lokasi yang tersimpan di *cache* jika usianya belum melebihi batas ini (Misal: 60000ms). Mengurangi request ke hardware GPS.
* `distanceFilter`: Update lokasi hanya jika pengguna berpindah sejauh X meter (Misal: 10 meter). Sangat bagus untuk hemat baterai.

### C. Pola Penggunaan

Untuk pengalaman pengguna (UX) yang baik, ikuti alur ini:

1. **Cek Izin Dulu:** Gunakan `PermissionsAndroid` di Android. Di iOS, cek kode error permission.
2. **Tampilkan Penjelasan:** Jika user menolak, beri penjelasan (Rationale) kenapa fitur ini penting.
3. **Integrasi Peta:** Gunakan koordinat dari Geolocation untuk mengupdate `region` atau `Marker` di `react-native-maps`.

### D. Studi Kasus

**Kasus 1: Navigasi Real-Time (Live Tracking)**

* **Tujuan:** Marker di peta bergerak mengikuti user.
* **Cara:** Gunakan `watchPosition` dengan `distanceFilter: 10`. Setiap ada update, set state koordinat baru. Jangan lupa `clearWatch` saat komponen di-unmount.

**Kasus 2: Geofencing (Check-In Otomatis)**

* **Tujuan:** Aplikasi tahu jika user sudah sampai di toko.
* **Cara:** Hitung jarak antara lokasi user saat ini dengan lokasi toko. Jika jarak \< 50 meter, picu fungsi `triggerCheckIn()`.

[Image of Geofencing Concept Diagram]

-----

## 3\. Contoh Implementasi Kode

Berikut adalah contoh kode yang bisa langsung dicoba.

### A. Dasar: Mengambil Lokasi Sekali (`getCurrentPosition`)

Kode ini cocok untuk fitur seperti "Tag Lokasi Saya".

```jsx
import React, { useState } from 'react';
import { View, Text, Button, PermissionsAndroid, Platform } from 'react-native';
import Geolocation from '@react-native-community/geolocation';

const BasicGeolocation = () => {
  const [location, setLocation] = useState(null);
  const [errorMsg, setErrorMsg] = useState('');

  // Fungsi meminta izin di Android
  const requestPermission = async () => {
    if (Platform.OS === 'android') {
      const granted = await PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION,
        {
          title: 'Izin Lokasi',
          message: 'Aplikasi butuh akses lokasi Anda',
          buttonPositive: 'OK',
        }
      );
      return granted === PermissionsAndroid.RESULTS.GRANTED;
    }
    return true; // iOS diatur otomatis via Info.plist
  };

  const getLocation = async () => {
    const hasPermission = await requestPermission();
    if (!hasPermission) {
      setErrorMsg('Izin ditolak');
      return;
    }

    Geolocation.getCurrentPosition(
      (position) => {
        setLocation(position.coords); // Sukses
        setErrorMsg('');
      },
      (error) => setErrorMsg(`Error: ${error.message}`), // Gagal
      { enableHighAccuracy: true, timeout: 15000, maximumAge: 10000 }
    );
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Button title="Ambil Lokasi" onPress={getLocation} />
      {location && (
        <Text style={{ marginTop: 20 }}>
          Lat: {location.latitude}, Long: {location.longitude}
        </Text>
      )}
      {errorMsg ? <Text style={{ color: 'red' }}>{errorMsg}</Text> : null}
    </View>
  );
};

export default BasicGeolocation;
```

### B. Interaktif: Pelacakan Posisi (`watchPosition`)

Kode ini cocok untuk aplikasi navigasi atau olahraga.

```jsx
import React, { useState, useEffect } from 'react';
import { View, Text, Button } from 'react-native';
import Geolocation from '@react-native-community/geolocation';

const LocationTracker = () => {
  const [coords, setCoords] = useState(null);
  const [watchId, setWatchId] = useState(null);

  const startTracking = () => {
    // Mulai melacak
    const id = Geolocation.watchPosition(
      (position) => setCoords(position.coords),
      (error) => console.log('Error Watch:', error),
      { 
        enableHighAccuracy: true, 
        distanceFilter: 10, // Update setiap 10 meter
        interval: 5000 // (Android only) Update setiap 5 detik
      }
    );
    setWatchId(id);
  };

  const stopTracking = () => {
    if (watchId !== null) {
      Geolocation.clearWatch(watchId); // Hentikan pelacakan
      setWatchId(null);
    }
  };

  // Cleanup: Pastikan pelacakan berhenti jika user menutup layar
  useEffect(() => {
    return () => {
      if (watchId !== null) Geolocation.clearWatch(watchId);
    };
  }, [watchId]);

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Button 
        title={watchId ? 'Stop Tracking' : 'Mulai Tracking'} 
        onPress={watchId ? stopTracking : startTracking} 
      />
      {coords && (
        <Text style={{ marginTop: 20 }}>
          Posisi Terkini: {coords.latitude}, {coords.longitude}
        </Text>
      )}
    </View>
  );
};

export default LocationTracker;
```

### C. Lanjutan: Integrasi dengan Peta (`react-native-maps`)

Contoh ini menunjukkan cara mengambil lokasi pengguna dan memusatkan tampilan maps (Camera) ke lokasi tersebut, serta menambahkan penanda (Marker).

**Prasyarat:** Pastikan Anda sudah menginstal library maps:
`npm install react-native-maps`

```jsx
import React, { useState } from 'react';
import { View, Button, StyleSheet, Alert } from 'react-native';
import MapView, { Marker } from 'react-native-maps'; // Import Peta
import Geolocation from '@react-native-community/geolocation';

const MapIntegration = () => {
  // 1. Set default lokasi (Misal: Monas, Jakarta) agar peta tidak blank saat awal
  const [region, setRegion] = useState({
    latitude: -6.175392,
    longitude: 106.827153,
    latitudeDelta: 0.05, // Tingkat Zoom (makin kecil makin dekat)
    longitudeDelta: 0.05,
  });

  const goToMyLocation = () => {
    // 2. Ambil posisi saat ini
    Geolocation.getCurrentPosition(
      (position) => {
        const { latitude, longitude } = position.coords;

        // 3. Update state region agar peta berpindah ke lokasi user
        setRegion({
          latitude: latitude,
          longitude: longitude,
          latitudeDelta: 0.01, // Zoom lebih dekat
          longitudeDelta: 0.01,
        });
      },
      (error) => Alert.alert('Error', 'Gagal mengambil lokasi'),
      { enableHighAccuracy: true, timeout: 20000 }
    );
  };

  return (
    <View style={styles.container}>
      {/* Tampilkan Peta */}
      <MapView
        style={styles.map}
        region={region} // Peta akan mengikuti state ini
      >
        {/* Tambahkan Pin/Marker di lokasi tersebut */}
        <Marker
          coordinate={{
            latitude: region.latitude,
            longitude: region.longitude,
          }}
          title="Lokasi Saya"
          description="Anda berada di sini"
        />
      </MapView>

      {/* Tombol Aksi */}
      <View style={styles.buttonContainer}>
        <Button title="Cari Lokasi Saya" onPress={goToMyLocation} />
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1, // Penting agar peta memenuhi layar
  },
  map: {
    ...StyleSheet.absoluteFillObject, // Peta mengisi seluruh area container
  },
  buttonContainer: {
    position: 'absolute', // Tombol melayang di atas peta
    bottom: 20,
    alignSelf: 'center',
    width: '80%',
  },
});

export default MapIntegration;
```

-----

## 4\. Rangkuman

Hari ke-21 ini Anda telah mempelajari cara mengakses fitur lokasi di React Native.

**Poin Penting:**

* **Permission First:** Selalu minta izin user sebelum akses GPS.
* **Pilih Metode Tepat:** Gunakan `getCurrentPosition` untuk sekali ambil, dan `watchPosition` untuk pelacakan terus-menerus.
* **Hemat Baterai:** Gunakan `distanceFilter` dan `maximumAge`. Jangan lupa panggil `clearWatch` saat fitur tidak dipakai.
* **Debugging:** Gunakan log dan tes di perangkat asli untuk hasil GPS yang akurat.

**Referensi:**

* [Dokumentasi GitHub - react-native-geolocation](https://github.com/michalchudziak/react-native-geolocation)
* [Tutorial LogRocket - Geolocation Lengkap](https://blog.logrocket.com/react-native-geolocation-complete-tutorial/)

-----

## 5\. Evaluasi Harian: Soal Praktik

`Lanjutan Project Mini E-Commerce`

### 1\. Izin Lokasi dengan Penjelasan (Rationale)

**Skenario:** Fitur pencari "Toko Terdekat" butuh izin lokasi.
**Tugas:**

* Buat fungsi `requestLocationPermission`.
* Gunakan `PermissionsAndroid.request` dengan menyertakan objek **Rationale** (Judul & Pesan).
* Contoh pesan: "Kami butuh lokasi Anda untuk menampilkan toko terdekat secara akurat."
* Fungsi harus mengembalikan `true` jika disetujui, `false` jika ditolak.

### 2\. Optimasi Baterai (One-Time Fetch)

**Skenario:** Hitung ongkir otomatis saat checkout.
**Tugas:**

* Gunakan `getCurrentPosition` dengan opsi:
  * `enableHighAccuracy: true` (Agar akurat).
  * `timeout: 10000` (10 detik batas waktu).
  * `maximumAge: 60000` (Gunakan cache jika umur lokasi \< 1 menit).
* Jika terjadi error timeout (Code 3), tampilkan Alert: "Periksa koneksi GPS Anda".

### 3\. Live Tracking & Cleanup

**Skenario:** Fitur navigasi kurir. Update setiap 20 meter.
**Tugas:**

* Gunakan `watchPosition` dengan `distanceFilter: 20`.
* Simpan `watchId` ke dalam state.
* Buat `useEffect` dengan fungsi **cleanup**. Di dalamnya, panggil `clearWatch(watchId)` agar tracking berhenti total saat komponen di-unmount (layar ditutup).

### 4\. Integrasi Networking Hemat Data

**Skenario:** Kirim lokasi user ke server untuk analitik, tapi jangan spam server.
**Tugas:**

* Buat fungsi `sendLocationToServer(position)`.
* Panggil `getCurrentPosition` dengan `maximumAge: 120000` (2 menit).
* Jelaskan (bisa dalam komentar kode) bagaimana `maximumAge` membantu mengurangi beban server dan baterai dengan tidak mengambil data GPS baru jika data lama masih segar (di bawah 2 menit).

### 5\. Geofencing Sederhana (Promo Radius)

**Skenario:** Beri notifikasi jika user berada dalam radius 100m dari Toko Utama.
**Tugas:**

* Buat fungsi hitung jarak `calculateDistance(lat1, lon1, lat2, lon2)`.
* Gunakan `watchPosition` dengan `distanceFilter: 50`.
* Di dalam callback sukses: Hitung jarak user ke toko. Jika jarak \< 100 meter, munculkan `Alert` "PROMO DEKAT TOKO\!" dan langsung matikan tracking (`clearWatch`).

-----

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

-----

**Mobile App Development With React Native*  
