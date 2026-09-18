Jalankan perintah berikut di terminal server untuk membuat folder dan file HTML-nya.

**1. Buat direktori untuk kedua website:**

```bash
sudo mkdir -p /var/www/sekolah
sudo mkdir -p /var/www/kasir

```

**2. Buat file HTML Web Sekolah:**
Buka teks editor:

```bash
sudo nano /var/www/sekolah/index.html

```

Masukkan kodingan berikut, lalu simpan (Ctrl+O, Enter, Ctrl+X):

```html
<!DOCTYPE html>
<html>
<head>
    <title>Web Sekolah</title>
</head>
<body>
    <h1>Selamat Datang di Sistem Informasi Sekolah</h1>
    <p>Ini adalah halaman utama Web Sekolah yang berjalan secara lokal.</p>
</body>
</html>

```

**3. Buat file HTML Web Kasir:**
Buka teks editor:

```bash
sudo nano /var/www/kasir/index.html

```

Masukkan kodingan berikut, lalu simpan (Ctrl+O, Enter, Ctrl+X):

```html
<!DOCTYPE html>
<html>
<head>
    <title>Web Kasir</title>
</head>
<body>
    <h1>Sistem Kasir Utama</h1>
    <p>Ini adalah halaman utama Web Kasir yang berjalan secara lokal.</p>
</body>
</html>

```
