# Membangun DNS Server & Web Server Lokal di Ubuntu 24.04

## Tahap 1: Instalasi Nginx

Mengunduh mesin *web server* dan membuat folder fisik untuk menyimpan file *website*.

```bash
sudo apt update
sudo apt install nginx -y
sudo mkdir -p /var/www/sekolah
sudo mkdir -p /var/www/kasir

```

* **`apt update`**: Menyegarkan katalog aplikasi internal Ubuntu agar mengenali versi aplikasi terbaru.
* **`install nginx -y`**: Mengunduh dan memasang Nginx. Parameter `-y` (yes) memaksa instalasi berjalan otomatis tanpa meminta konfirmasi pengguna.
* **`mkdir -p`**: Membuat direktori (*make directory*). Parameter `-p` (parents) memungkinkan pembuatan folder bersarang sekaligus tanpa peringatan *error* jika folder sudah ada.

Selanjutnya, buat file halaman utama untuk masing-masing *website*:

```bash
sudo nano /var/www/sekolah/index.html
sudo nano /var/www/kasir/index.html

```

*(Isi dengan kode HTML sederhana. Simpan dengan Ctrl+O, Enter, lalu Ctrl+X).*

---

## Tahap 2: Konfigurasi Lalu Lintas Nginx

Sebelum membuat rute baru, wajib mematikan halaman *default* Ubuntu yang menguasai Port 80.

```bash
sudo rm -f /etc/nginx/sites-enabled/default

```

* **`rm -f`**: Menghapus file secara paksa (*force*). Ini membuang pintasan halaman bawaan dari folder aktif (etalase).

### Opsi A: Metode Sub-Path (Satu Port 80)

Mengakses web menggunakan jalur direktori, contoh: `[http://192.168.200.6/sekolah/](http://192.168.200.6/sekolah/)`

1. Buat file konfigurasi di folder gudang:
```bash
sudo nano /etc/nginx/sites-available/web

```


2. Isi file konfigurasi:
```nginx
server {
    listen 80;
    server_name _;

    location /sekolah {
        alias /var/www/sekolah/;
        index index.html;
        try_files $uri $uri/ /sekolah/index.html;
    }

    location /kasir {
        alias /var/www/kasir/;
        index index.html;
        try_files $uri $uri/ /kasir/index.html;
    }
}

```


* **`server_name _`**: Nginx akan merespons permintaan masuk tanpa mempedulikan nama domain apa yang diketik klien (opsi sapu jagat).
* **`alias`**: Mengarahkan URL yang diminta tepat ke folder fisik secara spesifik (berbeda dengan `root` yang menempelkan nama URL ke path folder).
* **`try_files`**: Menginstruksikan Nginx mencari file asli. Jika tidak ada, ia mengarahkan kembali ke `index.html` alih-alih melempar *Error 404 Not Found*.



### Opsi B: Metode Port Berbeda & Domain

Mengakses web menggunakan nama spesifik, contoh: `[http://kasir.local:8181](http://kasir.local:8181)`

1. Buat file terpisah untuk setiap web:
```bash
sudo nano /etc/nginx/sites-available/kasir

```


2. Isi file konfigurasi:
```nginx
server {
    listen 8181; 
    server_name kasir.local;
    root /var/www/kasir;
    index index.html;
}

```


* **`listen 8181`**: Membuka gerbang masuk khusus di Port 8181.
* **`server_name kasir.local`**: Mesin hanya akan menampilkan halaman ini jika pengunjung mengetik domain `kasir.local` di *browser*.



### Mengaktifkan Konfigurasi (Symlink)

Pindahkan file dari Gudang (`sites-available`) ke Etalase (`sites-enabled`) menggunakan jalan pintas (*symlink*).

```bash
sudo ln -s /etc/nginx/sites-available/web /etc/nginx/sites-enabled/

```

*(Ganti kata `web` dengan `kasir` atau `sekolah` jika menggunakan Opsi B).*

Terapkan perubahan ke sistem:

```bash
sudo nginx -t
sudo systemctl reload nginx

```

* **`nginx -t`**: Membaca ulang kode konfigurasi untuk mencari *typo* atau tanda titik koma (`;`) yang tertinggal.
* **`systemctl reload`**: Memuat ulang aturan baru secara halus tanpa memutuskan koneksi pengguna yang sedang membuka web.

---

## Tahap 3: Resolusi Konflik Port 53 DNS

Ubuntu 24.04 memiliki layanan bawaan (`systemd-resolved`) yang otomatis membajak Port 53. Ini wajib dimatikan agar CoreDNS tidak mengalami *error "bind: address already in use"*.

1. Edit konfigurasi sistem jaringan Ubuntu:
```bash
sudo nano /etc/systemd/resolved.conf

```


2. Cari baris `#DNSStubListener=yes`. Hapus tanda pagar dan ubah menjadi `no`:
```ini
DNSStubListener=no

```


* **`DNSStubListener=no`**: Melarang OS Ubuntu menyediakan layanan DNS bawaan secara lokal, sehingga Port 53 terbebas sepenuhnya.


3. Terapkan konfigurasi:
```bash
sudo systemctl restart systemd-resolved

```



---

## Tahap 4: Instalasi CoreDNS

