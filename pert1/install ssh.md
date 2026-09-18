# Cara Instal OpenSSH Server di Ubuntu

### 1. Install

Jika Anda lupa mencentang "Install OpenSSH Server" saat proses instalasi awal tadi, Anda harus menginstalnya secara manual sekarang. Buka terminal Ubuntu di dalam VirtualBox dan ketik:

```bash
sudo apt update
sudo apt install openssh-server -y

```

Pastikan layanan SSH berjalan:

```bash
sudo systemctl status ssh

```

Jika tulisannya `active (running)`, berarti SSH sudah siap menerima koneksi.

### 2. Atur Jaringan VirtualBox (Penting!)

Agar laptop Anda bisa berkomunikasi dengan Ubuntu di VirtualBox melalui SSH, pengaturan jaringan mesin virtual harus diubah.

1. Matikan Ubuntu Server Anda terlebih dahulu (`sudo poweroff`).
2. Di jendela VirtualBox utama, klik kanan mesin virtual Ubuntu > **Settings**.
3. Buka menu **Network**.
4. Ubah "Attached to:" dari NAT menjadi **Bridged Adapter**.
*(Catatan: Bridged Adapter membuat Ubuntu mendapatkan IP dari Wi-Fi/Router yang sama dengan laptop Anda).*
5. Klik **OK** dan nyalakan ulang (*Start*) mesin virtualnya.

### 3. Cek IP Address Ubuntu

Setelah Ubuntu menyala dan login, ketik perintah ini untuk mengetahui alamat IP-nya:

```bash
ip a

```

Cari bagian `enp0s3` atau `eth0`, lalu catat angka setelah `inet` (misalnya `192.168.1.15`).

### 4. Hubungkan dari Laptop Asli (Windows)

1. Buka **Command Prompt (CMD)** atau **PowerShell** di Windows Anda.
2. Ketik perintah ini:
```bash
ssh username_ubuntu@ip_ubuntu

```


*(Contoh: `ssh agis@192.168.1.15`)*

3. Jika muncul pertanyaan *"Are you sure you want to continue connecting?"*, ketik `yes` lalu Enter.
4. Masukkan *password* Ubuntu Anda (saat mengetik *password*, teks tidak akan muncul di layar, ini normal).

Sekarang Anda sudah masuk ke server melalui CMD Windows dan bisa dengan bebas melakukan *copy-paste* perintah-perintah konfigurasi Nginx dan CoreDNS selanjutnya!

---
---
---

# Cara mengizinkan *login* SSH sebagai `root`

**1. Setel Password untuk Root (Jika Belum Ada)**
Secara bawaan, akun `root` di Ubuntu tidak memiliki *password* aktif. Anda harus membuat *password*-nya terlebih dahulu.
Ketik di terminal Ubuntu Anda (atau via koneksi SSH dari *user* biasa yang sudah aktif):

```bash
sudo passwd root

```

Masukkan *password* baru untuk `root` sebanyak dua kali.

**2. Edit File Konfigurasi SSH**
Buka file `sshd_config` menggunakan editor Nano:

```bash
sudo nano /etc/ssh/sshd_config

```

**3. Ubah Aturan PermitRootLogin**
Di dalam file tersebut, cari baris yang bertuliskan:
`#PermitRootLogin prohibit-password`
(Biasanya ada tanda pagar `#` di depannya, yang berarti baris itu dinonaktifkan).

Ubah baris tersebut menjadi:

```text
PermitRootLogin yes

```

*(Pastikan Anda menghapus tanda `#` di depannya).*
Simpan perubahan (Ctrl+O, Enter, Ctrl+X).

**4. Restart Layanan SSH**
Terapkan perubahan dengan memulai ulang layanan SSH:

```bash
sudo systemctl restart ssh

```

**5. Tes Login Root**
Sekarang, coba hubungkan ulang dari CMD/PowerShell di laptop Anda langsung menggunakan *username* `root`:

```bash
ssh root@ip_ubuntu

```

Masukkan *password* `root` yang baru Anda buat pada langkah pertama. Sekarang Anda sudah memiliki akses penuh sebagai administrator tertinggi tanpa perlu mengetik `sudo` lagi saat praktik instalasi Nginx dan konfigurasi HTML selanjutnya.

---
---
---

# Cara agar Network NAT VirtualBox bisa Connect dengan Host

### 1. Port Forwarding (Tetap Pakai NAT)

Gunakan cara ini jika Anda hanya butuh akses SSH ke Ubuntu tanpa mengubah pengaturan jaringan. Kekurangannya: **tetap tidak bisa di-ping**.

1. Di layar utama VirtualBox, klik kanan mesin virtual Ubuntu Anda -> **Settings** -> **Network**.
2. Di bagian **Adapter 1** (yang menggunakan NAT), klik menu panah **Advanced** di bawahnya.
3. Klik tombol **Port Forwarding**.
4. Klik ikon tambah (➕) berwarna hijau di pojok kanan atas, lalu isi datanya seperti ini:
    * **Name:** `SSH`
    * **Protocol:** `TCP`
    * **Host IP:** *(biarkan kosong)*
    * **Host Port:** `2222`
    * **Guest IP:** *(biarkan kosong)*
    * **Guest Port:** `22`


5. Klik **OK** dan simpan pengaturan.
6. Buka CMD Windows, jalankan perintah ini untuk masuk SSH:
`ssh root@127.0.0.1 -p 2222`

---

### 2.Tambah "Host-Only Adapter"

Gunakan cara ini jika target Anda adalah agar mesin virtual bisa **di-ping** dan dipanggil lewat domain lokal di laptop mahasiswa, tanpa perlu terhubung ke jaringan Wi-Fi kampus (karena Wi-Fi kampus sering memblokir komunikasi antar-klien).

1. Matikan Ubuntu Server Anda terlebih dahulu (`sudo poweroff`).
2. Buka **Settings** -> **Network** di VirtualBox.
3. Biarkan **Adapter 1** tetap **NAT** (agar Ubuntu tetap punya koneksi internet untuk menginstal Nginx).
4. Pindah ke tab **Adapter 2**, centang **Enable Network Adapter**, lalu ubah opsi *Attached to* menjadi **Host-only Adapter**.
5. Klik **OK** dan nyalakan ulang Ubuntu.
6. Setelah *login*, ketik perintah `ip a`.
7. Sekarang Anda akan melihat dua alamat IP aktif. Satu adalah `10.0.2.15` (milik NAT), dan satu lagi adalah IP baru (biasanya berawalan `192.168.56.x` milik Host-Only).

Gunakan IP baru yang berawalan `192.168.56.x` tersebut di CMD Windows Anda. Ping dan koneksi SSH (`ssh root@192.168.56.x`) akan langsung berhasil tersambung.
