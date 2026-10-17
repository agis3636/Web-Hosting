# Membangun DNS Server dan Web Server Lokal

## Topologi jaringan

<img width="896" height="481" alt="Image" src="https://github.com/user-attachments/assets/49fbcdeb-2aa4-470e-89d3-7644e10b22f2" />

### Hasil praktek pertemuan kemarin

| Domain          | IP hasil DNS                   | Port web |
| --------------- | ------------------------------ | -------- |
| `sekolah.local` | `192.168.200.6`                | 8080     |
| `kasir.local`   | `192.168.200.6`                | 8181     |
| `google.com`    | IP publik hasil DNS forwarding | —        |

---

## Periksa alamat IP

Jalankan:

```
ip -br addr
ip route
```

Penjelasan:

- `ip -br addr` menampilkan nama interface dan IP secara ringkas.
- `ip route` menampilkan jalur jaringan, termasuk default gateway.

## Perbarui paket Ubuntu

```
sudo apt update
sudo apt install wget tar dnsutils -y
```

---

## 1. Unduh CoreDNS

```
wget https://github.com/coredns/coredns/releases/download/v1.11.1/coredns_1.11.1_linux_amd64.tgz
```

Penjelasan:

- `wget URL` mengunduh arsip CoreDNS dari GitHub.
- `v1.11.1` adalah versi CoreDNS yang digunakan dalam modul ini.
- `linux_amd64` menunjukkan binary untuk Linux 64-bit arsitektur x86-64.
- `.tgz` adalah arsip terkompresi yang perlu diekstrak.

Periksa file hasil download:

```
ls -lh ~/coredns_1.11.1_linux_amd64.tgz
```

## 2. Ekstrak

```
tar -xzf ~/coredns_1.11.1_linux_amd64.tgz
sudo install -m 755 coredns /usr/local/bin/coredns
sudo mkdir -p /etc/coredns
```

| Perintah                | Penjelasan                                                                    |
| ----------------------- | ----------------------------------------------------------------------------- |
| `tar -xzf`              | Mengekstrak arsip `.tgz`.                                                     |
| `install -m 755`        | Menyalin binary ke `/usr/local/bin/coredns` sekaligus mengatur izin eksekusi. |
| `mkdir -p /etc/coredns` | Membuat direktori konfigurasi CoreDNS jika belum ada.                         |

Verifikasi:

```
/usr/local/bin/coredns -version
```

---

## 3. Konfigurasi port DNS Ubuntu

Ubuntu 24.04 biasanya menggunakan `systemd-resolved` untuk menangani DNS lokal. Pada konfigurasi standar, layanan ini dapat menggunakan alamat `127.0.0.53` pada port 53.

CoreDNS juga perlu menggunakan port 53. Karena itu, kita perlu memastikan tidak ada konflik pada alamat yang akan digunakan.

### 6.1 Periksa port 53

```
sudo ss -lnutp | grep ':53'
```

Penjelasan:

- `ss` menampilkan socket jaringan.
- `-l` menampilkan socket yang sedang mendengarkan.
- `-n` menampilkan nomor port apa adanya.
- `-u` menampilkan UDP.
- `-t` menampilkan TCP.
- `-p` menampilkan proses pemilik socket.
- `grep ':53'` menyaring baris yang berkaitan dengan port 53.

## 4. Ubah konfigurasi `systemd-resolved`

Buka file:

```
sudo nano /etc/systemd/resolved.conf
```

Cari bagian `[Resolve]`, lalu pastikan baris berikut aktif:

```
[Resolve]
DNSStubListener=no
```

Penjelasan isi file:

- `[Resolve]` adalah bagian konfigurasi untuk layanan resolusi DNS.
- `DNSStubListener=no` menonaktifkan listener DNS stub lokal milik `systemd-resolved`, sehingga port 53 dapat digunakan CoreDNS pada alamat yang sama.

Terapkan perubahan:

```
sudo systemctl restart systemd-resolved
sudo ss -lnutp | grep ':53'
```

Hasil yang harus muncul di terminal kalian adalah "listener stub `systemd-resolved` pada `127.0.0.53:53` dan `127.0.0.54:53` sudah tidak muncul".

## 5. Membuat konfigurasi CoreDNS

File `Corefile` adalah file utama yang berisi aturan kerja CoreDNS. Di dalamnya kita menentukan domain lokal, IP tujuan, DNS forwarding, dan pencatatan log.

Buat atau edit file:

```
sudo nano /etc/coredns/Corefile
```

Isi file dengan konfigurasi berikut:

```
.:53 {
    errors
    log

    hosts {
        192.168.200.6 kasir.local sekolah.local
        fallthrough
    }

    forward . 8.8.8.8 1.1.1.1
    cache 30
}
```

### Penjelasan setiap bagian

| Syntax                                    | Fungsi                                                                                           |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `.:53`                                    | CoreDNS melayani permintaan DNS pada port 53 untuk semua nama domain.                            |
| `errors`                                  | Mencatat kesalahan yang terjadi saat pemrosesan DNS.                                             |
| `log`                                     | Mencatat permintaan DNS untuk membantu pemeriksaan.                                              |
| `hosts`                                   | Mengaktifkan pemetaan nama domain ke IP secara manual.                                           |
| `192.168.200.6 kasir.local sekolah.local` | Memetakan kedua domain ke IP Ubuntu Server.                                                      |
| `fallthrough`                             | Jika nama tidak ditemukan di daftar `hosts`, lanjutkan ke plugin berikutnya.                     |
| `forward . 8.8.8.8 1.1.1.1`               | Meneruskan permintaan domain yang tidak ditangani secara lokal ke Google DNS dan Cloudflare DNS. |
| `cache 30`                                | Menyimpan hasil DNS sementara selama TTL maksimum 30 detik sesuai konfigurasi cache.             |

