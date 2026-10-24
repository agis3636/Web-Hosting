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
