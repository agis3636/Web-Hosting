# Instalasi & Konfigurasi Nginx (Versi Mainline) di Ubuntu 24.04 dan juga DNS CoreDNS

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

---

---

---


# menggabungkan seluruh metode penayangan dua website (`sekolah` dan `kasir`) di satu server Ubuntu menggunakan Nginx.

---

# Hosting Multi-Website Nginx di Ubuntu

#### 1. Perbandingan Metode Akses

Jika membandingkan dua pendekatan penayangan website dalam satu server, perbedaannya dirangkum dalam tabel berikut:

| Metode | Jalur / Port | Contoh Alamat Akses | Karakteristik |
| --- | --- | --- | --- |
| **Metode 1: Sub-path** | Port 80 (Standar) | `http://192.168.80.18/sekolah` `http://192.168.80.18/kasir` | Menggunakan satu port utama (80) dan memisahkan folder melalui ekstensi direktori URL. |
| **Metode 2: Port Berbeda** | Port 8080 & 8181 | `http://192.168.80.18:8080` `http://192.168.80.18:8181` | Memisahkan jalur akses menggunakan nomor *port* yang berbeda untuk setiap aplikasi. |

---

#### 2. Langkah 1: Membuat Direktori Penyimpanan Website

Setiap website membutuhkan folder fisik terpisah di dalam server untuk menyimpan file kodingannya.

* **Perintah Terminal:**
```bash
sudo mkdir -p /var/www/sekolah
sudo mkdir -p /var/www/kasir

```


* **Penjelasan & Alasan:**
* `sudo`: Menjalankan perintah dengan hak akses administrator tertinggi.
* `mkdir`: Perintah dasar Linux untuk membuat folder baru (*make directory*).
* `-p`: Parameter pengaman agar Linux membuatkan folder utama sekaligus sub-folder di dalamnya secara otomatis tanpa *error* jika sudah ada.
* `/var/www/`: Direktori standar di Linux untuk menyimpan file penayangan web server.



---

#### 3. Langkah 2: Mengisi File Konten Website (`index.html`)

Masukkan kode program HTML ke dalam masing-masing folder.

* **Perintah Terminal:**
```bash
sudo nano /var/www/sekolah/index.html
sudo nano /var/www/kasir/index.html

```


* **Penjelasan & Alasan:**
* `nano`: Editor teks bawaan terminal untuk menulis atau menempel kodingan HTML.
* Nama file wajib `index.html` agar dibaca otomatis oleh Nginx sebagai halaman utama.
* *Cara simpan & keluar:* Tekan **Ctrl + O**, lalu **Enter**, kemudian **Ctrl + X**.



---

#### 4. Langkah 3: Konfigurasi Nginx (Pilih Salah Satu Metode)

Hapus file konfigurasi bawaan agar tidak terjadi bentrok (*port conflict*):

```bash
sudo rm -f /etc/nginx/conf.d/default.conf

```

* **Opsi A: Konfigurasi Metode Sub-path (Satu Port 80)**
Buat file konfigurasi utama:
```bash
sudo nano /etc/nginx/conf.d/web.conf

```


*Isi kodingan:*
```nginx
server {
    listen 80;
    server_name _;

    location /sekolah {
        alias /var/www/sekolah/;
        index index.html;
        try_files $uri $uri/ =404;
    }

    location /kasir {
        alias /var/www/kasir/;
        index index.html;
        try_files $uri $uri/ =404;
    }
}

```


* **Opsi B: Konfigurasi Metode Port Berbeda (8080 & 8181)**
Buat file konfigurasi web sekolah (`sekolah.conf`):
```bash
sudo nano /etc/nginx/conf.d/sekolah.conf

```


*Isi kodingan:*
```nginx
server {
    listen 8080;
    server_name _;
    root /var/www/sekolah;
    index index.html;
}

```


Buat file konfigurasi web kasir (`kasir.conf`):
```bash
sudo nano /etc/nginx/conf.d/kasir.conf

```


*Isi kodingan:*
```nginx
server {
    listen 8181;
    server_name _;
    root /var/www/kasir;
    index index.html;
}

```



---

#### 5. Langkah 4: Pengaturan Firewall (UFW)

Jika Anda menggunakan **Metode 2 (Port Berbeda)**, Anda wajib membuka *port* 8080 dan 8181 di firewall agar tidak terblokir. (Jika menggunakan Metode 1, cukup pastikan port 80 terbuka).

* **Perintah Terminal:**
```bash
sudo ufw allow 80/tcp
sudo ufw allow 8080/tcp
sudo ufw allow 8181/tcp
sudo ufw reload

```


* **Penjelasan & Alasan:**
* `ufw allow .../tcp`: Membuka jalur komunikasi masuk pada *port* spesifik. Jika dilewati, browser akan memunculkan *error* `ERR_CONNECTION_TIMED_OUT`.
* `ufw reload`: Menyegarkan aturan firewall tanpa mematikan sistem proteksi.



---

#### 6. Langkah 5: Verifikasi dan Penerapan Sistem

Uji integritas sintaks Nginx dan terapkan perubahan tanpa mematikan layanan.

* **Perintah Terminal:**
```bash
sudo nginx -t
sudo systemctl reload nginx

```


