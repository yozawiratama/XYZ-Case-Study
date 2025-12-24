## 2. Pernyataan Masalah & Kebutuhan Bisnis

PT XYZ berencana mengembangkan aplikasi mobile pinjaman online untuk memperluas jangkauan layanan pembiayaan kepada pengguna akhir. Tantangan utama yang ingin diselesaikan melalui aplikasi ini adalah bagaimana menyediakan proses pinjaman yang **mudah diakses oleh pengguna**, namun tetap **terkendali, aman, dan sesuai dengan aturan bisnis yang ditetapkan perusahaan**.

Dari sisi bisnis, aplikasi diharapkan mampu menangani seluruh proses pinjaman secara digital, mulai dari onboarding pengguna hingga pemantauan status pinjaman yang sedang berjalan. Proses tersebut harus dirancang dengan alur yang jelas, transparan bagi pengguna, serta memberikan kontrol yang memadai bagi sistem dalam mengelola risiko pinjaman.

### 2.1 Permasalahan Utama

Permasalahan utama yang menjadi fokus dalam perancangan solusi ini meliputi:

1. **Digital onboarding dan verifikasi pengguna**
   Sistem harus mampu mengelola proses registrasi pengguna secara mandiri, termasuk pengumpulan data dasar, unggah identitas (KTP), dan foto pengguna sebagai bagian dari proses verifikasi awal (*Know Your Customer / KYC*).

2. **Pengelolaan pengajuan pinjaman yang terkontrol**
   Aplikasi harus memastikan bahwa setiap pengguna hanya dapat memiliki **satu pinjaman aktif atau dalam proses** pada satu waktu, untuk mencegah risiko pengajuan ganda dan over-lending.

3. **Transparansi status pinjaman dan tagihan**
   Pengguna perlu dapat melihat status pengajuan pinjaman, status pinjaman yang sedang berjalan, serta rincian tagihan bulanan secara jelas melalui aplikasi.

4. **Penyampaian hasil keputusan pinjaman secara andal**
   Sistem harus dapat menginformasikan hasil pengajuan pinjaman (diterima atau ditolak) kepada pengguna melalui kanal komunikasi yang tersedia, seperti email dan nomor telepon.

### 2.2 Kebutuhan Fungsional

Berdasarkan permasalahan tersebut, kebutuhan fungsional utama sistem meliputi:

* Registrasi pengguna menggunakan data pribadi, email, dan nomor telepon.
* Unggah foto identitas (KTP) dan foto pengguna sebagai bagian dari proses verifikasi.
* Autentikasi pengguna menggunakan password dan dukungan biometrik pada perangkat yang mendukung.
* Pengajuan pinjaman dengan ketentuan:

  * Nilai pinjaman maksimum Rp 12.000.000.
  * Tenor pinjaman maksimum 12 bulan.
* Proses penilaian pengajuan pinjaman dengan hasil akhir berupa persetujuan atau penolakan.
* Pembatasan pengajuan pinjaman baru apabila pengguna masih memiliki pinjaman dalam proses atau belum lunas.
* Informasi status pinjaman dan tagihan bulanan yang dapat diakses oleh pengguna melalui aplikasi.
* Pengiriman notifikasi hasil pengajuan pinjaman melalui email dan nomor telepon pengguna.

### 2.3 Kebutuhan Non-Fungsional (Ringkas)

Selain kebutuhan fungsional, sistem juga harus memperhatikan kebutuhan non-fungsional berikut:

* **Keamanan data**: Perlindungan data pribadi dan dokumen identitas pengguna.
* **Keandalan sistem**: Proses pengajuan dan notifikasi harus berjalan konsisten dan dapat diandalkan.
* **Skalabilitas**: Arsitektur sistem harus memungkinkan pengembangan dan penambahan fitur di masa depan tanpa perubahan besar pada fondasi sistem.
* **Auditabilitas**: Aktivitas penting seperti registrasi, pengajuan pinjaman, dan perubahan status harus dapat ditelusuri.
