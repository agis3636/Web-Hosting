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
