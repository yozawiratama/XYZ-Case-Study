## 3. Asumsi & Keputusan Desain

Dokumen ini disusun berdasarkan kebutuhan bisnis yang dijelaskan sebelumnya, dengan sejumlah asumsi dan keputusan desain yang ditetapkan secara eksplisit. Asumsi ini diperlukan untuk melengkapi detail yang tidak dijabarkan secara rinci pada kebutuhan awal, sekaligus menjaga konsistensi dan kelayakan desain solusi.

### 3.1 Asumsi Desain

Asumsi-asumsi berikut digunakan sebagai dasar perancangan sistem:

1. **Proses verifikasi identitas (KYC)**
   Proses unggah KTP dan foto pengguna diasumsikan sebagai bentuk verifikasi identitas dasar. Mekanisme verifikasi lanjutan seperti integrasi dengan layanan kependudukan atau *liveness detection* tidak termasuk dalam cakupan awal, namun dapat ditambahkan pada tahap pengembangan berikutnya.

2. **Satu pinjaman aktif per pengguna**
   Setiap pengguna hanya diperbolehkan memiliki satu pinjaman yang berstatus:

   * dalam proses penilaian, atau
   * aktif dan belum lunas.
     Aturan ini diterapkan secara konsisten pada lapisan aplikasi dan basis data untuk mencegah pelanggaran aturan bisnis.

3. **Notifikasi melalui email dan nomor telepon**
   Pengiriman notifikasi kepada pengguna diasumsikan menggunakan kanal email dan pesan singkat (SMS atau kanal serupa) melalui layanan pihak ketiga. Mekanisme retry dan pencatatan status pengiriman dianggap sebagai bagian dari desain sistem.

4. **Pembayaran cicilan di luar cakupan awal**
   Sistem dirancang untuk menampilkan informasi status pinjaman dan tagihan bulanan. Proses pembayaran cicilan secara langsung melalui aplikasi diasumsikan sebagai fitur lanjutan dan tidak dibahas secara detail dalam dokumen ini.

5. **Akses aplikasi melalui perangkat mobile**
   Aplikasi ditujukan untuk penggunaan pada perangkat mobile, dengan asumsi dukungan autentikasi biometrik mengikuti kemampuan perangkat masing-masing pengguna.

---

### 3.2 Keputusan Desain Utama

Berdasarkan asumsi tersebut, berikut adalah keputusan desain utama yang diambil:

1. **Pendekatan arsitektur berlapis (Layered Architecture)**
   Sistem dirancang dengan pemisahan yang jelas antara:

   * lapisan presentasi (mobile application),
   * layanan backend (API),
   * pengelolaan data dan integrasi layanan eksternal.
     Pendekatan ini dipilih untuk meningkatkan keterbacaan sistem, kemudahan pemeliharaan, dan fleksibilitas pengembangan.

2. **Validasi aturan bisnis di beberapa lapisan**
   Aturan penting seperti pembatasan satu pinjaman aktif diterapkan:

   * pada logika aplikasi backend, dan
   * sebagai constraint pada model data.
     Hal ini bertujuan untuk menjaga konsistensi data dan mencegah kondisi tidak valid akibat kegagalan di satu lapisan.

3. **Pengelolaan status pinjaman berbasis state**
   Pinjaman diperlakukan sebagai entitas dengan siklus hidup yang jelas, mulai dari pengajuan hingga penyelesaian. Setiap perubahan status dikontrol dan dicatat untuk mendukung auditabilitas dan pelacakan proses.

4. **Pemisahan data sensitif dan data operasional**
   Data identitas dan dokumen pengguna diperlakukan sebagai data sensitif, dengan pengelolaan dan akses yang dibatasi. Keputusan ini diambil untuk mendukung kebutuhan keamanan dan kepatuhan terhadap regulasi perlindungan data.

5. **Desain yang siap dikembangkan**
   Arsitektur dan model data dirancang agar dapat diperluas, misalnya untuk:

   * integrasi mesin penilaian risiko (*credit scoring*),
   * penambahan metode pembayaran,
   * atau peningkatan proses verifikasi identitas,
     tanpa memerlukan perubahan fundamental pada struktur sistem yang ada.
