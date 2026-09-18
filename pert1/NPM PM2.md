# Apa bedanya  NPM dengan PM2?

NPM dan PM2 adalah dua aplikasi yang berbeda, tetapi mereka saling bekerja sama.

**NPM (Node Package Manager)** adalah program bawaan Node.js yang bertugas membaca dan menjalankan kodingan Next.js Anda. Perintah aslinya memang `npm run start`.

Namun, jika Anda langsung mengetik `npm run start` di terminal, terminal tersebut akan "terkunci" untuk menampilkan *log* proses berjalannya web. Begitu Anda menutup jendela SSH di laptop Anda, perintah `npm` akan terhenti paksa dan web sekolah Anda langsung *down* (mati).

Di sinilah **PM2** masuk sebagai solusi.

**PM2 (Process Manager 2)** adalah program pihak ketiga yang bertugas menjalankan perintah NPM tersebut ke latar belakang (*background*). PM2 bertindak sebagai manajer yang memastikan NPM tidak mati meskipun Anda keluar dari server.

**Analogi Sederhana:**

* **NPM** adalah mesin pembuat kopi. Jika Anda yang menekan tombolnya (manual di terminal), Anda harus berdiri terus di depan mesin menunggunya. Jika Anda pergi, mesin mati.
* **PM2** adalah robot asisten. Anda menyuruh PM2 untuk menekan tombol mesin kopi (NPM). Setelah itu, Anda bisa bebas pergi menutup pintu (menutup SSH), dan PM2 akan berjaga 24 jam memastikan mesin kopi itu tetap menyala.

Itulah sebabnya pada tahap *deploy* untuk web Next.js, perintahnya digabung. Anda menyuruh PM2 untuk menjalankan NPM:
`pm2 start npm --name "web-sekolah" -- run start`

Dengan perintah tersebut, aplikasi jalan stabil, terminal Anda langsung bersih kembali, dan Anda bisa lanjut mengonfigurasi web yang satunya (Python).
