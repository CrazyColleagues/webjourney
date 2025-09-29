---
marp: true
---

# Materi Pengembangan Web Modern: Dari Arsitektur Hingga Praktik Terbaik
### Prinsip Dasar untuk Kode yang Bersih, Aman, dan Terukur

---

## 1. Pendekatan Arsitektur Web Dasar: Monolith (MVC/MVP)
 Monolit
**Monolith** adalah model arsitektur di mana semua fungsionalitas aplikasi (UI, Logika Bisnis, Akses Data) digabungkan ke dalam **satu basis kode tunggal** (repository) dan di-deploy sebagai satu unit.

*   **Kelebihan**: Sederhana untuk dikembangkan dan di-deploy di awal.
*   **Kekurangan**: Sulit untuk diskalakan secara independen dan membutuhkan *full redeploy* untuk perubahan kecil.

---

### Konteks & Pola Arsitektur

Untuk mengelola kompleksitas dalam monolit, kita menggunakan pola arsitektur untuk memisahkan tanggung jawab (*Separation of Concerns*).

| Pola | Model | View | Controller/Presenter | Hubungan Kunci |
| :--- | :--- | :--- | :--- | :--- |
| **MVC** | Data & Logika Bisnis | Antarmuka Pengguna (UI) | Menerima input, memanipulasi Model, memilih View. | Controller **memilih** View. |
| **MVP** | Data & Logika Bisnis | Antarmuka Pengguna (UI) (Pasif) | Mengambil input, memanipulasi Model, **memperbarui** View. | Presenter **memiliki referensi** ke View. |

---

### Aplikasi Dunia Nyata

*   **MVC**: Umum di *framework* sisi server seperti **Ruby on Rails**, **Django (Python)**, dan **Laravel (PHP)**.
*   **MVP**: Umum di aplikasi yang membutuhkan interaksi UI yang lebih ketat dan pengujian logika presentasi yang mudah, seperti **Aplikasi Android** (dengan *Presenter* yang mengelola *Activity/Fragment*).

---

## 2. Middleware dan Role-Based Access Control (RBAC)

### Middleware: Penjaga Gerbang Aplikasi

**Middleware** adalah lapisan perangkat lunak yang memproses permintaan HTTP **sebelum** mencapai logika utama aplikasi (Controller) dan **sebelum** respons dikirim kembali.

Middleware ideal untuk fungsionalitas lintas-fungsi (*cross-cutting concerns*):
*   **Otentikasi** (Memeriksa identitas pengguna).
*   **Otorisasi** (Memeriksa izin pengguna).
*   **Logging** (Mencatat permintaan).
*   **Validasi Input**.

---

### Role-Based Access Control (RBAC)

**RBAC** adalah metode otorisasi di mana hak akses ke sumber daya sistem ditentukan berdasarkan **Peran** (*Role*) yang dimiliki pengguna.

| Entitas | Deskripsi | Contoh |
| :--- | :--- | :--- |
| **User** | Individu yang mengakses sistem. | Budi, Ani |
| **Role** | Kumpulan izin yang telah ditentukan. | Admin, Editor, Guest |
| **Permission** | Hak untuk melakukan tindakan tertentu. | `create_post`, `delete_user`, `view_dashboard` |

---

### Use Case RBAC dengan Middleware (Pseudocode)

```javascript
// Middleware untuk memeriksa peran 'Admin'
function isAdmin(request, response, next) {
  const user = request.user; // Data user dari middleware Otentikasi

  if (user && user.role === 'Admin') {
    next(); // Lanjutkan ke Controller
  } else {
    // Menghentikan request dan mengirim error 403 Forbidden
    response.status(403).send('Akses ditolak: Peran tidak diizinkan.');
  }
}

// Penerapan pada route
app.delete('/posts/:id', isAuthenticated, isAdmin, (request, response) => {
  // Logika Controller: Hanya dieksekusi jika user adalah Admin
  deletePost(request.params.id);
  response.send('Post berhasil dihapus.');
});
```

---

## 3. Enkripsi dan Hashing untuk Otentikasi

### Hashing: Mekanisme Satu Arah

