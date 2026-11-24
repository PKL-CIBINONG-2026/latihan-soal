# Lembar Tugas Praktik Kerja Lapangan (PKL)
## Eksplorasi HTTP Methods & Integrasi Frontend dengan Mock API

Pada tugas ini, kamu akan membangun aplikasi web interaktif menggunakan **HTML, CSS, dan Pure JavaScript (`fetch` API)** yang terhubung dengan **Mock API Portal**:
**Portal Dokumentasi & Playground:** `https://pkl-cibinong-2026.github.io/mock-api/#/articles/list`

Berbeda dengan mock API biasa, server ini menerapkan **aturan validasi HTTP yang ketat**:
* **`PUT`** wajib mengirim seluruh data utama secara utuh (*full replacement*) atau akan ditolak server (`400 Bad Request`).
* **`PATCH`** hanya menerima perubahan sebagian (*partial update*) dan menolak field asing.
* **`DELETE`** mengembalikan status `204 No Content` dengan *body* benar-benar kosong.
* **`POST` & `PUT`** memvalidasi konflik data (`409 Conflict`) dan kesalahan format/logika (`422 Unprocessable Entity`).

---

## Persiapan & Cara Menghubungkan ke Mock API

Sebelum mulai mengoding, ikuti langkah persiapan berikut:

### 1. Eksplorasi Portal Mock API
1. Buka `https://pkl-cibinong-2026.github.io/mock-api/#/articles/list` di browser.
2. Pelajari struktur data, daftar endpoint, query parameter, serta respons error di setiap tab studi kasus.
3. Gunakan fitur **Playground / Try It Out** di portal tersebut untuk mencoba mengirim request terlebih dahulu.
4. Salin **Script SDK Snippet** (untuk penggunaan dari `localhost`) serta catat **Base URL** API yang tertera pada halaman portal.

### 2. Setup Proyek Lokal (`localhost`)
1. Buat folder proyek baru (misalnya `latihan-fetch-pkl`), lalu jalankan menggunakan **VS Code Live Server** (`http://127.0.0.1:5500` atau `http://localhost:5500`).
2. Pasang script SDK dari portal Mock API di dalam `<head>` file `index.html` kamu (sesuaikan dengan snippet yang ada di portal):

```html
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Latihan Fetch API - PKL</title>
  <!-- Pasang SDK Mock API sebelum app.js -->
  <script type="module" src="https://pkl-cibinong-2026.github.io/mock-api/client.js"></script>
</head>
<body>
  <div id="app"></div>
  <script type="module" src="./app.js"></script>
</body>
</html>
```

### 3. Template Helper `apiRequest()` (Wajib Dipahami)
Secara bawaan, `fetch()` **tidak melempar error** ke blok `catch` apabila server membalas status `400`, `404`, `409`, atau `422`. Selain itu, memanggil `res.json()` pada status `204 No Content` akan menyebabkan error *Unexpected end of JSON input*.

Gunakan fungsi pembungkus (*wrapper*) berikut di file `api.js` kamu:

```javascript
// Sesuaikan BASE_URL dengan yang tertera di portal https://pkl-cibinong-2026.github.io/mock-api/
const BASE_URL = "https://mock-api.pkl.local/api/v1";

export async function apiRequest(endpoint, options = {}) {
  const response = await fetch(`${BASE_URL}${endpoint}`, {
    headers: {
      "Content-Type": "application/json",
      ...options.headers,
    },
    ...options,
  });

  // 1. Tangani status 204 No Content (biasanya dari DELETE) & method HEAD
  if (response.status === 204 || options.method === "HEAD") {
    return {
      ok: response.ok,
      status: response.status,
      headers: response.headers,
      data: null,
    };
  }

  // 2. Parse JSON response body
  const data = await response.json().catch(() => null);

  // 3. Lempar error terstruktur jika status HTTP bukan 2xx (misal 400, 404, 409, 422)
  if (!response.ok) {
    const error = new Error(data?.message || `HTTP Error ${response.status}`);
    error.status = response.status;
    error.details = data;
    throw error;
  }

  return {
    ok: true,
    status: response.status,
    headers: response.headers,
    data,
  };
}
```

---

## Daftar Tugas Studi Kasus (4 Modul)

> **Catatan Pengerjaan:** Kerjakan keempat studi kasus berikut secara berurutan (atau sesuai pembagian dari pembimbing PKL). Pastikan seluruh kode manipulasi data lokal tetap menerapkan **Immutability** (`.map()`, `.filter()`, *spread operator* `...`) dan **Event Delegation** pada DOM.

---

