# Tutorial Instalasi Wazuh di Debian (Step-by-Step)

Wazuh adalah platform SIEM dan XDR open-source yang digunakan untuk monitoring keamanan server, mendeteksi serangan, memantau aktivitas sistem, mendeteksi perubahan file, dan mengumpulkan log dari berbagai komputer.

Saya akan menggunakan metode All-in-One Installation, yaitu menginstal seluruh komponen Wazuh dalam satu server Debian.

Wazuh España: Implementación, ENS y Soporte | AI Security

[aisecurity.es](<https://aisecurity.es/wazuh?utm_source=chatgpt.com>)

## 1. Arsitektur Wazuh

Wazuh memiliki tiga komponen utama:

- Wazuh Manager: menerima dan menganalisis data dari agent.
- Wazuh Indexer: menyimpan dan mengindeks data keamanan.
- Wazuh Dashboard: antarmuka web untuk monitoring.

Gambaran arsitektur:

Wazuh Dashboard

Web Interface — HTTPS Port 443

Wazuh Manager

Analisis log dan deteksi ancaman

Wazuh Indexer

Penyimpanan dan pencarian data

Endpoint yang dimonitor

Debian AgentWindows AgentLinux Agent

## 2. Persiapan server Debian

Spesifikasi yang saya rekomendasikan untuk lab atau pembelajaran:

| Komponen | Rekomendasi |
| --- | --- |
| OS | Debian 12/13 64-bit |
| CPU | 4 vCPU |
| RAM | 8 GB |
| Storage | 50 GB atau lebih |
| Koneksi | Internet |
| Hak akses | Root / sudo |

Catatan: Wazuh Quickstart saat ini merekomendasikan 4 vCPU dan 8 GB RAM untuk deployment kecil. Pastikan versi Debian yang digunakan sesuai dengan persyaratan paket Wazuh yang dipilih.

GitHub

+1

## 3. Update sistem Debian

Login ke server Debian melalui SSH atau terminal.

Jalankan:

bash

```bash
sudo apt update
sudo apt upgrade -y
```

Install kebutuhan dasar:

bash

```bash
sudo apt install -y \
curl \
wget \
gnupg \
apt-transport-https \
ca-certificates \
lsb-release
```

Periksa waktu:

bash

```bash
timedatectl
```

Sinkronisasi waktu penting untuk menghindari masalah sertifikat dan perbedaan timestamp log.

Wazuh documentation

## 4. Download dan install Wazuh

Kita gunakan Wazuh versi 4.14 dari repository resmi.

Download installation assistant:

bash

```bash
cd /root

curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
```

Periksa file yang berhasil didownload:

bash

```bash
ls -lh wazuh-install.sh
```

Jalankan instalasi All-in-One:

bash

```bash
bash wazuh-install.sh -a
```

All-in-One InstallationPerintah tersebut akan menginstal dan mengonfigurasi:

1. Wazuh Indexer
2. Wazuh Manager
3. Wazuh Dashboard

Tunggu sampai proses selesai. Jangan mematikan server selama instalasi.

Dokumentasi resmi: [Wazuh Quickstart Installation](<https://documentation.wazuh.com/current/quickstart.html?trk=article-ssr-frontend-pulse_little-text-block&utm_source=chatgpt.com>).

## 5. Simpan username dan password Wazuh

Setelah instalasi berhasil, Wazuh akan menampilkan informasi login dashboard.

Biasanya:

text

```text
URL      : https://IP-SERVER
Username : admin
Password : PASSWORD-GENERATED
```

Jika ingin melihat password yang dihasilkan installer:

bash

```bash
tar -O -xvf wazuh-install-files.tar \
wazuh-install-files/wazuh-passwords.txt
```

Penting: Simpan password tersebut di tempat aman. Jangan membagikannya kepada orang lain.

## 6. Konfigurasi firewall Debian

Jika menggunakan UFW, install terlebih dahulu:

bash

```bash
sudo apt install ufw -y
```

Izinkan SSH agar koneksi remote tidak terputus:

bash

```bash
sudo ufw allow 22/tcp
```

Izinkan akses dashboard:

bash

```bash
sudo ufw allow 443/tcp
```

Izinkan komunikasi agent Wazuh 4.x:

bash

```bash
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
```

Aktifkan firewall:

bash

```bash
sudo ufw enable
```

Periksa status:

bash

```bash
sudo ufw status
```

Port yang perlu diketahui:

| Port | Fungsi |
| --- | --- |
| 443/TCP | Dashboard HTTPS |
| 1514/TCP | Komunikasi agent |
| 1515/TCP | Registrasi agent |
| 55000/TCP | Wazuh API |
| 9200/TCP | Indexer API |

Jangan membuka port API dan Indexer ke internet secara publik. Batasi aksesnya ke jaringan administrasi.

GitHub

+1

## 7. Cek status service Wazuh

Setelah instalasi, periksa service berikut:

bash

```bash
systemctl status wazuh-manager
```

bash

```bash
systemctl status wazuh-indexer
```

bash

```bash
systemctl status wazuh-dashboard
```

Pastikan ketiganya menunjukkan:

text

```text
Active: active (running)
```

Jika ingin melihat service secara bersamaan:

bash

```bash
systemctl --type=service | grep wazuh
```

## 8. Akses Wazuh Dashboard

Buka browser dari komputer Anda:

text

```text
https://IP-SERVER-DEBIAN
```

Contoh:

text

```text
https://192.168.1.100
```

Wazuh: Guide Complet pour l'installation

[blog.fikara.io](<https://blog.fikara.io/wazuh-installation?utm_source=chatgpt.com>)

Masukkan:

- Username: `admin`
- Password: password hasil instalasi.

Jika muncul peringatan sertifikat SSL pada browser, itu bisa terjadi karena instalasi menggunakan sertifikat self-signed bawaan. Untuk penggunaan produksi, sebaiknya gunakan sertifikat yang tepercaya.

Platform Overview: Open Source XDR & SIEM | Wazuh

[wazuh.com](<https://wazuh.com/platform/overview/?utm_source=chatgpt.com>)

## 9. Menambahkan Agent ke Wazuh

Setelah dashboard berhasil dibuka, langkah selanjutnya adalah menghubungkan komputer yang ingin dimonitor.

Contohnya:

- Server Debian lainnya.
- Server Ubuntu.
- Komputer Windows.
- Server Linux lainnya.

Caranya:

1. Buka Wazuh Dashboard.
2. Masuk ke menu Agents.
3. Pilih Deploy new agent.
4. Pilih sistem operasi endpoint.
5. Masukkan IP Wazuh Manager.
6. Ikuti perintah instalasi yang diberikan dashboard.
7. Jalankan service agent.

Contoh untuk Debian:

bash

```bash
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

Periksa status:

bash

```bash
sudo systemctl status wazuh-agent
```

Jika berhasil terhubung, agent akan muncul di menu Agents pada dashboard.

Dokumentasi: [Wazuh Agent Installation](<https://github.com/wazuh/wazuh/blob/main/docs/ref/getting-started/installation.md?utm_source=chatgpt.com>).

## 10. Troubleshooting dasar

Jika Wazuh tidak berjalan, gunakan beberapa perintah berikut.

Memeriksa log manager:

bash

```bash
sudo journalctl -u wazuh-manager -n 50
```

Memeriksa log indexer:

bash

```bash
sudo journalctl -u wazuh-indexer -n 50
```

Memeriksa log dashboard:

bash

```bash
sudo journalctl -u wazuh-dashboard -n 50
```

Restart service:

bash

```bash
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-dashboard
```

Memeriksa penggunaan RAM dan CPU:

bash

```bash
htop
```

## 11. Tips keamanan untuk penggunaan produksi

- Jangan membuka port 9200 dan 55000 untuk akses publik.
- Gunakan password administrator yang kuat.
- Batasi akses SSH.
- Aktifkan firewall.
- Lakukan backup konfigurasi dan data.
- Gunakan HTTPS dengan sertifikat tepercaya.
- Pastikan waktu server tersinkronisasi.
- Pantau kapasitas disk karena data log akan terus bertambah.

## Saya ingin memastikan tutorial berikutnya sesuai dengan server Anda

Ada satu hal penting: instalasi Wazuh pada Debian bisa disesuaikan dengan kebutuhan, terutama versi Debian dan kapasitas server.

Konfigurasi server Anda

1. Versi Debian

Debian 11

Debian 12 Debian 12

Debian 13

2. Model instalasi

All-in-One (semua komponen satu server) All-in-One (satu server)

Distributed (server terpisah)

3. Kebutuhan

Lab belajar SIEM / Cyber Security Lab belajar SIEM / Cyber Security

Monitoring server perusahaan

SOC (Security Operations Center)

4. RAM server

4 GB

8 GB 8 GB

16 GB

32 GB atau lebih

 Sesuai konfigurasi server saya

Saya bisa lanjutkan sampai Wazuh benar-benar berjalan, agent terhubung, dan dashboard bisa mendeteksi aktivitas keamanan server.
