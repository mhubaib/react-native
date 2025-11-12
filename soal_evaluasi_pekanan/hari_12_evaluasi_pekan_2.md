# Hari ke-12: Evaluasi pekan 2

`Lanjutan project Mini E-Commerce`

## 1. Durasi

**Waktu pengerjaan:** 1 hari (maksimal 8 jam efektif)  
**Batas pengumpulan:** sesuai instruksi mentor  

---

## 2. Tujuan Evaluasi

Soal-soal ini menguji kemampuan Anda dalam mengelola hierarki navigasi dan fitur navigasi lanjutan, termasuk responsivitas UI.

---

## 3. Soal Praktik

### Pengiriman Parameter dari Root ke Nested Tab

**Tugas:** Setelah pengguna berhasil **Login** (simulasikan ini dengan tombol di layar awal yang mengirim `userID: 'U123'` ke parameter Root Drawer), pastikan **Tab 'Profile'** dapat **menerima dan menampilkan** `userID` tersebut. Lakukan ini dengan mengakses parameter dari Parent Navigator (Drawer) saat Tab Profile dimuat.

---

### Header Ganda dan Penyesuaian UI

**Tugas:** Dalam arsitektur navigasi Anda, perbaiki masalah umum **header ganda** yang mungkin muncul. Pastikan hanya **satu header** dari *Stack Navigator* yang terlihat, berada di atas **Top Tabs Bar**, dan *header* tersebut secara dinamis menampilkan tombol *toggle* Drawer di sebelah kiri.

---

### Desain Responsif untuk Layout Produk

**Tugas:** Terapkan desain yang **responsif dan *flexible*** di halaman **Detail Produk** dan **Top Tabs Navigator**. Pastikan tampilan halaman *Screen* tersebut **beradaptasi dengan baik** ketika perangkat dirotasi dari mode **portrait** ke mode **landscape** (atau sebaliknya), misalnya dengan mengubah jumlah kolom pada daftar produk atau menyesuaikan ukuran gambar produk dll.

---

### Pengelolaan Gestur Tumpang Tindih (Swipe Conflict)

**Tugas:** Dalam hierarki navigasi aplikasi Anda, pecahkan *swipe conflict* (konflik geser) yang mungkin terjadi:

1. Jika pengguna mencoba **swipe horizontal** di area **Top Tabs Bar** untuk berpindah tab.
2. Jika pengguna mencoba **swipe horizontal** dari tepi kiri layar untuk membuka **Drawer**.

Pastikan **swipe dari tepi** selalu memprioritaskan **membuka Drawer**, tetapi **swipe di area Top Tabs** memprioritaskan **perpindahan antar tab**.

---

### Implementasi Modal dalam Stack untuk *Checkout*

**Tugas:** Buat *Screen* **'Checkout'** yang harus muncul sebagai **Modal Layar Penuh** di atas seluruh hierarki navigasi utama. Tambahkan tombol di halaman **Detail Produk** yang memicu modal ini. Pastikan saat modal muncul, **semua UI navigasi lainnya** (Drawer, Tabs Bar, Header, dll) **tersembunyi** sepenuhnya.

---

### *Listener* Global untuk *Analytics* Simulasi

**Tugas:** Integrasikan *listener* global di **`<NavigationContainer>`** (akar aplikasi). Setiap kali status navigasi berubah (misalnya, pengguna berpindah dari satu *Screen* ke *Screen* lain, terlepas dari tingkat *nesting*-nya), log ke suatu screen (e.g screen *history*) sebuah pesan simulasi yang menyertakan **nama rute baru** yang dikunjungi (misalnya, `[ANALYTICS] Rute dikunjungi: Detail Produk`).

---

### Dinamika Header Berdasarkan *Focus*

**Tugas:** Implementasikan perubahan dinamis pada *header Stack Navigator*. Gunakan *hook* **`useFocusEffect`** di **Tab 'Populer'** untuk mengatur ulang judul *header* Parent Stack menjadi **"Product ter Populer!"** saat tab ini **aktif** (fokus). Ketika tab kehilangan fokus (*blur*), kembalikan judul *header* ke nilai aslinya (misalnya, "Jelajahi Produk").

---

### Penguncian Drawer Bersyarat

**Tugas:** Kunci **Drawer Navigator** secara bersyarat. Setel `drawerLockMode` agar Drawer **otomatis terkunci** (`'locked-closed'`) setiap kali *Screen* **Detail Produk** atau **Checkout Modal** sedang aktif. Drawer harus **otomatis dibuka** kembali (`'unlocked'`) hanya saat pengguna berada di **Top Tabs Navigator** atau **Settings Screen**.

---

### Mengakses Fungsi Parent dari Child (Cross-Level Action)

**Tugas:** Di **Tab 'Populer'**, sediakan tombol yang secara programatik memanggil fungsi **`toggleDrawer()`** pada **Drawer Navigator**. Lakukan ini tanpa meneruskan `navigation` prop ke bawah, melainkan dengan mendapatkan referensi ke **Parent Navigator** (Drawer) secara langsung melalui *child navigation object*.

---

## 4. Kriteria Penilaian

| No. | Kriteria Penilaian | Bobot (%) |
| :--- | :--- | :--- |
| **1** | **Arsitektur Bersarang & Pengelolaan Hierarki (Nesting)** | 20% |
| | **Fokus:** Ketepatan membangun hierarki navigasi, menghilangkan *header* ganda, dan implementasi *Modal Penuh* yang berhasil menyembunyikan semua UI navigasi *parent*. | |
| **2** | **Aksi Lintas Level & Data Dinamis (Cross-Level Actions & Data)** | 20% |
| | **Fokus:** Kemampuan **membaca parameter** dari Root ke level dalam (Tingkat 3), **mengubah konfigurasi *parent*** (header) dari level *child* menggunakan `useFocusEffect`, dan **memanggil fungsi *parent*** (toggle Drawer) dari *child* menggunakan `getParent()`. | |
| **3** | **Manajemen State & Aliran Kontrol (State & Flow Control)** | 20% |
| | **Fokus:** Pengaturan *state* yang efisien, termasuk **penguncian Drawer bersyarat** (`drawerLockMode`) berdasarkan rute aktif (Soal 14), dan implementasi **reset *state* navigasi** saat *Tab blur* (Soal 10) untuk kebersihan data. | |
| **4** | **Performa & Integritas Pengalaman Pengguna (UX & Responsiveness)** | 20% |
| | **Fokus:** Implementasi **desain responsif** yang adaptif terhadap rotasi layar (Portrait $\leftrightarrow$ Landscape) (Soal 8), dan penyelesaian konflik gestur *Swipe* antara Top Tabs dan Drawer untuk UX yang mulus. | |
| **5** | **Custom Action & Global Event Handling** | 20% |
| | **Fokus:** Penggunaan **Global Listener** (`onStateChange`) pada `NavigationContainer` untuk *analytics* simulasi (Soal 12), dan implementasi **aksi *reset* Stack** khusus yang kompleks (Soal 3/Revisi: *Reset* Stack ke awal dan tutup Drawer). | |

---

## Ketentuan

```md
- Jawaban wajib dikumpulkan sebelum jam yang telah ditetapkan
- Soal wajib dikerjakan mandiri sesuai instruksi
```  

---

**Mobile App Development With React Native*