### Studi Kasus 1: CMS Artikel & Blog (`/articles`)
**Fokus Pembelajaran:** `GET` + Query Filter, `HEAD` (cek ketersediaan real-time), `POST`, `PUT` (edit penuh), `PATCH` (aksi cepat status/pin), dan `DELETE`.

#### Fitur Wajib di Antarmuka Web:
1. **Daftar Artikel & Filter (`GET /articles`):**
   * Tampilkan daftar artikel lengkap dengan badge status (`published` / `draft`), label **Pinned** jika `isPinned: true`, kategori, tags, dan jumlah `views`.
   * Sediakan filter tab status (`All`, `Published`, `Draft`), input pencarian judul (`?search=...`), serta informasi total artikel yang dibaca dari header `X-Total-Count`.
2. **Validasi Slug Otomatis (`HEAD /articles/check-slug/:slug`):**
   * Pada form pembuatan artikel, saat user mengetik judul atau slug (gunakan *debounce* atau event `blur`), tembak endpoint `HEAD /articles/check-slug/:slug`.
   * Tampilkan indikator hijau *"Slug tersedia"* jika status `404`, atau peringatan merah *"Slug sudah digunakan"* jika status `200`.
3. **Buat Draft Artikel Baru (`POST /articles`):**
   * Form input: `title`, `slug`, `category`, `tags` (pisahkan dengan koma lalu ubah menjadi array), dan `content`.
   * Tangani error `409 Conflict` jika slug duplikat dan tampilkan pesan error di layar.
4. **Editor Naskah Penuh (`PUT /articles/:id`):**
   * Tombol **"Edit Naskah"** membuka form yang terisi data artikel saat ini (`title`, `slug`, `content`, `category`, `tags`, `status`, `isPinned`) dan mengirim seluruh field tersebut menggunakan `PUT`.
5. **Aksi Cepat Dashboard (`PATCH /articles/:id`):**
   * Tombol **"Publish / Unpublish"** di kartu artikel yang hanya mengirim `{ "status": "published" }` atau `{ "status": "draft" }`.
   * Tombol **"Pin / Unpin"** yang hanya mengirim `{ "isPinned": true/false }`.
6. **Hapus Artikel (`DELETE /articles/:id`):**
   * Tombol **"Hapus"** dengan dialog konfirmasi, lalu hapus item dari state tampilan setelah menerima status `204`.

---

### Studi Kasus 2: Sistem Reservasi Ruangan Meeting (`/bookings`)
**Fokus Pembelajaran:** Validasi konflik jadwal (`409 Conflict`), validasi logika waktu (`422`), `PUT` untuk *reschedule* penuh, dan `PATCH` untuk *check-in* resepsionis.

#### Fitur Wajib di Antarmuka Web:
1. **Jadwal Ruangan (`GET /bookings`):**
   * Tampilkan daftar reservasi beserta filter berdasarkan `roomId` (`ROOM-A`, `ROOM-B`, dll.) dan tanggal (`<input type="date">`).
2. **Form Booking Baru (`POST /bookings`):**
   * Input: `roomId`, `bookedBy`, `date`, `startTime`, `endTime`, `participants`, dan `notes`.
   * Tangani respons error dari server secara informatif di UI:
     * `422 Unprocessable Entity` (jika jam selesai lebih awal/sama dengan jam mulai).
     * `409 Conflict` (jika ruangan sudah dibooking pada jam yang bertabrakan).
3. **Reschedule Jadwal (`PUT /bookings/:id`):**
   * Tombol **"Reschedule"** untuk mengubah ruangan, tanggal, jam mulai-selesai, jumlah peserta, dan catatan secara lengkap menggunakan `PUT`.
4. **Aksi Resepsionis (`PATCH /bookings/:id`):**
   * Tombol cepat **"Check-In"** yang hanya mengirim `{ "status": "checked_in" }`.
   * Tombol cepat **"Batalkan"** yang hanya mengirim `{ "status": "cancelled" }`.
5. **Hapus Data Reservasi (`DELETE /bookings/:id`):**
   * Tombol **"Hapus"** untuk menghapus data reservasi secara permanen (`204 No Content`).

---

### Studi Kasus 3: Manajemen Profil & Pengaturan Akun (`/users`)
**Fokus Pembelajaran:** Memahami perbedaan mendasar antara **`PUT`** (mengganti seluruh biodata dan mereset field opsional yang kosong menjadi `null`) dengan **`PATCH`** (*deep merge* pada nested object `preferences`).

#### Fitur Wajib di Antarmuka Web:
1. **Pilih & Registrasi User (`GET /users` & `POST /users`):**
   * Dropdown atau daftar user untuk memilih akun aktif yang sedang dikelola (`GET /users/:id`).
   * Form **"Tambah User Baru"** (`fullName`, `email`, `phone`). Tampilkan pesan error jika server mengembalikan `422` (format email tidak valid atau email sudah terdaftar).