| Konsep | Enkripsi | Hashing |
| :--- | :--- | :--- |
| **Arah** | Dua Arah (Dapat didekripsi) | Satu Arah (Tidak dapat dikembalikan) |
| **Tujuan** | Melindungi data saat transit (SSL/TLS) atau saat disimpan (*at rest*). | Memverifikasi integritas data dan menyimpan kata sandi. |
| **Contoh Algoritma** | AES, RSA | **bcrypt**, **Argon2**, SHA-256 (untuk integritas) |

---

### Mengapa Hashing untuk Kata Sandi?

Menyimpan kata sandi dalam bentuk *plaintext* atau *terenkripsi* adalah **sangat tidak aman**.

1.  **Keamanan**: Jika *database* bocor, penyerang hanya mendapatkan *hash* (string acak), bukan kata sandi asli.
2.  **Verifikasi**: Sistem membandingkan *hash* dari input login dengan *hash* yang tersimpan. Jika cocok, kata sandi benar.

---

### Proses Otentikasi yang Aman

1.  **Registrasi**:
    *   `Password` + `Salt` (string acak unik) $\rightarrow$ **Hashing** (e.g., bcrypt) $\rightarrow$ `Hash`
    *   Simpan `Hash` dan `Salt` di database.
2.  **Login**:
    *   Ambil `Hash` dan `Salt` dari database.
    *   `Input Password` + `Salt` $\rightarrow$ **Hashing** $\rightarrow$ `New Hash`
    *   Bandingkan `New Hash` dengan `Hash` yang tersimpan.

---

## 4. Conventional Commits
 & Struktu
**Conventional Commits** adalah konvensi penulisan pesan commit Git untuk menciptakan riwayat yang eksplisit dan memungkinkan otomatisasi (misalnya, *changelog* otomatis).

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

---

### Jenis-jenis Commit (`<type>`)

| Type | Deskripsi | Semantic Versioning |
| :--- | :--- | :--- |
| **feat** | Menambahkan fitur baru. | `MINOR` (e.g., v1.0.0 $\rightarrow$ v1.1.0) |
| **fix** | Memperbaiki bug. | `PATCH` (e.g., v1.0.0 $\rightarrow$ v1.0.1) |
| **docs** | Perubahan pada dokumentasi. | - |
| **refactor** | Perubahan kode tanpa menambah fitur/memperbaiki bug. | - |
| **chore** | Perubahan pada proses build, dependensi, atau tugas lain. | - |
| **BREAKING CHANGE** | Perubahan yang merusak kompatibilitas API. | `MAJOR` (e.g., v1.0.0 $\rightarrow$ v2.0.0) |

---

### Contoh Commit

```
fix: perbaiki typo pada validasi login pengguna
```

```
feat(auth): implementasi fitur reset password
```

```
refactor!: hapus dukungan untuk Node.js 12

BREAKING CHANGE: Versi minimum Node.js yang dibutuhkan sekarang adalah 14.
```

---

## 5. Manajemen File & Struktur Proyek

### Prinsip Dasar

Manajemen file yang baik adalah tentang **keterbacaan** dan **pemeliharaan**.

1.  **Struktur Direktori yang Jelas**: Kelompokkan file berdasarkan **fungsi** (e.g., `/controllers`, `/services`) atau **fitur** (e.g., `/user`, `/product`).
2.  **Pemisahan Tanggung Jawab**: Setiap file/direktori harus memiliki satu tujuan yang jelas (sejalan dengan SRP).
3.  **Hindari *Deep Nesting***: Struktur yang terlalu dalam (lebih dari 3-4 level) menyulitkan navigasi.

---

### Contoh Struktur Proyek (General)

```
/
├── config/        # Konfigurasi aplikasi (DB, server, dll.)
├── src/           # Kode Sumber Utama
│   ├── api/       # Lapisan API/Controller
│   ├── core/      # Logika Bisnis Inti (Services)
│   ├── data/      # Lapisan Akses Data (Models/Repositories)
│   └── utils/     # Fungsi-fungsi bantuan (Helpers)
├── tests/         # Semua file pengujian
├── docs/          # Dokumentasi proyek
├── .env           # Variabel lingkungan
└── package.json   # Metadata proyek
```

---