* **Penjelasan & Alasan:**
* `nginx -t`: Memeriksa apakah ada kesalahan pengetikan sintaks pada file konfigurasi. Pastikan bernilai *successful*.
* `systemctl reload nginx`: Menerapkan pembaruan konfigurasi secara halus.


---
---
---

# Membangun DNS Server Lokal dengan CoreDNS & Nginx di Ubuntu 24.04

## 1. Pengaturan IP Statis & Firewall Server

Server DNS wajib memiliki IP tetap agar klien tidak terputus saat *router* di-restart.

**Langkah:**
Buka file Netplan (contoh: `sudo nano /etc/netplan/50-cloud-init.yaml`), masukkan konfigurasi berikut:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses: [192.168.2.50/24]
      routes:
        - to: default
          via: 192.168.2.1
      nameservers:
        addresses: [8.8.8.8]

```

* **Penjelasan Sintaks:**
* `dhcp4: false`: Mematikan permintaan IP otomatis.
* `addresses`: Mengunci IP server di `192.168.2.50`.



Terapkan jaringan: `sudo netplan apply`

**Langkah Firewall (UFW):**
Buka gerbang lalu lintas agar klien Windows bisa mengakses server.

```bash
sudo ufw allow 22/tcp
sudo ufw allow 53
sudo ufw allow 80/tcp
sudo ufw allow 8080/tcp
sudo ufw enable

```

* **Penjelasan Sintaks:** Port `53` adalah jalur utama DNS CoreDNS. Port `80`/`8080` untuk web Nginx. Port `22` wajib dibuka agar koneksi SSH tidak terputus saat UFW diaktifkan.

## 2. Instalasi dan Konfigurasi CoreDNS

CoreDNS bertugas menerjemahkan nama domain (`.local`) menjadi IP server.

1. **Unduh dan Pasang:**
```bash
wget https://github.com/coredns/coredns/releases/download/v1.11.1/coredns_1.11.1_linux_amd64.tgz
tar -zxvf coredns_1.11.1_linux_amd64.tgz
sudo mv coredns /usr/local/bin/

```


2. **Konfigurasi File DNS:**
Buat folder dan file `sudo nano /etc/coredns/Corefile`, isi dengan:
```text
sekolah.local kasir.local {
    hosts {
        192.168.2.50 sekolah.local kasir.local
        fallthrough
    }
    forward . 8.8.8.8
    log
    errors
}

```


* **Penjelasan Sintaks:** `hosts` memaksa domain dialihkan ke IP `192.168.2.50`. `forward . 8.8.8.8` memastikan jika klien mencari domain internet (seperti google.com), CoreDNS akan melempar permintaannya ke DNS Google.


3. **Jalankan sebagai Layanan (Service):**
Buat `sudo nano /etc/systemd/system/coredns.service`, salin skrip `[Unit]` dan `[Service]` standar, lalu jalankan `sudo systemctl daemon-reload` dan `sudo systemctl start coredns`.

## 3. Konfigurasi Web Server (Nginx)

Membuat domain yang sudah diterjemahkan CoreDNS menampilkan halaman web yang benar.

Buka konfigurasi Nginx (contoh: `sudo nano /etc/nginx/conf.d/sekolah.conf`):

```nginx
server {
    listen 80;
    server_name sekolah.local;
    root /var/www/sekolah;
    index index.html;
}

```

* **Penjelasan Sintaks:**
* `listen 80;`: Jika menggunakan port 80, domain bisa diakses tanpa menuliskan angka port di browser.
* `server_name`: Nginx akan mencocokkan URL yang diketik klien dengan nama domain di sini.


* **Terapkan:** `sudo nginx -t` (untuk mengecek salah ketik), lalu `sudo systemctl reload nginx`.

## 4. Klien Windows & Analisis Masalah

Di laptop Windows (Host), buka **IPv4 Properties** pada adapter Wi-Fi/Ethernet Anda. Atur:

1. **Preferred DNS server:** `192.168.2.50` (IP Ubuntu).
2. **Alternate DNS server:** `8.8.8.8` (DNS Google).
3. Buka CMD, ketik `ipconfig /flushdns`.

| Error / Kendala | Penyebab Utama | Solusi Bertahap |
| --- | --- | --- |
| **DNS_PROBE_POSSIBLE** (Ping domain gagal) | Port 53 diblokir UFW, atau urutan DNS Windows terbalik. | 1. Di Ubuntu: `sudo ufw allow 53`. 2. Di Windows: Pastikan IP Ubuntu berada di **Preferred**, bukan Alternate. |
| **Web Tidak Tampil** (Ping domain sukses) | Konfigurasi *port* Nginx belum sesuai URL, atau *cache* peramban. | 1. Jika Nginx memakai port `8080`, akses wajib pakai URL `[http://sekolah.local:8080](http://sekolah.local:8080)`. 2. Gunakan mode *Incognito* peramban. |
| **IP Ubuntu Nyangkut** (Beda segmen jaringan) | VirtualBox *Bridged Adapter* gagal memperbarui IP saat Host pindah Wi-Fi. | 1. Set IP Statis baru di Netplan sesuai segmen Wi-Fi baru. 2. Sesuaikan IP tersebut di `/etc/coredns/Corefile`. |
