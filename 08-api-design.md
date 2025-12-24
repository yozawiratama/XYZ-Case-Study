## 8. API Design (Key Endpoints)

Bagian ini menjelaskan rancangan API backend yang berfungsi sebagai **lapisan penegak aturan bisnis (business rule enforcement layer)** dan **boundary keamanan** antara aplikasi mobile dan data inti sistem. Desain API ini disusun agar konsisten dengan siklus hidup pinjaman, model data, serta perilaku layar yang telah dibahas pada bagian sebelumnya.

### 8.1 Prinsip Desain API dan Boundary Akses Data

Meskipun sistem memanfaatkan Supabase sebagai layanan terkelola (terutama untuk autentikasi), **backend service tetap diperlukan** dan tidak dapat sepenuhnya digantikan oleh query langsung dari client. Prinsip boundary yang digunakan adalah sebagai berikut:

1. **Client hanya melakukan read terbatas untuk data non-kritis**
   Akses baca langsung dari client (misalnya melalui Supabase) hanya diperbolehkan untuk data yang:

   * tidak sensitif,
   * tidak memengaruhi aturan bisnis,
   * dan tidak berdampak pada konsistensi sistem.

   Contohnya: informasi profil dasar atau status ringkas yang sudah disanitasi.

2. **Seluruh Create / Update / Delete (CUD) wajib melalui Backend API**
   Operasi yang mengubah data dan status sistem, khususnya:

   * pengajuan pinjaman,
   * perubahan status pinjaman,
   * pencatatan audit,
   * orkestrasi notifikasi,
     harus dilakukan melalui backend untuk menjamin konsistensi, keamanan, dan auditabilitas.

3. **Backend sebagai Policy Enforcement Layer**
   Backend bertanggung jawab untuk:

   * menegakkan aturan “satu pinjaman aktif/dalam proses”,
   * mencegah race condition akibat retry atau double submit,
   * memastikan transisi status sesuai state machine (Bagian 6).

4. **Minim duplikasi dengan Supabase**
   Backend tidak menggantikan fungsi Supabase (misalnya autentikasi), melainkan memanfaatkannya sebagai *capability*, sehingga kompleksitas dan biaya tetap terkendali.

Prinsip ini umum digunakan pada sistem fintech dan aplikasi dengan data sensitif, serta memberikan keseimbangan antara kecepatan MVP dan kontrol jangka panjang.

---

### 8.2 Kategori API

Secara fungsional, API backend dikelompokkan ke dalam beberapa kategori utama:

1. **Authentication & Session Context**
2. **User & KYC**
3. **Loan Application & Lifecycle**
4. **Repayment & Read Models**
5. **Notification Trigger**
6. **Audit & Observability (Internal)**

---

### 8.3 Authentication & Session Context

Backend mengandalkan Supabase Auth untuk autentikasi dasar (login, refresh token), namun tetap memvalidasi konteks pengguna pada setiap request.

**Tujuan:**

* memastikan setiap request memiliki konteks user yang valid,
* memudahkan penerapan role atau policy tambahan di backend.

**Contoh endpoint:**

```
GET /api/me
```

**Deskripsi:**

* Mengembalikan ringkasan data user yang sedang login.
* Digunakan oleh aplikasi mobile saat startup setelah login.

**Catatan desain:**

* Endpoint ini bersifat read-only.
* Data yang dikembalikan sudah disanitasi untuk kebutuhan UI.

---

### 8.4 User & KYC API

API pada kategori ini digunakan untuk mengelola data pengguna dan status KYC, tanpa mengekspos detail sensitif secara langsung ke client.

#### Submit / Update KYC

```
POST /api/kyc
```

**Deskripsi:**

* Mencatat referensi dokumen KYC (KTP dan foto) yang telah diunggah.
* Mengubah status KYC menjadi *Submitted*.

**Validasi utama:**

* User terautentikasi.
* Dokumen referensi tersedia.

**Side effects:**

* Menulis audit log `KYC_SUBMITTED`.

**Alasan melalui backend:**

* Status KYC memengaruhi kelayakan pengajuan pinjaman.
* Perubahan status harus tercatat secara konsisten.

---

### 8.5 Loan Application & Lifecycle API (Core)

Ini adalah **bagian paling kritis** dari API karena langsung berkaitan dengan aturan bisnis utama.

#### Submit Loan Application

```
POST /api/loans
```

**Deskripsi:**

* Membuat pengajuan pinjaman baru.

**Validasi (SELF-CHECK dengan Bagian 6 & 7):**

* User terautentikasi.
* KYC minimal tersedia (upload KTP & foto).
* Nilai pinjaman ≤ Rp 12.000.000.
* Tenor ≤ 12 bulan.
* Tidak ada pinjaman lain dengan status:
  `SUBMITTED`, `IN_REVIEW`, atau `ACTIVE`.

**Side effects:**

* Membuat record Loan dengan status `SUBMITTED`.
* Mengisi snapshot data (nama user, status KYC).
* Menulis audit log `LOAN_SUBMITTED`.

**Alasan tidak boleh client direct query:**

* Aturan “single active loan” harus atomic.
* Mencegah race condition akibat double submit.

---

#### Get Active / Latest Loan

```
GET /api/loans/active
```

**Deskripsi:**

* Mengembalikan pinjaman aktif atau pengajuan terakhir milik user.

**Digunakan oleh:**

* Dashboard (Bagian 7.3.3).
* Penentuan apakah tombol *Ajukan Pinjaman* aktif atau tidak.

**Catatan desain:**

* Endpoint ini read-only dan read-optimized.
* Boleh memanfaatkan snapshot data untuk performa.

---

#### Get Loan Detail

```
GET /api/loans/{loanId}
```

**Deskripsi:**

* Mengembalikan detail pinjaman atau pengajuan.

**Validasi:**

* Loan milik user yang sedang login.

---

### 8.6 Repayment & Read Model API

#### Get Repayment Schedule

```
GET /api/loans/{loanId}/repayments
```

**Deskripsi:**

* Mengembalikan daftar tagihan bulanan untuk pinjaman aktif.

**Kondisi:**

* Hanya tersedia jika status pinjaman `ACTIVE`.

**SELF-CHECK:**

* Konsisten dengan Bagian 7.3.6 (Repayment Schedule Screen).
* Data berasal dari schedule yang di-generate saat pinjaman disetujui (Bagian 6).

---

### 8.7 Notification Trigger API (Internal)

Notifikasi email dan SMS tidak dipicu langsung dari client.

```
POST /internal/notifications/loan-approved
```

**Deskripsi:**

* Endpoint internal untuk memicu pengiriman notifikasi persetujuan pinjaman.

**Catatan desain:**

* Dipanggil sebagai side effect dari perubahan status `IN_REVIEW → APPROVED`.
* Tidak diekspos ke client.

---

### 8.8 Audit & Observability

Setiap endpoint kritis secara implisit:

* mencatat audit log (aksi, waktu, reference ID),
* memungkinkan tracing antar service (jika dikembangkan lebih lanjut).

Audit log **tidak** diisi oleh client dan **tidak** dapat dimanipulasi dari luar backend.