## 6. Conventional Naming (Konvensi Penamaan)
Konvensi penamaan adalah seperangkat aturan untuk memilih *identifier* (variabel, fungsi, kelas, file) agar kode mudah dibaca dan dipahami.

| Gaya | Contoh | Penggunaan Umum |
| :--- | :--- | :--- |
| **camelCase** | `totalUsers` | Variabel, Fungsi (JavaScript, Java) |
| **PascalCase** | `UserAccount`, `HttpRequest` | Kelas, Komponen (Semua Bahasa) |
| **snake_case** | `total_users` | Variabel, Fungsi (Python), Nama Kolom DB |
| **kebab-case** | `user-profile.js` | Nama File, CSS Classes |
| **UPPER_CASE** | `API_KEY` | Konstanta Global |

---

### Praktik Terbaik

*   **Deskriptif**: Gunakan nama yang jelas. `userList` lebih baik daripada `ul`.
*   **Konsisten**: Terapkan satu gaya secara konsisten untuk setiap jenis *identifier* di seluruh proyek.
*   **Ikuti Konvensi Bahasa**: Patuhi panduan gaya resmi bahasa yang digunakan (misalnya, **PEP 8** untuk Python).

---

## 7. Single Responsibility Principle (SRP)
**SRP** (Prinsip Tanggung Jawab Tunggal) adalah prinsip pertama dari SOLID.

> **"Sebuah kelas atau modul hanya boleh memiliki satu alasan untuk berubah."**

Setiap unit kode harus memiliki **satu tanggung jawab tunggal** yang terdefinisi dengan baik.

---

### Pelanggaran SRP (Contoh Umum)

Kelas yang melakukan lebih dari satu tugas utama.

```java
// Pelanggaran SRP: Dua alasan untuk berubah
class User {
    // Tanggung jawab 1: Mengelola data pengguna
    private String name;
    public String getName() { ... }

    // Tanggung jawab 2: Menyimpan data ke database
    public void saveToDatabase() {
        // Logika koneksi dan penyimpanan DB
    }
}
```
*   **Alasan Berubah 1**: Jika struktur data pengguna berubah.
*   **Alasan Berubah 2**: Jika mekanisme penyimpanan data (DB) berubah.

---

### Penerapan SRP yang Benar

Pisahkan tanggung jawab ke dalam kelas yang berbeda.

```java
// Kelas 1: Hanya bertanggung jawab atas data (Entity/Model)
class User {
    private String name;
    private String email;
    // getters and setters
}

// Kelas 2: Hanya bertanggung jawab atas persistensi data (Repository/DAO)
class UserRepository {
    public void save(User user) {
        // Logika koneksi dan penyimpanan ke DB
    }
}
```
Setiap kelas kini memiliki satu alasan untuk berubah, membuat kode lebih modular dan mudah diuji.

---

## 8. Linter, Code Formatter, dan Style Guide

### Linter dan Code Formatter

| Alat | Fungsi Utama | Contoh Alat |
| :--- | :--- | :--- |
| **Linter** | **Analisis Statis**: Mengidentifikasi *bug* potensial, masalah gaya, dan praktik buruk dalam kode. | **ESLint** (JS), **Pylint** (Python) |
| **Formatter** | **Otomatisasi Gaya**: Secara otomatis mengatur ulang kode agar sesuai dengan aturan gaya yang ditentukan (spasi, *indentation*, titik koma). | **Prettier**, **Black** (Python) |

**Tujuan**: Menegakkan konsistensi kode di seluruh tim dan menangkap kesalahan sebelum kompilasi/eksekusi.

---

### Style Guide (Panduan Gaya)

**Style Guide** adalah seperangkat aturan dan rekomendasi tentang bagaimana kode harus ditulis. Ini adalah fondasi yang digunakan oleh Linter dan Formatter.

*   **Contoh Populer**:
    *   **Airbnb Style Guide (JavaScript)**: Salah satu panduan gaya JavaScript paling komprehensif dan ketat.
    *   **PEP 8 (Python)**: Panduan gaya resmi untuk kode Python.
    *   **Google Style Guides**: Tersedia untuk berbagai bahasa (Java, C++, Go, dll.).

**Pentingnya**: Memastikan semua anggota tim menulis kode dengan cara yang sama, mengurangi *cognitive load*, dan meningkatkan *code review*.