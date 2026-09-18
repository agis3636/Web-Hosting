# cara pengaturan optimal di VirtualBox agar Ubuntu 24.04 tidak *lag*

### Kebutuhan *resource* antara versi Server (hanya teks) dan Desktop (GUI) sangat berbeda.

## **1. Pengaturan RAM & CPU (Menu: System)**

* **Ubuntu Server (CLI):** Alokasikan RAM 1 GB (1024 MB) hingga 2 GB. Prosesor cukup 1 Core. Server murni sangat ringan dan tidak akan membebani laptop.
* **Ubuntu Desktop (GUI):** Alokasikan RAM minimal 4 GB (4096 MB) dan Prosesor minimal 2 Cores. *Penting:* Pastikan *slider* RAM dan CPU tetap berada di zona hijau agar OS utama (Windows/Mac) laptop mahasiswa tidak ikut *hang*.

## **2. Pengaturan Grafis (Menu: Display) — Sangat Kritis untuk Desktop**
Penyebab utama animasi patah-patah pada Ubuntu Desktop di *virtual machine* adalah kegagalan *rendering* grafis GNOME.

* **Video Memory:** Geser *slider* mentok kanan ke maksimal **128 MB**.
* **Graphics Controller:** Gunakan **VMSVGA** (standar terbaik untuk Linux).
* **Enable 3D Acceleration:** **Wajib dicentang** khusus untuk Ubuntu Desktop. (Abaikan untuk Server).

## **3. Pengaturan Penyimpanan (Menu: Storage)**

* Klik file *virtual hard disk* (`.vdi` pada menu *Storage Devices*), lalu **centang "Solid-state Drive"**. Jika laptop mahasiswa sudah menggunakan SSD, opsi ini akan menyesuaikan antarmuka baca-tulis I/O agar jauh lebih cepat dan tidak *lag* saat *booting*.

## **4. Solusi Final Pasca-Instalasi (Khusus Desktop)**
Setelah Ubuntu Desktop selesai diinstal, mahasiswa **wajib** memasang *VirtualBox Guest Additions*. Tanpa ini, *mouse* akan terasa berat, resolusi layar terkunci kecil, dan fitur *copy-paste* dari Windows ke Ubuntu tidak akan jalan.
Minta mahasiswa membuka terminal di Ubuntu mereka dan jalankan perintah ini:
`sudo apt update && sudo apt install virtualbox-guest-x11 virtualbox-guest-utils virtualbox-guest-dkms -y`
Lalu *restart* mesin virtual tersebut.