2. **Tab 1 — Edit Biodata Utama (`PUT /users/:id`):**
   * Form berisi `fullName`, `email`, `phone` (wajib), serta `bio`, `address`, dan `avatarUrl` (opsional).
   * *Uji Coba Penting:* Kosongkan input `bio` atau `address` saat menyimpan via `PUT`, lalu perhatikan bahwa server menimpanya menjadi `null` tanpa menghapus pengaturan `preferences` user.
3. **Tab 2 — Pengaturan & Keamanan (`PATCH /users/:id`):**
   * Sediakan *toggle switch* atau *checkbox* untuk `theme` (`light` / `dark`), `twoFactorEnabled`, dan `emailNotifications`.
   * Setiap kali salah satu pengaturan diubah, kirim request `PATCH` yang hanya berisi properti yang berubah di dalam `preferences` (contoh: `{ "preferences": { "twoFactorEnabled": true } }`) dan pastikan pengaturan lainnya tetap utuh.
4. **Hapus Akun (`DELETE /users/:id`):**
   * Tombol **"Hapus Akun"** di bagian bawah pengaturan yang menghapus user (`204 No Content`).

---

### Studi Kasus 4: Manajemen Inventaris & Stock Opname Gudang (`/products`)
**Fokus Pembelajaran:** Query filter bisnis (`?lowStock=true`), revisi master SKU (`PUT`), serta *inline quick update* stok (`PATCH`).

#### Fitur Wajib di Antarmuka Web:
1. **Tabel Inventaris & Filter Stok Menipis (`GET /products`):**
   * Tampilkan daftar produk (`sku`, `name`, `category`, `costPrice`, `sellPrice`, `stock`, `minStock`, `supplier`).
   * Berikan penanda visual merah/kuning jika `stock <= minStock`, serta sediakan checkbox/tombol filter **"Tampilkan Stok Menipis Saja"** (`GET /products?lowStock=true`) dan kotak pencarian SKU/Nama (`?search=...`).
2. **Tambah SKU Baru (`POST /products`):**
   * Form pendaftaran barang baru. Tangani error `409 Conflict` jika kode `sku` sudah terdaftar dan `422` jika harga/stok bernilai negatif.
3. **Revisi Master Data Produk (`PUT /products/:id`):**
   * Modal/Form **"Edit Master Produk"** untuk memperbarui seluruh spesifikasi barang (`sku`, `name`, `category`, `costPrice`, `sellPrice`, `stock`, `minStock`, `supplier`).
4. **Quick Stock Opname (`PATCH /products/:id`):**
   * Langsung di kolom stok pada tabel, sediakan input angka kecil atau tombol **`-1` / `+1` / Update Stok** yang menembak `PATCH /products/:id` dengan payload `{ "stock": nilaiBaru }` tanpa mengirim ulang data harga atau supplier.
5. **Hapus SKU Discontinued (`DELETE /products/:id`):**
   * Tombol **"Hapus"** untuk menghapus produk dari katalog (`204 No Content`).

---

## Standar Penilaian & Checklist Pengumpulan

Sebelum mengumpulkan tugas kepada pembimbing PKL, pastikan aplikasi buatanmu memenuhi seluruh standar berikut:

* [ ] **Async/Await & Error Handling:** Seluruh pemanggilan `fetch()` menggunakan `async/await` di dalam blok `try ... catch ... finally` dan menangani status non-2xx (`400`, `404`, `409`, `422`) dengan menampilkan pesan error yang ramah pengguna di UI (bukan hanya di `console.error`).
* [ ] **Loading State & Pencegahan Double-Submit:** Saat request sedang berlangsung, tombol aksi dinonaktifkan (`disabled = true`) atau menampilkan indikator *loading* hingga request selesai.
* [ ] **Penggunaan HTTP Method Tepat Sasaran:**
  * `PUT` hanya dipakai pada form edit lengkap (menimpa seluruh object).
  * `PATCH` dipakai pada tombol aksi cepat / update parsial (mengirim field yang berubah saja).
  * `DELETE` menangani respons `204 No Content` tanpa error JSON parsing.
* [ ] **Sinkronisasi State Immutable:** Setelah `POST`, `PUT`, `PATCH`, atau `DELETE` sukses, daftar data di UI diperbarui menggunakan pola *immutable* (`.map()`, `.filter()`, `[...arr, newItem]`) atau me-refresh query aktif.
* [ ] **Kode Bersih & Terstruktur:** Pemisahan yang jelas antara layer komunikasi API (`api.js`), pengelolaan state, dan fungsi render DOM (`app.js`).
