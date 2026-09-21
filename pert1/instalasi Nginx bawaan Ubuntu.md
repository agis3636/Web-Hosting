# Modul: Server DNS & Multi-Website Nginx (Bawaan Ubuntu 24.04)

Modul ini memandu instalasi Nginx menggunakan repositori resmi bawaan OS Ubuntu, dikombinasikan dengan CoreDNS sebagai DNS server lokal. Sistem konfigurasi Nginx Ubuntu menggunakan pemisahan direktori `sites-available` (gudang) dan `sites-enabled` (aktif).

## 1. Instalasi Web Server & Pembuatan Folder

Menginstal Nginx dari sumber bawaan OS dan menyiapkan ruang penyimpanan website.

```bash
sudo apt update
sudo apt install nginx -y
sudo mkdir -p /var/www/sekolah
sudo mkdir -p /var/www/kasir

```

* **`apt update`**: Memperbarui daftar "katalog" aplikasi sistem Ubuntu.
* **`install nginx -y`**: Mengunduh Nginx. Parameter `-y` menekan tombol Y otomatis agar instalasi tidak terhenti.
* **`mkdir -p`**: Membuat folder bersarang tanpa menghasilkan *error* jika folder utama sudah ada.

Selanjutnya, isi masing-masing folder dengan halaman web:

```bash
sudo nano /var/www/sekolah/index.html
sudo nano /var/www/kasir/index.html

```

## 2. Konfigurasi Nginx (Sistem Symlink)

Pilih salah satu metode penayangan di bawah ini. Sebelum memulai, **wajib** menghapus halaman *default* Ubuntu agar tidak terjadi bentrok di Port 80.

```bash
sudo rm -f /etc/nginx/sites-enabled/default

```

### Opsi A: Metode Sub-Path (Satu Port 80)

Mengakses web menggunakan jalur folder, contoh: `[http://192.168.80.18/sekolah/](http://192.168.80.18/sekolah/)`

1. Buat file konfigurasi di gudang `sites-available`:
```bash
sudo nano /etc/nginx/sites-available/web

```


2. Isi dengan konfigurasi berikut:
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


* **`alias`**: Mengarahkan URL spesifik ke folder fisik di dalam server.
* **`try_files`**: Mengamankan struktur URL agar mencegah *Error 404 Not Found*.


3. Aktifkan konfigurasi dengan membuat *Symlink* (pintasan) ke `sites-enabled`:
```bash
sudo ln -s /etc/nginx/sites-available/web /etc/nginx/sites-enabled/

```



### Opsi B: Metode Port Berbeda & DNS (Domain `.local`)

Mengakses web menggunakan domain buatan (CoreDNS), contoh: `[http://sekolah.local:8080](http://sekolah.local:8080)`

1. Buat file konfigurasi terpisah:
```bash
sudo nano /etc/nginx/sites-available/sekolah
sudo nano /etc/nginx/sites-available/kasir

```


2. Isi konfigurasi (Ubah port & root sesuai nama web):
```nginx
server {
    listen 8080; 
    server_name sekolah.local;
    root /var/www/sekolah;
    index index.html;
}

```


* **`listen`**: Menentukan gerbang masuk port (Gunakan 8181 untuk file `kasir`).
* **`server_name`**: Mendengarkan permintaan akses yang cocok dengan nama domain lokal.


3. Aktifkan kedua konfigurasi:
```bash
sudo ln -s /etc/nginx/sites-available/sekolah /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/kasir /etc/nginx/sites-enabled/

```



**Terapkan Perubahan Nginx (Berlaku untuk Opsi A & B):**

```bash
sudo nginx -t
sudo systemctl reload nginx

```

* **`nginx -t`**: Menguji apakah ada salah ketik/kurang titik koma (`;`) pada sintaks.
* **`systemctl reload`**: Memperbarui sistem tanpa memutus koneksi klien yang sedang aktif.

## 3. Instalasi & Konfigurasi CoreDNS

Berfungsi sebagai buku telepon lokal agar IP server dapat diakses dengan nama domain.

```bash
wget https://github.com/coredns/coredns/releases/download/v1.11.1/coredns_1.11.1_linux_amd64.tgz
tar -zxvf coredns_1.11.1_linux_amd64.tgz
sudo mv coredns /usr/local/bin/
sudo mkdir -p /etc/coredns
sudo nano /etc/coredns/Corefile

```

Isi konfigurasi (Sesuaikan IP `192.168.80.18` dengan IP Ubuntu Anda):

```text
sekolah.local kasir.local {
    hosts {
        192.168.80.18 sekolah.local kasir.local
        fallthrough
    }
    forward . 8.8.8.8
    log
    errors
}

```

* **`hosts`**: Memaksa permintaan nama domain `.local` menuju IP spesifik.
* **`forward`**: Meneruskan pencarian domain internet umum (contoh: google.com) ke DNS Google (8.8.8.8) agar server tetap terkoneksi ke internet luar.

Jalankan sebagai layanan latar belakang:

```bash
sudo nano /etc/systemd/system/coredns.service

```

Tempelkan skrip otomatisasi:

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

Aktifkan CoreDNS:

```bash
sudo systemctl daemon-reload
sudo systemctl enable coredns
sudo systemctl start coredns

```

## 4. Konfigurasi Keamanan (Firewall / UFW)

Membuka jalur lalu lintas agar tidak diblokir oleh OS Ubuntu.

```bash
sudo ufw allow 22/tcp
sudo ufw allow 53
sudo ufw allow 80/tcp
sudo ufw allow 8080/tcp
sudo ufw allow 8181/tcp
sudo ufw enable

```

* **`22/tcp`**: Wajib dieksekusi pertama agar remote SSH dari laptop tidak terputus.
* **`53`**: Port wajib untuk fungsi CoreDNS (TCP/UDP).
* **`ufw enable`**: Mengaktifkan sistem proteksi secara keseluruhan. Ketik `y` jika muncul peringatan.

## 5. Pemecahan Masalah (Troubleshooting)

| Gejala Error | Penyebab Utama | Solusi & Tindakan |
| --- | --- | --- |
| **Peringatan: `conflicting server name**` | Ada dua konfigurasi `server_name` yang berjalan di port yang sama secara bersamaan. | Cek folder `sites-enabled`. Hapus file konfigurasi ganda menggunakan perintah `sudo rm -f`. |
| **Web muncul "404 Not Found"** (Metode Sub-path) | URL diakses tanpa diakhiri garis miring `/`, atau struktur `try_files` salah. | Wajib mengetik URL dengan format: `http://IP_Server/kasir/`. |
| **Browser: `DNS_PROBE_POSSIBLE**` | Resolusi CoreDNS diblokir Firewall atau urutan DNS Windows terbalik. | Buka port 53 di Ubuntu (`sudo ufw allow 53`). Di Windows, jadikan IP Ubuntu sebagai **Preferred DNS**. |
| **Browser: Web Ubuntu Default Muncul** | Konfigurasi awal bawaan OS memonopoli lalu lintas port 80. | Hapus *symlink default*: `sudo rm -f /etc/nginx/sites-enabled/default`, lalu `systemctl reload nginx`. |
