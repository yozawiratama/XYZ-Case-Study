## 9. Security, Compliance & Audit

Bagian ini menjelaskan pendekatan keamanan, kepatuhan regulasi, serta mekanisme audit yang diterapkan pada sistem. Mengingat aplikasi ini bergerak di domain pinjaman (fintech), perlindungan data pengguna dan integritas proses bisnis merupakan aspek yang **tidak dapat ditawar** dan harus dipertimbangkan sejak tahap desain.

### 9.1 Security

Keamanan sistem dirancang secara **berlapis (defense in depth)**, mencakup keamanan komunikasi, autentikasi, otorisasi, serta perlindungan data sensitif.

#### 9.1.1 Keamanan Komunikasi

* Seluruh komunikasi antara aplikasi mobile, cloud edge, dan backend core **wajib menggunakan HTTPS (TLS)**.
* Akses ke on-prem core system dilakukan melalui **VPN atau private link**, sehingga tidak terekspos ke jaringan publik.

#### 9.1.2 Autentikasi dan Manajemen Token

* Autentikasi pengguna memanfaatkan **Supabase Auth** sebagai sistem identitas terkelola.
* Backend menggunakan **JWT (JSON Web Token)** sebagai mekanisme pembawa konteks autentikasi.

**Pertimbangan desain JWT:**

* Payload JWT hanya berisi informasi minimal yang diperlukan (misalnya `user_id`, `session_id`, `token_expiry`).
* Informasi sensitif tidak disimpan dalam bentuk *plain text* pada payload.
* Data penting dalam token dienkripsi (*encryptable & decryptable*) oleh backend, untuk mengurangi risiko eksploitasi melalui inspeksi token (misalnya menggunakan alat publik seperti jwt.io).
* Validasi token selalu dilakukan di backend, bukan hanya di sisi client.

Pendekatan ini bertujuan untuk:

* mengurangi risiko kebocoran data jika token terekspos,
* menjaga keseimbangan antara performa dan keamanan.

#### 9.1.3 Otorisasi dan Boundary Akses

* Setiap endpoint backend memverifikasi:

  * identitas pengguna,
  * kepemilikan resource (misalnya loan milik user tersebut).
* Operasi sensitif (Create/Update/Delete) **tidak pernah** dilakukan langsung oleh client melalui query database.
* Backend bertindak sebagai *policy enforcement layer* untuk mencegah bypass aturan bisnis.

#### 9.1.4 Perlindungan Data Sensitif

* Data identitas pengguna (KYC) dipisahkan antara:

  * metadata (disimpan di database),
  * file fisik (disimpan di object storage).
* Akses ke dokumen KYC dibatasi hanya untuk proses backend yang relevan.
* Password pengguna disimpan dalam bentuk hash menggunakan algoritma yang aman (ditangani oleh Supabase Auth).

---

### 9.2 Compliance

Sistem dirancang dengan mempertimbangkan kepatuhan terhadap regulasi yang relevan dengan layanan pinjaman dan pengelolaan data pribadi di Indonesia.

#### 9.2.1 Kepatuhan terhadap Regulasi Keuangan

* Proses pengajuan dan persetujuan pinjaman dirancang agar **dapat ditelusuri dan diaudit**, sesuai dengan praktik yang umumnya diwajibkan oleh regulator seperti **OJK**.
* Aturan bisnis utama (misalnya pembatasan satu pinjaman aktif) diterapkan secara konsisten untuk mitigasi risiko kredit.

#### 9.2.2 Perlindungan Data Pribadi

* Pengelolaan data pengguna memperhatikan prinsip:

  * *data minimization* (hanya menyimpan data yang diperlukan),
  * *purpose limitation* (data digunakan sesuai tujuan bisnis),
  * *access control* (pembatasan akses internal).
* Data sensitif tidak disebarkan ke layanan analitik atau pihak ketiga dalam bentuk mentah.

#### 9.2.3 Kesiapan Audit dan Pemeriksaan

* Struktur data dan log dirancang agar siap mendukung:

  * audit internal,
  * pemeriksaan regulator,
  * investigasi insiden jika diperlukan.
* Desain ini memungkinkan penambahan kebijakan retensi data dan prosedur penghapusan sesuai regulasi di tahap pengembangan berikutnya.

---

### 9.3 Audit

Audit diperlakukan sebagai **bagian inti dari sistem**, bukan fitur tambahan.

#### 9.3.1 Audit Log

Sistem menyediakan **Audit Log** terpusat untuk mencatat aktivitas penting, antara lain:

* registrasi dan login pengguna,
* pengiriman dan perubahan status KYC,
* pengajuan pinjaman,
* perubahan status pinjaman (submitted, approved, rejected, active, paid off),
* pemicu notifikasi penting.

Setiap audit log minimal mencatat:

* `reference_type` (misalnya USER, LOAN, KYC),
* `reference_id`,
* `action`,
* `timestamp`,
* konteks pelaku (user atau sistem).

Audit log:

* ditulis oleh backend,
* tidak dapat dimodifikasi oleh client,
* dan dapat diperlakukan sebagai *append-only*.

#### 9.3.2 Konsistensi Audit dengan State Machine

Setiap transisi status pada *Loan Lifecycle* (Bagian 6) **wajib menghasilkan audit log**.
Hal ini memastikan bahwa:

* tidak ada perubahan status “sunyi” tanpa jejak,
* urutan kejadian dapat direkonstruksi saat terjadi sengketa atau investigasi.

