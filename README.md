# Premium SSH & VPN Panel

Script ini menyediakan panel SSH dan konfigurasi Xray VLESS/VMess/Trojan.

## Deploy Voroa

Gunakan **Web Service → Docker runtime** dengan Dockerfile `/Dockerfile`. Lihat [VOROA_DEPLOY.md](VOROA_DEPLOY.md) untuk environment variables, batasan port, dan langkah deployment.

Voroa hanya mempublikasikan satu port Web Service (`PORT=3000` untuk Docker) untuk HTTP dan WebSocket. Port backend lain dipertahankan pada loopback. Oleh sebab itu, koneksi TCP/UDP mentah ke port terpisah tidak tersedia langsung dari internet melalui Voroa.

## Deploy platform lain

Dockerfile tetap menggunakan Ubuntu 22.04, Node.js, Dropbear, Stunnel, Cloudflared dan BadVPN UDP gateway. Pastikan platform target mendukung Docker serta kebutuhan jaringan/port yang dipakai.

## Konfigurasi akun

- User SSH default: `dd`
- Password SSH default: `dd`
- Ubah `ADMIN_PASSWORD`, `TOKEN`, `UUID` dan pengaturan lain melalui environment variables platform.

Jangan commit token Cloudflare atau password asli ke repository publik.
