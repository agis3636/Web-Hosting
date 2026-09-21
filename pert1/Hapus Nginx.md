**1. Hapus Nginx yang Salah Sampai ke Akar**
Jalankan perintah ini di terminal untuk mencabut Nginx OS beserta seluruh dependensinya:

```bash
sudo apt purge nginx nginx-common nginx-core -y
sudo apt autoremove -y

```

**2. Bersihkan Sisa Konfigurasi Lama**
Hapus direktori Nginx yang tersisa agar tidak terjadi bentrok saat menginstal versi yang baru:

```bash
sudo rm -rf /etc/nginx/

```

