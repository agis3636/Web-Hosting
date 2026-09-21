# Cara setting IP Statik di Ubuntu 24.04 Server

### Langkah 1: Cari Tahu Nama File Konfigurasi

Ketik perintah ini untuk melihat nama file konfigurasi Netplan:

```bash
ls /etc/netplan/

```

*(Ingat nama filenya, biasanya berakhiran `.yaml`, contoh: `50-cloud-init.yaml` atau `00-installer-config.yaml`).*

### Langkah 2: Edit File Netplan

Buka file tersebut menggunakan editor Nano (ganti `nama_file.yaml` dengan hasil dari langkah 1):

```bash
sudo nano /etc/netplan/nama_file.yaml

```

### Langkah 3: Masukkan Konfigurasi IP Statik

Hapus semua isi di dalamnya, lalu ganti dengan kode berikut. (Pastikan indentasi/spasinya persis seperti ini, jangan gunakan tombol `Tab`, gunakan tombol `Spasi`).

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 192.168.2.50/24
      routes:
        - to: default
          via: 192.168.2.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1

```


### Langkah 4: Terapkan Konfigurasi

Jalankan perintah ini untuk menerapkan IP statik:

```bash
sudo netplan apply

```

Cek hasilnya dengan mengetik:

```bash
ip a

```
