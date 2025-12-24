## 10. Future Considerations

Bagian ini menjelaskan potensi pengembangan lanjutan sistem seiring dengan pertumbuhan pengguna, kompleksitas bisnis, dan tuntutan operasional. Seluruh poin di bawah ini **sengaja tidak diimplementasikan pada tahap MVP**, namun telah dipertimbangkan sejak awal desain agar sistem dapat berevolusi tanpa perubahan fundamental.

### 10.1 Pemisahan Read Model dan Write Model (CQRS)

Pada tahap MVP, sistem masih menggunakan satu model data dengan **selective denormalization** untuk kebutuhan baca cepat.
Seiring meningkatnya trafik dan kompleksitas query, sistem dapat ditingkatkan dengan:

* pemisahan *read model* dan *write model*,
* penggunaan materialized view atau projection table,
* optimalisasi query dashboard tanpa membebani database transaksional.

Pendekatan ini memungkinkan peningkatan performa baca dan skalabilitas, tanpa mengubah aturan bisnis inti.

---

### 10.2 Event-Driven Workflow untuk Lifecycle Pinjaman

Saat ini, perubahan status pinjaman diproses secara sinkron untuk menjaga kesederhanaan dan keandalan MVP.
Sebagai pengembangan lanjutan, sistem dapat diperluas dengan pendekatan **event-driven**, khususnya untuk:

* perubahan status pinjaman (approved, active, paid off),
* pemicu notifikasi,
* integrasi dengan sistem eksternal.

Pendekatan ini meningkatkan fleksibilitas dan observability, namun baru relevan ketika kebutuhan integrasi dan volume transaksi meningkat.

---

### 10.3 Realtime Update melalui Socket / Subscription

Untuk meningkatkan pengalaman pengguna dan mengurangi kebutuhan polling data:

* sistem dapat mendukung **realtime update** berbasis WebSocket atau mekanisme subscription,
* perubahan status pinjaman atau notifikasi dapat langsung tercermin di aplikasi tanpa perlu refresh manual.

Realtime digunakan **hanya untuk update tampilan (read/update)**, bukan untuk pengambilan keputusan bisnis.
Sumber kebenaran tetap berada pada backend dan database inti.

---

### 10.4 Redis untuk Performa, Concurrency, dan Proteksi

Redis dapat diperkenalkan sebagai komponen pendukung ketika beban sistem meningkat, dengan fungsi utama:

* caching data yang bersifat read-heavy (dashboard, summary pinjaman),
* distributed locking untuk mencegah race condition (misalnya double submit pengajuan),
* rate limiting dan proteksi abuse,
* penyimpanan idempotency key untuk operasi kritis.

Redis diposisikan sebagai **performance and safety enhancer**, bukan sebagai komponen wajib pada tahap awal.

---

### 10.5 Sistem Notifikasi dengan Queue dan Retry

Pada tahap MVP, pengiriman email dan pesan dapat dilakukan secara sinkron.
Sebagai pengembangan lanjutan, sistem notifikasi dapat ditingkatkan dengan:

* message queue untuk pengiriman email dan SMS,
* mekanisme retry dengan backoff,
* dead-letter queue untuk kegagalan permanen,
* idempotency untuk mencegah pengiriman ganda.

Pendekatan ini meningkatkan keandalan notifikasi, terutama untuk pesan yang bersifat kritikal seperti persetujuan pinjaman.

---

### 10.6 Credit Scoring dan Fraud Detection

Untuk mitigasi risiko yang lebih baik, sistem dapat diperluas dengan:

* credit scoring engine berbasis rule atau integrasi pihak ketiga,
* deteksi anomali dan fraud (misalnya pola pengajuan tidak wajar, multiple account per device).

Fitur ini relevan ketika skala bisnis meningkat dan data historis sudah mencukupi untuk analisis risiko.

---

### 10.7 Compliance Lanjutan dan Tata Kelola Data

Seiring bertambahnya kewajiban regulasi, sistem dapat mendukung:

* kebijakan retensi data,
* anonymization atau pseudonymization data pengguna,
* mekanisme *right to erasure* sesuai regulasi perlindungan data pribadi,
* penguatan audit log (immutable log, signed events).

Hal ini memperkuat kesiapan sistem terhadap audit regulator dan pemeriksaan eksternal.

---

### 10.8 Observability, Monitoring, dan Operasional

Untuk mendukung operasi skala besar, sistem dapat dilengkapi dengan:

* monitoring performa dan error rate,
* alerting untuk kondisi anomali (lonjakan error, approval rate tidak normal),
* dashboard operasional bagi tim internal.

Fokus utama adalah menjaga stabilitas sistem dan respons cepat terhadap insiden.

---

### 10.9 Strategi Biaya dan Vendor

Karena sistem memanfaatkan layanan terkelola seperti Supabase, pengembangan lanjutan perlu mempertimbangkan:

* optimasi penggunaan fitur Supabase (RLS, direct query),
* visibilitas dan kontrol biaya,
* opsi migrasi atau decoupling vendor secara bertahap jika diperlukan.

Strategi ini memastikan sistem tetap fleksibel dan tidak terjebak pada satu vendor tertentu.
