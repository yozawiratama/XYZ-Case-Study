## 4. Arsitektur Sistem Tingkat Tinggi

Arsitektur sistem dirancang dengan tujuan utama untuk mendukung pengembangan MVP yang cepat, stabil, dan terkontrol, tanpa mengorbankan prinsip rekayasa perangkat lunak yang baik. Desain ini menempatkan fondasi yang jelas sejak awal agar sistem tetap mudah dikembangkan, minim risiko regresi, dan tidak menimbulkan tumpang tindih logika ketika terjadi penambahan fitur atau perubahan kebutuhan di kemudian hari.

Pendekatan yang digunakan adalah arsitektur _hybrid_, dengan memanfaatkan kemampuan cloud untuk kebutuhan *edge* dan non-core, serta menjaga komponen inti dan data sensitif tetap berada pada lingkungan yang lebih terkontrol.

---

### 4.1 Prinsip Arsitektur

Perancangan arsitektur sistem ini didasarkan pada prinsip-prinsip berikut:

1. **Proven MVP-first & Rapid Development**
   Arsitektur dirancang agar memungkinkan pengembangan MVP secara cepat dan efisien, menggunakan komponen dan teknologi yang telah terbukti stabil, dengan risiko bug dan kompleksitas operasional yang rendah.

2. **Separation of Concerns**
   Setiap lapisan dan layanan memiliki tanggung jawab yang jelas, untuk mencegah pencampuran logika bisnis, logika integrasi, dan logika presentasi.

3. **System of Record berada di Core**
   Seluruh data transaksi, status pinjaman, dan audit log diperlakukan sebagai *single source of truth* dan dikelola pada lapisan core system.

4. **Security & Data Sensitivity by Design**
   Data sensitif seperti identitas pengguna, dokumen KYC, dan data transaksi tidak dipaparkan langsung ke lapisan publik, serta dipisahkan dari layanan non-kritis.

5. **Maintainability & Change Readiness**
   Backend dirancang dengan prinsip SOLID dan memungkinkan penerapan pola seperti CQRS untuk meminimalkan dampak perubahan terhadap kode yang sudah ada.

---

### 4.2 Gambaran Arsitektur Hybrid

Arsitektur sistem dibagi ke dalam dua domain utama:

* **Cloud Edge Layer**
  Digunakan untuk menangani kebutuhan yang bersifat:

  * publik,
  * elastis,
  * berorientasi distribusi dan proteksi,
    seperti API gateway, autentikasi pengguna, notifikasi, dan analitik.

* **On-Prem Core System**
  Digunakan untuk mengelola:

  * proses bisnis inti,
  * data transaksi,
  * data KYC,
  * dan audit log,
    dengan kontrol penuh terhadap keamanan, konsistensi, dan integritas data.

Kedua domain ini dihubungkan melalui koneksi yang aman menggunakan **VPN atau private link**, untuk memastikan komunikasi data berlangsung secara terenkripsi dan terbatas.

---

### 4.3 Diagram Arsitektur Tingkat Tinggi

```mermaid
flowchart LR
    subgraph Mobile["Flutter Mobile App"]
        M1[UI / UX]
        M2[Biometric Authentication]
    end

    subgraph Edge["Cloud Edge Layer"]
        E1[API Gateway / WAF]
        E2[Auth Service]
        E3[Notification Orchestrator]
        E4[Analytics & Telemetry]
    end

    subgraph Secure["Secure Network"]
        S1[VPN / Private Link]
    end

    subgraph Core["On-Prem Core System"]
        C1[User & KYC Service]
        C2[Loan Management Service\n(CQRS-ready)]
        C3[Loan Ledger / Accounting]
        C4[Audit Log Service]
    end

    subgraph Data["On-Prem Data Layer"]
        D1[(PostgreSQL\nTransactional Data)]
        D2[(Object Storage\nKYC Files)]
    end

    M1 -->|HTTPS| E1
    M2 --> E2

    E1 --> E2
    E1 --> E3
    E1 --> S1

    S1 --> C1
    S1 --> C2
    S1 --> C3
    S1 --> C4

    C1 --> D1
    C1 --> D2
    C2 --> D1
    C3 --> D1
    C4 --> D1
```

---

### 4.4 Penjelasan Komponen Utama

**Flutter Mobile Application**
Berfungsi sebagai antarmuka pengguna. Autentikasi biometrik dilakukan di sisi perangkat dan dikaitkan dengan mekanisme token di backend.

**Cloud Edge Layer**
Menjadi lapisan pelindung dan orkestrator awal, mencakup:

* API Gateway dan WAF untuk proteksi dan kontrol trafik.
* Layanan autentikasi terkelola untuk mempercepat pengembangan MVP.
* Orkestrasi notifikasi (email, SMS, push notification).
* Pengumpulan data analitik berbasis event non-PII.

**On-Prem Core System**
Menjadi pusat logika bisnis dan data, mencakup:

* Pengelolaan pengguna dan status KYC.
* Manajemen pinjaman dengan kontrol status dan validasi aturan bisnis.
* Pencatatan ledger pinjaman sebagai dasar akuntabilitas.
* Audit log bersifat *append-only* untuk kebutuhan pelacakan dan kepatuhan.

**Data Layer**

* **PostgreSQL** digunakan untuk data transaksi dan status yang membutuhkan konsistensi tinggi.
* **Object storage** digunakan untuk penyimpanan dokumen KYC dan foto pengguna, terpisah dari data operasional.

---

### 4.5 Usulan Tech Stack (Opsional)

Usulan teknologi berikut disesuaikan dengan prinsip MVP-first dan kemudahan pengembangan:

* **Mobile App**: Flutter
* **API Gateway / WAF**: Cloud-managed gateway
* **Authentication**:

  * MVP: Supabase Auth (managed)
  * Enterprise/regulated: Keycloak (self-hosted)
* **Backend Core**: Service-oriented backend dengan penerapan SOLID dan kesiapan CQRS
* **Database**: PostgreSQL
* **Object Storage**: S3-compatible (on-prem)
* **Notification**: Email provider, SMS gateway, FCM/APNs
* **Observability**: Centralized logging dan basic telemetry

Pemilihan teknologi ini bersifat usulan dan dapat disesuaikan dengan kebijakan organisasi serta regulasi yang berlaku.