Analogi sederhana: `hosts` adalah daftar kontak lokal, sedangkan `forward` adalah layanan pencarian eksternal ketika nama yang dicari tidak ada di daftar lokal.

Kenapa menggunakan `.:53`? Karena konfigurasi ini melayani domain lokal sekaligus domain internet dalam satu server DNS. Dengan begitu, klien tidak perlu memakai DNS berbeda untuk mencari `kasir.local` dan `google.com`.

### Jika IP server berubah

Misalnya Ubuntu VM di VirtualBox memperoleh IP `192.168.1.50`. Baris `hosts` harus diubah menjadi:

```
192.168.1.50 kasir.local sekolah.local
```

IP dalam konfigurasi harus sesuai dengan IP Ubuntu yang benar-benar digunakan klien.

## 6. Membuat service CoreDNS

Agar CoreDNS dapat dikelola melalui `systemctl` dan berjalan otomatis saat Ubuntu menyala, buat unit service systemd.

```
sudo nano /etc/systemd/system/coredns.service
```

Isi file:

```
[Unit]
Description=CoreDNS DNS Server
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/coredns -conf /etc/coredns/Corefile
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Penjelasan isi file

| Syntax                           | Fungsi                                                                               |
| -------------------------------- | ------------------------------------------------------------------------------------ |
| `[Unit]`                         | Bagian identitas dan ketergantungan service.                                         |
| `Description=CoreDNS DNS Server` | Nama deskriptif layanan.                                                             |
| `After=network-online.target`    | Meminta service dijalankan setelah target jaringan online tercapai.                  |
| `Wants=network-online.target`    | Meminta systemd mengaktifkan target jaringan online.                                 |
| `[Service]`                      | Bagian pengaturan proses yang dijalankan.                                            |
| `ExecStart=...`                  | Perintah yang menjalankan binary CoreDNS dengan file `Corefile`.                     |
| `Restart=on-failure`             | Meminta systemd menjalankan ulang layanan jika berhenti karena kegagalan.            |
| `RestartSec=5`                   | Jeda lima detik sebelum mencoba menjalankan ulang.                                   |
| `[Install]`                      | Bagian pengaturan aktivasi service.                                                  |
| `WantedBy=multi-user.target`     | Memungkinkan service diaktifkan untuk berjalan saat sistem masuk ke mode multi-user. |

Konfigurasi ini tidak menggunakan `User=` sehingga service berjalan sebagai `root` secara default. Untuk lab dasar, cara ini sederhana; pada server produksi, sebaiknya gunakan akun khusus dengan hak minimum yang diperlukan.

### Aktifkan service

```
sudo systemctl daemon-reload
sudo systemctl enable --now coredns
sudo systemctl status coredns --no-pager
```

Jika statusnya `active (running)`, lanjutkan ke pengujian.

Jika gagal:

```
sudo journalctl -u coredns -n 50 --no-pager
```

Perintah ini menampilkan 50 baris log terakhir CoreDNS untuk membantu menemukan masalah, misalnya kesalahan penulisan `Corefile` atau port yang sedang digunakan.

## Konfigurasi firewall UFW

UFW adalah firewall yang digunakan untuk mengatur lalu lintas jaringan masuk dan keluar pada Ubuntu.

Periksa statusnya:

```
sudo ufw status numbered
```

Jika firewall aktif, izinkan DNS dari jaringan lokal. Contoh untuk jaringan `192.168.200.0/24`:

```
sudo ufw allow from 192.168.200.0/24 to any port 53
sudo ufw allow from 192.168.200.0/24 to any port 8080 proto tcp
sudo ufw allow from 192.168.200.0/24 to any port 8181 proto tcp
```

Penjelasan:

- `allow`: mengizinkan lalu lintas yang sesuai dengan aturan.
- `from 192.168.200.0/24`: membatasi sumber ke subnet tersebut.
- `to any port 53`: mengizinkan akses ke port DNS.
- `proto tcp`: menentukan protokol TCP, seperti yang digunakan HTTP.
- Port 53 tanpa `proto` tertentu mengizinkan protokol TCP dan UDP sesuai aturan UFW.

DNS menggunakan UDP dan TCP port 53, sedangkan website HTTP pada konfigurasi ini menggunakan TCP port 8080 dan 8181.

Periksa kembali:

```
sudo ufw status numbered
```

## Pengujian CoreDNS dari Ubuntu

Sebelum menguji ke Windows client, pastikan CoreDNS berhasil menerjemahkan domain lokal dan meneruskan permintaan domain internet.

Jalankan:

```
nslookup kasir.local localhost
nslookup sekolah.local localhost
nslookup google.com localhost
```

Dalam praktik kita, ketiga pengujian ini berhasil. Itu membuktikan pemetaan domain lokal dan DNS forwarding bekerja dari Ubuntu.

Jika `nslookup` menampilkan `Server: localhost` dan `Address: 127.0.0.1`, itu berarti permintaan dikirim ke resolver lokal. Namun, untuk memastikan proses yang benar-benar menjawab, periksa juga status CoreDNS dan log servicenya.


### 11.3 Tes domain lokal ke client windows

```
nslookup kasir.local 192.168.200.6
nslookup sekolah.local 192.168.200.6
nslookup google.com 192.168.200.6
```
