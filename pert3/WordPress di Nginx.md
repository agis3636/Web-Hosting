# Multi-Metode Akses Website WordPress di Nginx (Ubuntu 24.04)

---

## Tahap 1: Instalasi Paket Pendukung LEMP Stack & Database

Sebelum menjalankan website, pastikan mesin *database* dan penerjemah PHP sudah terpasang.

```bash
sudo apt update
sudo apt install mariadb-server php-fpm php-mysql php-curl php-xml php-gd php-mbstring php-zip -y

```

* **`mariadb-server`**: Aplikasi *database* relasional untuk menyimpan seluruh data konten dan pengaturan WordPress.
* **`php-fpm`**: *FastCGI Process Manager* yang bertugas memproses skrip PHP dari Nginx.
* **`php-mysql`, dll**: Kumpulan pustaka/ekstensi tambahan agar WordPress dapat berjalan tanpa kendala fungsi.

---

## Tahap 2: Pembuatan Database WordPress

Membuat wadah penyimpanan data khusus untuk aplikasi WordPress di MariaDB.

1. Masuk ke terminal MariaDB:
```bash
sudo mariadb -u root

```


2. Jalankan perintah SQL:


```sql
CREATE DATABASE db_wp;
CREATE USER 'agis'@'localhost' IDENTIFIED BY 'agis';
GRANT ALL PRIVILEGES ON db_wp.* TO 'agis'@'localhost';
FLUSH PRIVILEGES;
EXIT;

```


* **`CREATE DATABASE db_wp`**: Membuat *database* baru bernama `db_wp`.


* **`CREATE USER` & `GRANT**`: Membuat akun *user* `agis` dengan password `agis` dan memberikan hak akses penuh hanya pada *database* tersebut.





---

## Tahap 3: Pengunduhan File & Hak Akses Direktori

Mengunduh master WordPress resmi dan mengatur kepemilikan foldernya agar dapat dibaca oleh sistem.

```bash
sudo mkdir -p /var/www/wordpress
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzvf latest.tar.gz
sudo cp -a wordpress/. /var/www/wordpress/
sudo chown -R www-data:www-data /var/www/wordpress
sudo chmod -R 755 /var/www/wordpress

```

* **`mkdir -p /var/www/wordpress`**: Membuat direktori fisik penampung file web.
* **`wget` & `tar**`: Mengunduh dan mengekstrak file arsip WordPress terbaru.
* **`chown -R www-data:www-data`**: Mengubah kepemilikan folder agar dikuasai oleh proses Nginx (`www-data`).
* **`chmod -R 755`**: Memberikan izin standar aman (baca, tulis, eksekusi) pada folder web.

---

## Tahap 4: Konfigurasi DNS Lokal (CoreDNS)

Mendaftarkan nama domain agar terhubung ke alamat IP server.

1. Buka file konfigurasi:
```bash
sudo nano /etc/coredns/Corefile

```


2. Tambahkan domain `wordpress.local` ke dalam blok *hosts*:
```text
wordpress.local {
    hosts {
        192.168.200.6 wordpress.local
        fallthrough
    }
    forward . 8.8.8.8
    log
    errors
}

```


3. Mulai ulang CoreDNS:
```bash
sudo systemctl restart coredns

```


* **`hosts`**: Memetakan domain `wordpress.local` langsung menuju IP server `192.168.200.6` tanpa harus keluar ke internet.



---

## Tahap 5: Konfigurasi Nginx (3 Metode dalam 1 File)

Membuat satu file konfigurasi penampung tiga metode akses sekaligus (`/wordpress`, `:8080`, dan standar domain).

Buka file konfigurasi:

```bash
sudo nano /etc/nginx/sites-available/wordpress

```

Isi dengan kode berikut:

```nginx
# ==========================================
# METODE 1: Akses via IP Sub-Path (Port 80)
# Contoh: http://192.168.200.6/wordpress
# ==========================================
server {
    listen 80 default_server;
    server_name _;

    location /wordpress {
        alias /var/www/wordpress/;
        index index.php index.html;
        try_files $uri $uri/ /wordpress/index.php?$args;

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_param SCRIPT_FILENAME $request_filename;
            fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        }
    }
}

# ==========================================
# METODE 2: Akses via Port Khusus (Port 8080)
# Contoh: http://wordpress.local:8080
# ==========================================
server {
    listen 8080;
    server_name wordpress.local;
    root /var/www/wordpress;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    }
}

# ==========================================
# METODE 3: Standar Industri (Port 80 Murni)
# Contoh: http://wordpress.local
# ==========================================
server {
    listen 80;
    server_name wordpress.local;
    root /var/www/wordpress;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    }
}

```

* **`listen 80 default_server`**: Menangani seluruh lalu lintas IP utama yang tidak memiliki aturan domain spesifik (digunakan untuk metode sub-path).
* **`alias`**: Mengarahkan URL *sub-path* (`/wordpress`) secara langsung ke folder fisik server.
* **`try_files`**: Mengatur penulisan ulang URL agar mendukung struktur *permalink* WordPress tanpa memunculkan *Error 404*.
* **`fastcgi_pass`**: Jalur komunikasi *socket* penghubung antara Nginx dan mesin PHP-FPM.

Aktifkan file dan terapkan perubahan:

```bash
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

```
