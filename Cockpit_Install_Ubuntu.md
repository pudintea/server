# Cara Install Cockpit (Ubuntu 22.04 / 24.04)
Tentu. Untuk **Ubuntu 24.04 LTS**, instalasi Cockpit cukup mudah karena paketnya sudah tersedia di repository resmi Ubuntu.

## Apa itu Cockpit?

Cockpit adalah web panel untuk mengelola server Linux melalui browser. Anda bisa:

* Melihat penggunaan CPU, RAM, disk, dan network
* Mengelola service (`systemd`)
* Membuat dan menghapus user
* Melihat log sistem
* Mengelola storage
* Menjalankan terminal langsung dari browser
* Mengelola container (dengan plugin Podman)

Dokumentasi resmi:
[Cockpit Project](https://cockpit-project.org?utm_source=chatgpt.com)

---

# Spesifikasi Minimal

Untuk Ubuntu 24.04:

* CPU: 1 core
* RAM: 1 GB (minimal)
* RAM: 2 GB+ (direkomendasikan)
* Storage: sekitar 100 MB untuk instalasi dasar

Pada VPS 1–2 GB RAM, Cockpit biasanya terasa ringan.

---

# STEP 1 — Update Sistem

Masuk ke server:

```bash
ssh username@ip-server
```

Update paket:

```bash
sudo apt update
sudo apt upgrade -y
```

Opsional:

```bash
sudo reboot
```

Jika reboot, login kembali ke server.

---

# STEP 2 — Install Cockpit

Install paket Cockpit:

```bash
sudo apt install cockpit -y
```

Verifikasi:

```bash
dpkg -l | grep cockpit
```

Biasanya akan muncul paket seperti:

```text
cockpit
cockpit-bridge
cockpit-system
cockpit-ws
```

---

# STEP 3 — Jalankan dan Aktifkan Service

Cockpit menggunakan socket activation.

Aktifkan:

```bash
sudo systemctl enable --now cockpit.socket
```

Cek status:

```bash
sudo systemctl status cockpit.socket
```

Jika berhasil:

```text
Active: active (listening)
```

Keluar dari tampilan status:

```bash
q
```

---

# STEP 4 — Pastikan Port 9090 Terbuka

Cek:

```bash
sudo ss -tulpn | grep 9090
```

Output normal:

```text
LISTEN 0 4096 *:9090
```

Artinya Cockpit sudah berjalan.

---

# STEP 5 — Buka Firewall

Cek apakah UFW aktif:

```bash
sudo ufw status
```

Jika aktif, buka port 9090:

```bash
sudo ufw allow from 192.168.206.64/27 to any port 9090
```

Reload:

```bash
sudo ufw reload
```

Cek kembali:

```bash
sudo ufw status
```

Harus ada:

```text
9090/tcp ALLOW
```

---

# STEP 6 — Login ke Cockpit

Buka browser:

```text
https://IP_SERVER:9090
```

Contoh:

```text
https://103.xxx.xxx.xxx:9090
```

atau:

```text
https://domainanda.com:9090
```

Karena sertifikat bawaan self-signed, browser biasanya menampilkan peringatan keamanan.

Pilih:

```text
Advanced
Proceed
```

---

# STEP 7 — Login

Gunakan akun Ubuntu yang ada.

Misalnya:

```text
Username: ubuntu
Password: password-server
```

Atau user lain yang memiliki hak sudo.

Setelah login Anda akan melihat dashboard server.

---

# Menambahkan User Khusus Cockpit

Buat user:

```bash
sudo adduser admincockpit
```

Tambahkan ke grup sudo:

```bash
sudo usermod -aG sudo admincockpit
```

Login menggunakan user tersebut.

---

# Install Modul Tambahan yang Berguna

## Storage Management

```bash
sudo apt install cockpit-storaged -y
```

---

## Virtual Machine (KVM)

```bash
sudo apt install cockpit-machines -y
```

---

## Podman Container

```bash
sudo apt install cockpit-podman -y
```

Jika paket tidak ditemukan di Ubuntu 24.04, bisa menggunakan repository tambahan Cockpit.

---

# Apakah Port Bisa Diganti?

**Bisa.**

Secara default Cockpit berjalan di:

```text
9090
```

Ada beberapa cara.

---

## Cara 1 (Direkomendasikan): Gunakan Nginx Reverse Proxy

Biarkan Cockpit tetap di 9090 lalu akses melalui port 80 atau 443.

Install Nginx:

```bash
sudo apt install nginx -y
```

Buat konfigurasi:

```bash
sudo nano /etc/nginx/sites-available/cockpit
```

Isi:

```nginx
server {
    listen 80;

    server_name panel.domainanda.com;

    location / {
        proxy_pass https://127.0.0.1:9090;

        proxy_ssl_verify off;

        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Aktifkan:

```bash
sudo ln -s /etc/nginx/sites-available/cockpit /etc/nginx/sites-enabled/
```

Test:

```bash
sudo nginx -t
```

Reload:

```bash
sudo systemctl reload nginx
```

Sekarang akses:

```text
http://panel.domainanda.com
```

---

## Cara 2: Mengubah Port Cockpit Langsung

Buat override systemd:

```bash
sudo mkdir -p /etc/systemd/system/cockpit.socket.d
```

Buat file:

```bash
sudo nano /etc/systemd/system/cockpit.socket.d/listen.conf
```

Isi:

```ini
[Socket]
ListenStream=
ListenStream=8888
```

Perhatikan:

* Baris kosong `ListenStream=` menghapus port default 9090.
* `8888` bisa diganti sesuai kebutuhan.

Reload:

```bash
sudo systemctl daemon-reload
```

Restart:

```bash
sudo systemctl restart cockpit.socket
```

Cek:

```bash
sudo ss -tulpn | grep 8888
```

Jika muncul:

```text
LISTEN *:8888
```

maka berhasil.

Akses:

```text
https://IP_SERVER:8888
```

---

# Cara Mengembalikan ke Port 9090

Hapus file override:

```bash
sudo rm /etc/systemd/system/cockpit.socket.d/listen.conf
```

Reload:

```bash
sudo systemctl daemon-reload
sudo systemctl restart cockpit.socket
```

---

# Rekomendasi untuk Server Publik

Jika server dapat diakses dari internet:

1. Buat user non-root untuk login.
2. Gunakan SSH key.
3. Aktifkan UFW.
4. Gunakan domain dan HTTPS melalui Nginx.
5. Batasi akses Cockpit hanya dari IP tertentu jika memungkinkan.
6. Jangan login sebagai root kecuali benar-benar diperlukan.

Kalau Anda ingin mengakses Cockpit dari internet menggunakan domain (misalnya `panel.domain.com`) dengan SSL gratis dari Let's Encrypt, saya bisa berikan langkah lengkap konfigurasi Nginx + SSL + Cockpit untuk Ubuntu 24.04.
