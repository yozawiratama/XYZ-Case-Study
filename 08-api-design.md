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

---

### 8.7 Notification Trigger API (Internal)

Bagian ini mendefinisikan kontrak API internal untuk memicu pengiriman notifikasi melalui **Notification Orchestrator** (lihat Bagian 4.6). Notifikasi diposisikan sebagai proses *reliability-first* yang tidak dipicu dari client, serta wajib mendukung idempotency dan retry.

#### 8.7.1 Notification Flow (End-to-End)

Alur pengiriman notifikasi untuk event kritis (contoh: *Loan Approved*) adalah sebagai berikut:

1. **On-Prem Core** melakukan transisi status pinjaman `IN_REVIEW → APPROVED` (Bagian 6).
2. Core menulis **audit log** untuk perubahan status.
3. Core memanggil **Internal Notification Trigger API** ke Notification Orchestrator melalui jalur aman (VPN/Private Link).
4. Notification Orchestrator:

   * memvalidasi event (schema & signature jika ada),
   * melakukan **idempotency check**,
   * menyusun pesan berdasarkan template,
   * mengirim ke provider (email/SMS/push) sesuai policy,
   * menyimpan status attempt dan hasil.
5. Jika pengiriman gagal sementara, orchestrator melakukan retry sesuai kebijakan (lihat 8.7.4).

Catatan desain: Core hanya bertanggung jawab pada “*emit event*” dan audit internal; pengiriman dan reliability delivery menjadi tanggung jawab Notification Orchestrator.


#### 8.7.2 Internal Endpoint dan Payload Contract

**Endpoint internal (contoh):**

```
POST /internal/notifications/events
```

**Tujuan:**

* menerima event notifikasi dari core,
* memprosesnya secara deterministik dan idempotent.

**Contoh payload (ringkas, non-PII heavy):**

```json
{
  "event_id": "evt_01JHXXXXXX",
  "event_type": "LOAN_APPROVED",
  "occurred_at": "2025-12-29T10:15:30+07:00",
  "subject": {
    "user_id": "uuid-user",
    "loan_id": "uuid-loan"
  },
  "channels": ["EMAIL", "SMS", "PUSH"],
  "template": {
    "name": "loan_approved_v1",
    "variables": {
      "loan_reference": "LN-20251229-0001"
    }
  }
}
```

**Prinsip payload:**

* **Minim data sensitif**: tidak mengirim nominal, NIK, atau detail KYC lewat event.
* Template variables hanya membawa data yang aman; detail diambil oleh aplikasi melalui API saat user membuka aplikasi.
* `event_id` wajib unik untuk mendukung idempotency.


#### 8.7.3 Idempotency & Deduplication

Karena notifikasi rawan dipicu ulang akibat retry, network issue, atau reprocessing, maka strategi idempotency menjadi wajib.

**Idempotency Key:**

* Level event: `event_id`
* Level event-channel: `event_id + channel`

**Aturan:**

1. Jika Notification Orchestrator menerima `event_id` yang sudah pernah diproses:

   * tidak mengirim ulang (deduplicate),
   * mengembalikan respons “already processed”.
2. Jika event sama diproses sebagian (misalnya email sukses, SMS gagal):

   * orchestrator hanya mengulang kanal yang gagal (berdasarkan `event_id+channel`).

**Penyimpanan status minimal yang dibutuhkan:**

* `event_id`
* `channel`
* `attempt_count`
* `last_attempt_at`
* `status` (PENDING/SENT/FAILED)
* `provider_response_code` (disanitasi)


#### 8.7.4 Retry Strategy & Failure Handling

Notifikasi harus tahan terhadap kegagalan sementara (temporary failure) dari provider.

**Klasifikasi kegagalan:**

* **Temporary failure**: timeout, 5xx provider, rate limit → eligible untuk retry.
* **Permanent failure**: nomor tidak valid, email bounce hard, template invalid → tidak di-retry, masuk dead-letter/failed state.

**Kebijakan retry (target design, dapat disederhanakan di MVP):**

* Exponential backoff (misal: 1m, 5m, 15m, 60m)
* Maksimum attempt per channel (misal: 5x)
* Setelah melewati batas attempt:

  * status menjadi `FAILED_PERMANENT`,
  * dicatat untuk tindakan manual / investigasi.

**Queue vs non-queue:**

* MVP: retry dapat dilakukan di orchestrator dengan mekanisme scheduler sederhana.
* Scale stage: gunakan message queue (dibahas di Bagian 10) untuk menguatkan reliability dan throughput.


#### 8.7.5 Respons Endpoint (Internal)

Contoh respons sukses:

```json
{
  "event_id": "evt_01JHXXXXXX",
  "status": "ACCEPTED",
  "deduplicated": false
}
```

Jika event duplicate:

```json
{
  "event_id": "evt_01JHXXXXXX",
  "status": "ALREADY_PROCESSED",
  "deduplicated": true
}
```

Catatan: Endpoint internal ini tidak harus menunggu delivery selesai; cukup menerima dan memproses asynchronous di sisi orchestrator.

---

### 8.8 Audit & Observability

Selain audit log di core (perubahan status loan), notifikasi memerlukan observability khusus untuk memastikan sistem dapat dioperasikan dengan baik.

#### 8.8.1 Notification Delivery Logging (Orchestrator)

Setiap attempt pengiriman mencatat:

* `event_id`
* `channel`
* `attempt_count`
* `provider_name`
* `result` (SENT / TEMP_FAILED / PERM_FAILED)
* `timestamp`

**Prinsip keamanan log:**

* tidak menyimpan payload sensitif,
* response provider disanitasi (hindari menyimpan full message body).

#### 8.8.2 Metrics yang Disarankan

* delivery success rate per channel
* retry rate
* permanent failure rate
* provider latency (p50/p95)
* queue backlog (jika nanti menggunakan queue)
