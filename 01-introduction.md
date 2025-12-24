## 1. Ringkasan Eksekutif

Dokumen ini menyajikan rancangan solusi (*solution architecture & design*) untuk pengembangan **aplikasi mobile pinjaman online** milik PT XYZ. Aplikasi ini bertujuan untuk menyediakan layanan pinjaman yang mudah diakses, aman, dan terkontrol, mulai dari proses registrasi pengguna, pengajuan pinjaman, hingga pemantauan status hutang dan tagihan.

Fokus utama dari solusi ini adalah merancang sistem yang mampu menangani **siklus pinjaman secara end-to-end**, dengan tetap memperhatikan aturan bisnis utama, yaitu **pembatasan satu pinjaman aktif atau dalam proses untuk setiap pengguna**. Aturan ini menjadi fondasi penting dalam desain arsitektur, model data, alur aplikasi, serta validasi pada API.

Solusi yang diusulkan mengadopsi pendekatan arsitektur berlapis (*layered architecture*) yang memisahkan tanggung jawab antara antarmuka pengguna (mobile application), layanan backend, pengelolaan data, serta integrasi dengan layanan eksternal seperti notifikasi email dan pesan singkat. Pendekatan ini dipilih untuk memastikan sistem mudah dikembangkan, dipelihara, dan diskalakan di masa depan.

Dokumen ini mencakup:

* Pemahaman kebutuhan bisnis dan aturan utama sistem.
* Asumsi desain yang digunakan sebagai dasar perancangan.
* Arsitektur sistem tingkat tinggi.
* Model domain dan rancangan basis data (ERD).
* Siklus hidup pinjaman dan pengelolaan status.
* Alur layar aplikasi beserta perilaku utama pengguna.
* Desain API inti yang mendukung proses bisnis.
* Pertimbangan keamanan, kepatuhan, dan pengembangan lanjutan.

Seluruh rancangan disusun untuk memberikan gambaran yang jelas dan terstruktur mengenai solusi yang diusulkan, sehingga dapat digunakan sebagai referensi bersama oleh pemangku kepentingan bisnis, tim teknis, maupun pihak lain yang terlibat dalam pengembangan aplikasi ini.