Membuat "buku telepon" lokal yang mengubah IP angka menjadi nama domain (misal: `192.168.200.6` menjadi `kasir.local`).

1. Unduh dan pindahkan aplikasi:
```bash
wget https://github.com/coredns/coredns/releases/download/v1.11.1/coredns_1.11.1_linux_amd64.tgz
tar -zxvf coredns_1.11.1_linux_amd64.tgz
sudo mv coredns /usr/local/bin/
sudo mkdir -p /etc/coredns

```


2. Buat file konfigurasi DNS:
```bash
sudo nano /etc/coredns/Corefile

```


3. Isi dengan skrip berikut (sesuaikan IP dengan IP server):
```text
sekolah.local kasir.local {
    hosts {
        192.168.200.6 sekolah.local kasir.local
        fallthrough
    }
    forward . 8.8.8.8
    log
    errors
}

```


* **`hosts`**: Mendiktekan pemetaan manual (pemaksaan arah domain ke IP `192.168.200.6`).
* **`fallthrough`**: Jika CoreDNS tidak mengenali permintaan di area lokal, permintaan tidak langsung ditolak, melainkan dilanjutkan ke proses berikutnya.
* **`forward . 8.8.8.8`**: Melempar semua permintaan domain non-lokal (seperti `google.com`) ke DNS Google, memastikan klien tetap bisa mengakses internet luar.


4. Buat layanan agar aktif otomatis:
```bash
sudo nano /etc/systemd/system/coredns.service

```


```ini
[Unit]
Description=CoreDNS DNS Server
After=network.target

[Service]
User=root
ExecStart=/usr/local/bin/coredns -conf /etc/coredns/Corefile
Restart=always

[Install]
WantedBy=multi-user.target

```


* **`ExecStart`**: Perintah eksekusi utama yang menjalankan *binary* CoreDNS dan menunjuk ke file `Corefile` yang baru saja dibuat.
* **`Restart=always`**: Memaksa CoreDNS untuk hidup kembali secara instan jika tiba-tiba mengalami *crash*.


5. Aktifkan layanan:
```bash
sudo systemctl daemon-reload
sudo systemctl enable coredns
sudo systemctl start coredns

```



---

## Tahap 5: Keamanan Jaringan (Firewall UFW) - Solusi Whitelist

Mencegah OS Ubuntu memblokir lalu lintas jaringan klien Windows. Menggunakan izin *port* individu terkadang menyebabkan klien mendapat *error "Request timed out"* karena blokir paket Ping (ICMP) dan hilangnya status UDP DNS.

Solusi paling stabil untuk praktik lokal adalah mendaftarkan keseluruhan segmen jaringan (*Whitelist Subnet*).

```bash
sudo ufw allow 22/tcp
sudo ufw allow from 192.168.200.0/24
sudo ufw enable

```

* **`allow 22/tcp`**: Wajib dieksekusi agar sesi remote terminal SSH tidak terkunci dari luar.
* **`allow from 192.168.200.0/24`**: Memberikan izin "VIP" (*Whitelist*). Semua perangkat (*laptop, smartphone*) yang memiliki IP berawalan `192.168.200.x` bebas mengirimkan perintah `ping`, mengakses DNS, dan membuka website Nginx ke server ini tanpa hambatan. Perangkat dari segmen jaringan lain (misal: `192.168.1.x`) tetap diblokir.
* **`enable`**: Menyalakan satpam jaringan UFW (Ketik `y` jika muncul peringatan interupsi SSH).

---

## Tahap 6: Pengaturan Klien (Windows) & Analisis Kendala

**Langkah Klien:** Buka IPv4 Properties pada adapter Wi-Fi/Ethernet Windows. Ubah `Preferred DNS server` menjadi IP Server Ubuntu (`192.168.200.6`). Lalu bersihkan sisa *cache error* di CMD dengan perintah `ipconfig /flushdns`.

**Tabel Pemecahan Masalah:**

| Kendala (Pesan Error) | Akar Masalah | Solusi |
| --- | --- | --- |
| **`DNS request timed out`** (Ping RTO di CMD Windows) | Putus komunikasi fisik atau Adapter Jaringan salah konfigurasi. | Pada VirtualBox, pastikan Network Adapter disetel ke **Bridged Adapter** dan memilih WiFi/LAN yang terhubung langsung ke PC Anda. |
| **`DNS_PROBE_POSSIBLE`** | Windows gagal menghubungi CoreDNS. | Cek status `sudo systemctl status coredns`. Jika merah, cek *typo* di `Corefile`. |
| **`connection refused`** (Saat tes `nslookup` di Ubuntu) | Layanan `systemd-resolved` bentrok dan membajak Port 53. | Lakukan Tahap 3 (Ubah konfigurasi `DNSStubListener=no`). |
| **Web 404 Not Found** (Pada metode Sub-path) | Penulisan URL di browser tidak menggunakan penutup *slash* (`/`). | Ketik secara lengkap: `[http://192.168.200.6/kasir/](http://192.168.200.6/kasir/)` |
| **`conflicting server name _`** | File *default* bawaan Ubuntu belum dihapus. | Jalankan `sudo rm -f /etc/nginx/sites-enabled/default`, lalu `sudo systemctl reload nginx`. |
