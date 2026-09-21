# APA BEDANYA NGINX MAINLINE DENGAN NGINX BAWAAN UBUNTU?

Bedanya murni ada di **sumber tempat kalian mengunduh (repositori)**. Meskipun aplikasinya sama-sama Nginx, pembuat paket instalasinya berbeda, sehingga aturan main foldernya juga dibikin beda.

Berikut rincian perbedaannya:

**1. Nginx (Versi Mainline / Resmi)**

* **Sumber:** Diunduh langsung dari website resmi pembuatnya (`nginx.org`).
* **Struktur Folder:** Hanya menggunakan satu folder, yaitu `conf.d/`.
* **Analogi:** Seperti meja kerja. Apa pun file konfigurasi yang di taruh di atas meja `conf.d/`, mesin Nginx akan langsung membacanya dan menjalankannya secara otomatis. Gayanya simpel dan langsung ke sasaran (versi *vanilla*).

**2. Nginx Punya Mahasiswa (Versi Bawaan Ubuntu/OS)**

* **Sumber:** Diunduh dari "toko aplikasi" bawaan OS Ubuntu itu sendiri (hanya pakai perintah `apt install nginx` tanpa menambahkan kunci GPG resmi).
* **Struktur Folder:** Ubuntu merombak Nginx dengan menambahkan sistem dua folder: `sites-available` dan `sites-enabled`.
* **Analogi:** Seperti toko ritel. Mereka punya **Gudang** (`sites-available`) untuk menyimpan semua file konfigurasi. Tapi, konfigurasi itu tidak akan jalan sebelum barangnya dipajang ke **Etalase** (`sites-enabled`) menggunakan *symlink*.

**Kenapa Ubuntu repot-repot membuat gaya "Gudang dan Etalase"?**
Tujuannya untuk keamanan dan manajemen. Kalau server punya 50 website dan admin ingin mematikan 1 website sementara, dia tidak perlu menghapus file kodingannya. Dia cukup membuang pintasannya dari "Etalase" (`sites-enabled`), sedangkan file aslinya tetap aman tersimpan di "Gudang" (`sites-available`).
