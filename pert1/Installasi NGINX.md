### Modul: Instalasi & Konfigurasi Nginx (Versi Mainline) di Ubuntu 24.04

**1. Dependensi**
Sebelum mengambil Nginx dari sumber luar, Ubuntu membutuhkan alat bantu untuk mengunduh dan membaca sertifikat keamanan.

```bash
sudo apt update
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring -y

```

* `apt update`: Memperbarui daftar "katalog" aplikasi bawaan Ubuntu.
* `curl`: Perintah untuk mengunduh file dari internet melalui terminal.
* `gnupg2` & `ubuntu-keyring`: Aplikasi untuk memproses kunci keamanan (GPG Key).
* `ca-certificates`: Mengizinkan server membaca sertifikat HTTPS secara aman.
* `lsb-release`: Fitur untuk mendeteksi versi Ubuntu Anda secara otomatis (berguna di langkah 3).
* `-y`: Parameter otomatis untuk menekan tombol **Y** dan **Enter**. Jika tidak dipakai, instalasi akan terhenti untuk meminta konfirmasi Anda.

**2. Pemasangan Kunci Keamanan (GPG Key)**
Mencegah server menginstal file Nginx palsu yang disusupi virus.

```bash
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null

```

* `curl ...`: Mengunduh kunci digital (tanda tangan) resmi dari website Nginx.
* `|` (Simbol Pipa): Mengirim hasil unduhan ke perintah berikutnya.
* `gpg --dearmor`: Mengubah format kunci dari teks menjadi format biner yang bisa dibaca sistem Ubuntu.
* `tee ...`: Menyimpan kunci tersebut ke dalam folder brankas keamanan sistem (`/usr/share/keyrings/`).
* `>/dev/null`: Membuang teks laporan ke "tempat sampah" sistem agar layar terminal Anda tidak penuh dengan teks acak.

**3. Penambahan Repositori Resmi Nginx**
Menambahkan "toko aplikasi" Nginx resmi ke dalam Ubuntu.

```bash
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] http://nginx.org/packages/mainline/ubuntu $(lsb_release -cs) nginx" | sudo tee /etc/apt/sources.list.d/nginx.list

```

* `echo`: Mencetak teks.
* `[signed-by=...]`: Memerintahkan Ubuntu untuk memverifikasi toko ini menggunakan kunci yang kita pasang di Langkah 2.
* `$(lsb_release -cs)`: Kode ini akan otomatis mendeteksi OS Anda dan mengubah dirinya menjadi tulisan `noble` (nama sandi Ubuntu 24.04).
* `tee /etc/apt/sources.list.d/nginx.list`: Membuat file baru bernama `nginx.list` yang berisi alamat unduhan resmi tersebut.

**4. Mengubah Prioritas Repositori (Pinning)**
Saat ini Ubuntu memiliki dua toko yang menyediakan Nginx (toko bawaan OS vs toko resmi Nginx).

```bash
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" | sudo tee /etc/apt/preferences.d/99nginx

```

* **Kenapa perintah ini wajib?** Secara *default*, Ubuntu akan memilih aplikasinya sendiri (versi usang). Kode ini memaksa Ubuntu memberikan prioritas utama (Nilai 900) kepada file yang berasal dari `nginx.org`.
* Jika langkah ini dilewati, Anda akan gagal mendapatkan versi *Mainline* dan sistem akan menginstal versi lawas bawaan OS.

**5. Eksekusi Instalasi**
Menerapkan perubahan dan mulai menginstal.

```bash
sudo apt update
sudo apt install nginx -y

```

* `apt update` (Kedua): **Sangat krusial.** Perintah ini memaksa Ubuntu memindai ulang katalognya agar menyadari keberadaan toko Nginx yang baru saja dimasukkan di Langkah 3. Jika dilewati, Nginx gagal terinstal.
* `apt install nginx -y`: Mengunduh dan memasang aplikasi utamanya.

**6. Konfigurasi Firewall Keamanan (UFW)**
Analogi: Firewall adalah satpam. Jika dinyalakan tanpa aturan tertulis, satpam akan mengunci semua pintu dan Anda akan tertendang keluar dari server (SSH terputus).

```bash
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable

```

* `allow ssh`: **Wajib diketik pertama.** Membuka *port* 22. Jika ini terlewat dan UFW dinyalakan, Anda tidak akan bisa melakukan *remote* lewat Windows lagi (terkunci dari luar).
* `allow 80/tcp`: Membuka jalur HTTP untuk website (Web Sekolah & Kasir).
* **Kenapa tidak pakai perintah `allow 'Nginx Full'`?** Karena aplikasi yang diunduh langsung dari *nginx.org* tidak menyertakan profil bawaan Ubuntu, sehingga akan memicu *error* "Could not find a profile". Menggunakan angka *port* (80 & 443) secara manual adalah solusi paling akurat.
* `enable`: Menyalakan satpam (Firewall).
* *Tombol yang dipencet:* Akan muncul peringatan *"Command may disrupt existing ssh connections"*. Ketik tombol **y** di *keyboard*, lalu tekan **Enter**.



**7. Manajemen Layanan (Systemd)**

```bash
sudo systemctl enable nginx
sudo systemctl start nginx

```

* **Kenapa harus di-enable?** `enable` berfungsi memasang alarm otomatis. Jika suatu saat server *restart* (karena mati listrik atau *maintenance*), Nginx akan otomatis menyala sendiri (*autostart*). Jika tidak di-*enable*, pengunjung web akan melihat tampilan *error* sampai Anda menyalakannya secara manual.
* `start`: Menghidupkan mesin Nginx saat ini juga.

**8. Pengecekan Akhir**

```bash
nginx -v
sudo systemctl status nginx

```

* `nginx -v`: Mengecek versi. Pastikan *output* menunjukkan angka versi terbaru (misal: 1.25.x atau 1.27.x).
* `status nginx`: Memastikan mesin menyala tanpa gangguan.
* *Pemecahan Masalah:* Jika muncul tulisan merah *failed* atau *address already in use*, artinya *port* 80 sedang dipakai oleh aplikasi lain (biasanya Apache2). Anda harus mematikan Apache2 dengan perintah `sudo systemctl stop apache2 && sudo systemctl disable apache2`.
* *Tombol yang dipencet:* Perintah `status` terkadang membuat layar terminal tertahan untuk menampilkan log. Jika terminal tidak bisa diketik perintah baru, tekan tombol **q** pada *keyboard* untuk keluar dari mode laporan status.
