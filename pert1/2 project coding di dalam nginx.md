# Bagaimana jika ada 2 SourceCode yang tersimpan di Nginx?

Mereka berjalan dari direktorinya masing-masing menggunakan **port internal (localhost) yang berbeda**. Alamat IP `192.168.12.12` hanya digunakan sebagai pintu masuk utama dari luar.

Berikut alur jalannya:

1. **Di Direktori Web Sekolah (Next.js):** Anda menjalankan perintah *run* (misal pakai `pm2`), aplikasi jalan secara lokal di `127.0.0.1:3000`.
2. **Di Direktori Web Kasir (Python):** Anda menjalankan perintah *run* di direktorinya, aplikasi jalan secara lokal di `127.0.0.1:5000`.

Karena pintu masuk utama Anda hanya ada satu IP (`192.168.12.12`), Nginx bertugas membagi lalu lintasnya. Ada 3 metode yang bisa diterapkan di Nginx:

| Metode | URL yang Diketik Pengunjung | Cara Kerja Nginx |
| --- | --- | --- |
| **Beda Port** | `192.168.12.12` (Otomatis port 80 ke Next.js) `192.168.12.12:81` (Ke Python) | Nginx mendengarkan port 80 dan 81, lalu meneruskannya ke port lokal 3000 dan 5000. |
| **Beda Path** | `192.168.12.12/sekolah` `192.168.12.12/kasir` | Nginx mendengarkan port 80, membaca akhiran *path*, lalu meneruskan ke aplikasi yang sesuai. |
| **Beda Domain Lokal** | `sekolah.local` `kasir.local` | Nginx membedakan dari nama domain (`server_name`). Syaratnya, Anda harus mengatur DNS di router (misal: MikroTik) agar domain tersebut mengarah ke `192.168.12.12`. |

Cukup memilih salah satu metode di atas untuk diatur dalam file konfigurasi Nginx.
