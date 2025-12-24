## 7. Screen Flow & User Behavior

Bagian ini menjelaskan alur layar aplikasi dan perilaku pengguna berdasarkan **status autentikasi, status KYC, dan status pinjaman**. Pendekatan yang digunakan bersifat *state-driven*, di mana tampilan dan interaksi yang tersedia bagi pengguna ditentukan oleh kondisi sistem, bukan sekadar urutan layar statis.

### 7.1 Prinsip Desain Screen Flow

1. **State-driven UI**
   Tampilan dan aksi yang tersedia pada setiap layar ditentukan oleh status pinjaman (`loan_status`) dan status pengguna, sehingga menghindari inkonsistensi antara UI dan logika backend.

2. **Fast Read, Controlled Write**
   Informasi penting seperti status pinjaman dan tagihan ditampilkan secepat mungkin (read-optimized), sementara aksi yang mengubah data (submit, approve, bayar) diberikan feedback yang jelas meskipun memerlukan waktu proses.

3. **Explicit Blocking Behavior**
   Jika pengguna tidak diperbolehkan melakukan suatu aksi (misalnya mengajukan pinjaman baru), sistem harus menjelaskan alasannya secara eksplisit, bukan sekadar menonaktifkan tombol tanpa konteks.

4. **Predictable Navigation**
   Pengguna selalu diarahkan kembali ke layar yang relevan dengan status pinjaman mereka saat ini, terutama setelah login.

---

### 7.2 Daftar Screen Utama

Screen utama dalam aplikasi ini meliputi:

1. **Login Screen**
2. **Registration & KYC Screen**
3. **Dashboard**
4. **Loan Application Screen**
5. **Loan Status Detail Screen**
6. **Repayment Schedule Screen**
7. **Notification & Message Feedback**

---

### 7.3 Perilaku Screen Berdasarkan Kondisi

#### 7.3.1 Login Screen

* Input: email/nomor telepon + password.
* Opsi biometrik muncul jika device mendukung dan user sudah pernah login.
* Setelah login berhasil:

  * sistem mengambil status pinjaman terakhir,
  * user langsung diarahkan ke **Dashboard**.

#### 7.3.2 Registration & KYC Screen

* Digunakan untuk:

  * pendaftaran akun baru,
  * unggah KTP dan foto pengguna.
* User tidak dapat melanjutkan ke pengajuan pinjaman jika data KYC belum tersedia.
* Status KYC ditampilkan secara ringkas (misal: *Submitted*, *Verified*).

---

#### 7.3.3 Dashboard (Entry Point Utama)

Dashboard adalah layar pertama setelah login dan berfungsi sebagai **ringkasan kondisi pengguna**.

Perilaku dashboard bergantung pada status pinjaman:

* **Tidak ada pinjaman**

  * Tombol *Ajukan Pinjaman* aktif.
  * Ditampilkan informasi batas maksimum pinjaman.

* **Pinjaman dengan status SUBMITTED / IN_REVIEW**

  * Ditampilkan status “Pengajuan sedang diproses”.
  * Tombol *Ajukan Pinjaman* dinonaktifkan.
  * Ditampilkan alasan: *“Anda masih memiliki pengajuan pinjaman yang sedang diproses.”*

* **Pinjaman dengan status ACTIVE**

  * Ditampilkan:

    * sisa hutang,
    * tagihan terdekat,
    * status pembayaran.
  * Tombol *Ajukan Pinjaman* dinonaktifkan.
  * Ditampilkan alasan: *“Pinjaman Anda belum lunas.”*

* **Pinjaman dengan status PAID_OFF**

  * Ditampilkan status pinjaman terakhir sebagai lunas.
  * Tombol *Ajukan Pinjaman* kembali aktif.

---

#### 7.3.4 Loan Application Screen

* Berisi input:

  * jumlah pinjaman (≤ Rp 12.000.000),
  * tenor (≤ 12 bulan).
* Validasi dilakukan:

  * secara client-side (range nilai),
  * dan server-side (aturan bisnis).
* Jika submit berhasil:

  * user diarahkan ke Dashboard,
  * status berubah menjadi `SUBMITTED`.

---

#### 7.3.5 Loan Status Detail Screen

* Menampilkan detail pengajuan atau pinjaman:

  * status saat ini,
  * tanggal pengajuan/persetujuan,
  * ringkasan pinjaman.
* Untuk status `REJECTED`, ditampilkan pesan penolakan secara umum (tanpa detail sensitif).

---

#### 7.3.6 Repayment Schedule Screen

* Tersedia hanya untuk pinjaman dengan status `ACTIVE`.
* Menampilkan:

  * daftar tagihan bulanan,
  * tanggal jatuh tempo,
  * status pembayaran.
* Data ditarik dari schedule yang telah digenerate saat pinjaman disetujui.

---

### 7.4 Mapping Screen terhadap Loan State

| Loan State | Dashboard | Apply Loan | Status Detail | Repayment |
| ---------- | --------- | ---------- | ------------- | --------- |
| No Loan    | ✓         | ✓          | –             | –         |
| SUBMITTED  | ✓         | ✕          | ✓             | –         |
| IN_REVIEW  | ✓         | ✕          | ✓             | –         |
| APPROVED   | ✓         | ✕          | ✓             | –         |
| ACTIVE     | ✓         | ✕          | ✓             | ✓         |
| PAID_OFF   | ✓         | ✓          | ✓             | –         |
| REJECTED   | ✓         | ✓          | ✓             | –         |

---

### 7.5 Error Handling & Edge Cases

* **Double submit pengajuan**
  Backend mengembalikan error terstruktur dan UI menampilkan pesan bahwa pengajuan sedang diproses.

* **Network timeout saat submit**
  UI menampilkan status “Sedang diproses” dan mencegah submit ulang sampai status dikonfirmasi.

* **Mismatch data snapshot**
  Jika terjadi perbedaan antara snapshot dan status aktual, UI wajib mengikuti status dari system of record.

