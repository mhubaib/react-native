# Hari Ke-18: Evaluasi Pekan 3

`Lanjutan Project Mini E-Commerce`

## 1. Durasi

**Waktu pengerjaan:** 1 hari (maksimal 8 jam efektif)  
**Batas pengumpulan:** sesuai instruksi mentor  

---

## 2. Tujuan Evaluasi

Setelah menyelesaikan evaluasi ini, Anda diharapkan mampu:

- Mengintegrasikan Deep Linking (Linking API dan react-navigation) untuk menangani cold start dan warm start dengan parsing parameter yang akurat.

- Menerapkan pola Hybrid Storage yang aman, membedakan antara data sensitif (Token Auth di Keystore) dan data non-sensitif (Preferensi, Cache di AsyncStorage).

- Mengimplementasikan Error Handling yang komprehensif (try-catch, Custom Error, Axios Interceptor) untuk skenario kegagalan I/O (Quota Exceeded) dan kegagalan otentikasi (401 Unauthorized).

- Membuat logic otentikasi yang tangguh (e.g., reset token kadaluarsa, redirect aman, cleanup data) sesuai praktik terbaik keamanan.

## 3. Soal Praktik

### a. Secure Access Token + Expiry Handling

- Setelah user login, simpan access token (dari `https://dummyjson.com/docs/auth`) di Keystore.
- Simpan juga **expiredAt** di AsyncStorage yang berfungsi sebagai tanggal expired token (simulasi token expired)
- Buat util yang memeriksa **expiredAt**, Jika sudah kedaluwarsa &rightarrow; hapus token dari Keystore dan redirect ke **LoginScreen**

---

### b. Protected Routes dengan Token Check

- Buat wrapper **ProtectedRoute** yang memeriksa token di Keystore, jika token tidak ada &rightarrow; navigasi ke **LoginScreen**, jika ada &rightarrow; render children.
- Terapkan pada **CartScreen** dan **CheckoutScreen**
- Uji dengan deep link (e.g: miniecom://cart) &rightarrow; jika belum login &rightarrow; diarahkan ke **LoginScreen**

---

### c. Wishlist Persistence

- Implementasikan fitur wishlist (add / remove toggle)
- Simpan wishlist ke **AsyncStorage** dengan format array of ids
- Saat menyimpan meta (e.g: count, updatedAt), gunakan **multiSet()** untuk menyimpan kedua key
- Ketika startup, ambil keduanya sekaligus

---

### d. Cache Product Detail (TTL per item)

- Di **ProductDetailScreen**, cache setiap product dengan key dinamis (e.g: @product_detail:{id}).
- Simpan dalam format **{ value: <productData>, ttl_product}**
- Ketika cache masih valid & user access product yang sama, tampilkan cache tanpa fetch ulang.

---

### e. Retry Logic dengan Exponential Backoff

- Untuk fetch product, implementasikan retry otomatis jika tidak ada koneksi / request timeout
- Jika masih gagal &rightarrow; lempar error sehingga **ErrorBoundary** menampilkan ui error

---

### f. Event Listener Deep Link + Cart Action

- Tangani deep link warm start (app background) untuk aksi modifikasi state.
- Jika Link (e.g: **miniecom://add-to-cart/55**) datang, parse id **55**, tambahkan product ke cart.
- Pastikan listener di bersihkan ketika unmount.

---

### g. State Hydration: Load Semua Data Startup

- Di root project, muat semua data secara paralel ketika startup
- Tampilkan **SplashScreen** sampai semua proses selesai

---

### h. Clear Storage

- Buat fungsi logout yang akan menghapus semua data di Storage.
- Pastikan tidak terjadi race condition ketika proses menghapus data
- Setelah logout &rightarrow; reset navigation ke **LoginScreen**
- Pastikan deep link (e.g: miniecom://cart) tidak berfungsi dan redirect ke **LoginScreen** setelah logout

---

### i. Handling Corrupted Storage

- Simulasikan data AsyncStorage yang corrupt (e.g: value yang bukan JSON)
- Ketika parsing data, ada yang corrupt: Log error, hapus item yang corrupt, tampilkan fallback UI.
- Pastikan recovery berjalan otomatis tanpa aplikasi crash

---

### j. Linking + Auth + Fallback Navigation

- Saat menerima deep link, validasi params: (e.g: miniecom://product/abcxyz) &rightarrow; invalid (id harus angka)
- Jika invalid: redirect ke **/home**, tampilkan alert "Tautan tidak valid, dialihkan ke beranda"
- Jika valid tapi user belum login: arahkan ke **LoginScreen**, setelah login sukses &rightarrow; otomatis navigate kembali ke target screen (e.g: miniecom://product/12 &rightarrow; akan navigasi ke detail screen dengan parameter id 12)

## 4. Kriteria Penilaian

| No. | Kriteria | Bobot |
| :--- | :--- | :--- |
| **1** | **Integrasi Deep Link** | 35% |
| **2** | **Pola Hybrid Storage** | 30% |
| **3** | **Error Handling** | 25% |
| **4** | **Keamanan & Konsistensi** | 10% |

---

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

---

**Mobile App Development With React Native*
