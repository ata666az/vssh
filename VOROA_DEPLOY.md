# Deploy vmesssh di Voroa

Paket ini disiapkan untuk **Voroa Web Service dengan Docker runtime**. Dashboard, API, dan upgrade WebSocket menggunakan satu listener publik. Proses SSH/Xray pendukung tetap berjalan sebagai backend internal di loopback.

## Pengaturan service

1. Push isi repository ini ke GitHub.
2. Di Voroa pilih **New service → Web Service**, hubungkan repository dan branch yang benar.
3. Pilih **Docker runtime** dan Dockerfile `/Dockerfile`.
4. Jangan gunakan Static Site atau Background Worker. Service ini harus menjawab HTTP dan menerima WebSocket.
5. Deploy. Pada Docker runtime, perintah start berasal dari `CMD` dalam Dockerfile; field Start command dashboard tidak menggantikan `CMD`.

Dockerfile menetapkan default `PORT=3000`, dan `server.js` juga membaca `process.env.PORT`. Voroa menetapkan port Docker Web Service ke `3000`; hanya server UI/API utama yang bind ke `0.0.0.0:$PORT`.

## Environment variables

Tambahkan melalui Settings → Environment Variables di Voroa (jangan menyimpan secret ke GitHub):

| Nama | Wajib | Keterangan |
|---|---:|---|
| `TOKEN` | Opsional | Token Cloudflare Named Tunnel. Jika diisi, tunnel diarahkan ke WS-proxy internal pada port `8880`.
| `ADMIN_PASSWORD` | Sangat disarankan | Password admin awal yang kuat. Mencegah pengunjung pertama mengambil setup password admin.
| `UUID` | Opsional | UUID untuk VLESS/VMess/Trojan. Jika kosong, script memakai nilai default di `server.js`.
| `NAME` | Opsional | Nama node pada konfigurasi. Default `ata`.
| `CFIP` | Opsional | IP/host tujuan untuk template konfigurasi; default `104.17.3.81`.
| `CFPORT` | Opsional | Port tujuan template konfigurasi; default `443`.
| `SUB_PATH` | Opsional | Path subscription; default `ata`.

`XRAY_PORT` (default `8001`), `WS_PORT` (default `8880`) dan `MUX_PORT` (default `8881`) tidak perlu diubah.

## Perubahan yang diterapkan

- HTTP UI/API: `process.env.PORT` (Voroa Docker default `3000`), bind `0.0.0.0`.
- Dropbear: `127.0.0.1:22` (sudah internal).
- Stunnel: `127.0.0.1:2443` (sudah internal).
- Xray inbound utama: `127.0.0.1:8001`; fallback WS ports tetap di `127.0.0.1`.
- WS-proxy: `127.0.0.1:8880`.
- Mux: `127.0.0.1:8881`.
- BadVPN UDP gateway: `127.0.0.1:7300`.
- Cloudflare quick/named tunnel tetap dapat membuat koneksi keluar menuju port internal.

## Batasan jaringan Voroa yang penting

Voroa Web Service hanya memiliki **satu port publik**. HTTP dan WebSocket harus memakai listener yang sama (`PORT`). Karena itu, port TCP/UDP terpisah seperti SSH mentah `22`, TLS mentah `2443`, Mux `8881`, dan UDP gateway `7300` tidak dapat diakses langsung dari internet melalui alamat Voroa. Pada paket ini port-port tersebut sengaja dibuat loopback-only agar tidak memicu error `listening on more than one port`.

Jalur SSH WebSocket pada server utama tersedia melalui `wss://<service>.getvoroa.com/ssh-ws` (ganti domain dengan URL service Anda). Named Cloudflare Tunnel (jika `TOKEN` diisi) diarahkan langsung ke WS-proxy internal `8880`. Jalur Xray WS tetap dijalankan melalui Cloudflare quick tunnel internal pada `XRAY_PORT`. Periksa alamat dan status tunnel di panel setelah service Live.

Mode TCP langsung melalui Mux dan UDP gateway publik bukan fitur yang dapat dipertahankan sebagai port publik pada satu Voroa Web Service. Untuk kebutuhan TCP/UDP publik terpisah, gunakan VPS/host yang menyediakan inbound TCP/UDP ports atau layanan tunneling yang memang mendukung protokol tersebut.

## Catatan WebSocket

Dokumentasi Voroa menyatakan koneksi WebSocket idle yang tidak membawa data dapat diputus setelah kira-kira 100 detik; dokumentasinya menyarankan ping/pong setidaknya setiap 30 detik. Script asli melakukan raw socket piping dan tidak menambahkan heartbeat WebSocket tingkat aplikasi pada jalur `/ssh-ws`, jadi uji koneksi idle/panjang dari client Anda. Koneksi juga akan terputus ketika service di-deploy ulang atau restart; client perlu melakukan reconnect.

## Jika deploy masih gagal

- Pastikan runtime yang dipilih adalah Docker dan file path `/Dockerfile`.
- Pastikan `PORT` tidak dioverride menjadi port lain.
- Baca 50 baris terakhir pada bagian `Output from your service before it stopped`.
- Error build berbeda dengan error start: log Anda sebelumnya menunjukkan build sukses dan deploy dibatalkan secara khusus karena lima port publik.
- Jangan expose kembali port internal ke `0.0.0.0`.
