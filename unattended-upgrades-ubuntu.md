# unattended-upgrades-ubuntu

## 1. Apa itu unattended-upgrades?

`unattended-upgrades` adalah mekanisme otomatis pada Ubuntu untuk menginstal pembaruan paket tanpa perlu intervensi administrator.

Misalnya, kamu mempunyai Ubuntu Server dengan software:

- OpenSSH
- Nginx
- OpenSSL
- Linux Kernel
- PHP
- System Libraries

Ketika Ubuntu menyediakan security patch untuk paket-paket tersebut, `unattended-upgrades` dapat menginstalnya secara otomatis sesuai kebijakan yang kamu konfigurasi.

## 2. Apa saja yang bisa dilakukan?

Berikut kemampuan utamanya:

| Fitur | Keterangan |
| --- | --- |
| Automatic security updates | Menginstal security patch |
| Package filtering | Memilih repository atau origin yang boleh di-update |
| Package blacklist | Mengecualikan paket tertentu |
| Automatic cleanup | Menghapus dependensi yang tidak diperlukan jika dikonfigurasi |
| Email notification | Memberikan laporan melalui email jika mail system dikonfigurasi |
| Automatic reboot | Bisa melakukan reboot setelah update jika diaktifkan |
| Logging | Mencatat aktivitas update |

Perlu dipahami bahwa `unattended-upgrades` bukan sistem patch management lengkap. Ia tidak menggantikan monitoring, backup, vulnerability management, atau pengujian aplikasi.

## 3. Standar keamanan yang saya rekomendasikan

Saya membedakan konfigurasi berdasarkan jenis server.

Server pribadiSecurity updates otomatis, regular updates boleh otomatis Cocok untuk VPS pribadi, homelab, dan development server.

Server productionSecurity updates otomatis, application updates terkontrol Cocok untuk web server, database, API, dan layanan bisnis.

### A. Untuk server pribadi

Saya menyarankan:

- Security updates: otomatis.
- Regular updates: boleh otomatis jika aplikasi kompatibel.
- Kernel updates: boleh otomatis dipasang.
- Automatic reboot: sebaiknya tetap manual.
- Backup: otomatis dan terjadwal.
- Monitoring: aktif.

Alasannya, server pribadi umumnya lebih fleksibel jika terjadi perubahan versi atau gangguan sementara.

### B. Untuk server production

Saya menyarankan kebijakan yang lebih ketat:

- Security updates: otomatis.
- Regular updates: melalui maintenance window atau pipeline deployment.
- Kernel updates: diuji dan dijadwalkan.
- Automatic reboot: nonaktif.
- Backup: wajib sebelum maintenance.
- Monitoring: wajib.
- Rollback plan: wajib.
- Staging environment: sangat disarankan.

Untuk production, jangan menganggap semua security patch pasti tidak memiliki dampak terhadap aplikasi.

## 4. Implementasi konfigurasi yang saya rekomendasikan

Sekarang kita masuk ke praktik.

Saya menyarankan menggunakan configuration drop-in, bukan mengubah konfigurasi bawaan Ubuntu secara langsung. Ubuntu juga merekomendasikan pendekatan ini agar konfigurasi kustom tidak mudah terganggu ketika paket diperbarui.

### Langkah 1 — Instal dan aktifkan

bash

```bash
sudo apt update
sudo apt install unattended-upgrades apt-listchanges
```

Aktifkan:

bash

```bash
sudo dpkg-reconfigure unattended-upgrades
```

Pilih `Yes`.

Periksa status:

bash

```bash
sudo systemctl status apt-daily.timer
sudo systemctl status apt-daily-upgrade.timer
```

Pastikan konfigurasi periodik:

bash

```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```

Konfigurasi yang diharapkan:

bash

```bash
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

Artinya:

- Refresh daftar repository setiap hari.
- Menjalankan automatic upgrades setiap hari.

Catatan: Tidak perlu menambahkan cron baru. Ubuntu sudah mengatur penjadwalannya melalui systemd timer.

Ubuntu Server documentation

### Langkah 2 — Buat konfigurasi khusus

Buat file:

bash

```bash
sudo nano /etc/apt/apt.conf.d/60unattended-local
```

Isi konfigurasi berikut:

bash

```bash
// Enable automatic security updates
Unattended-Upgrade::Origins-Pattern {
    "origin=Ubuntu,codename=${distro_codename}-security";
    "origin=UbuntuESMApps,codename=${distro_codename}-apps-security";
    "origin=UbuntuESM,codename=${distro_codename}-infra-security";
};

// Do not automatically reboot
Unattended-Upgrade::Automatic-Reboot "false";

// Remove unused dependencies
Unattended-Upgrade::Remove-Unused-Dependencies "true";

// Remove unused kernel packages
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";

// Keep package cache clean
Unattended-Upgrade::Remove-New-Unused-Dependencies "true";

// Log to syslog
Unattended-Upgrade::SyslogEnable "true";
```

Penjelasan konfigurasi:

| Parameter | Tujuan |
| --- | --- |
| Origins-Pattern | Membatasi sumber update |
| Automatic-Reboot | Mencegah reboot otomatis |
| Remove-Unused-Dependencies | Membersihkan dependensi tidak terpakai |
| Remove-Unused-Kernel-Packages | Membersihkan kernel lama yang tidak digunakan |
| SyslogEnable | Mengirim aktivitas ke syslog |

Catatan penting: pola origin harus sesuai metadata repository pada sistem. Ubuntu Pro/ESM juga hanya relevan jika tersedia dan diaktifkan. Untuk instalasi standar, periksa origin yang benar dengan `apt-cache policy`. Jangan menambahkan repository pihak ketiga secara otomatis tanpa evaluasi.

Saya sengaja tidak mengaktifkan automatic reboot.

Saya juga tidak menyarankan blacklist paket seperti `linux-*` atau `libc6` secara sembarangan. Paket inti tersebut tetap membutuhkan security patch.

### Langkah 3 — Uji konfigurasi sebelum diterapkan

Ini bagian yang sangat penting untuk Sysadmin.

Jalankan simulasi:

bash

```bash
sudo unattended-upgrade --dry-run --debug
```

Perintah ini tidak menginstal update. Tujuannya memeriksa paket yang akan diproses dan melihat apakah konfigurasi bekerja sebagaimana mestinya.

Ubuntu Manpages

Kemudian jalankan pemeriksaan:

bash

```bash
sudo apt-config dump | grep -E 'Unattended-Upgrade|APT::Periodic'
```

### Langkah 4 — Monitoring log

Lokasi log utama:

bash

```bash
/var/log/unattended-upgrades/
```

Lihat aktivitas:

bash

```bash
sudo tail -f /var/log/unattended-upgrades/unattended-upgrades.log
```

Lihat hasil instalasi paket:

bash

```bash
sudo tail -f /var/log/unattended-upgrades/unattended-upgrades-dpkg.log
```

Lihat histori paket:

bash

```bash
sudo grep " upgrade " /var/log/dpkg.log
```

Untuk production, jangan hanya mengaktifkan automatic updates lalu melupakannya. Buat monitoring dan alert ketika update gagal.

## 5. Apakah konfigurasi ini cocok untuk production dan pribadi?

Saya akan sedikit menyempurnakan contoh sebelumnya.

Untuk konfigurasi yang benar-benar aman, saya justru menyarankan mempertahankan Allowed Origins bawaan Ubuntu, kemudian hanya menambahkan konfigurasi lokal untuk perilaku update.

Jadi, tidak perlu mendefinisikan ulang `Origins-Pattern` jika repository bawaan sudah benar.

Buat atau ubah:

bash

```bash
sudo nano /etc/apt/apt.conf.d/60unattended-local
```

Gunakan konfigurasi berikut:

bash

```bash
// Never reboot automatically
Unattended-Upgrade::Automatic-Reboot "false";

// Remove unused dependencies
Unattended-Upgrade::Remove-Unused-Dependencies "true";

// Remove unused kernel packages
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";

// Enable syslog
Unattended-Upgrade::SyslogEnable "true";

// Enable verbose logging
Unattended-Upgrade::Verbose "true";
```

Dengan cara ini:

- Repository security bawaan Ubuntu tetap digunakan.
- Tidak sembarangan mengaktifkan repository tambahan.
- Tidak melakukan reboot otomatis.
- Dependensi dan kernel lama yang tidak diperlukan dapat dibersihkan.

Konfigurasi ini cocok sebagai baseline awal untuk kedua jenis server.

## 6. Hal penting yang sering dilupakan Sysadmin

Ada beberapa hal yang perlu kamu perhatikan.

### A. unattended-upgrades tidak meng-update semuanya

Contohnya:

| Software | Dikelola unattended-upgrades? |
| --- | --- |
| Nginx dari APT | Ya, sesuai repository yang diizinkan |
| OpenSSH dari APT | Ya |
| OpenSSL dari APT | Ya |
| PHP dari APT | Ya, sesuai repository yang diizinkan |
| Docker image | Tidak |
| Container image | Tidak |
| Aplikasi Node.js via npm | Tidak |
| Aplikasi Python via pip | Tidak |
| Snap packages | Tidak, punya mekanisme refresh sendiri |

Jadi, jika kamu menjalankan Docker, kamu tetap membutuhkan strategi update image dan container secara terpisah.

### B. Jangan lupa backup

Sebelum update besar, pastikan ada backup.

Untuk production, saya menyarankan:

- Backup database secara berkala.
- Backup konfigurasi Nginx, SSH, dan aplikasi.
- Snapshot VPS sebelum maintenance besar jika provider mendukung.
- Uji restore backup secara berkala.

Backup yang tidak pernah diuji restore belum bisa dianggap sebagai mekanisme recovery yang andal.

### C. Jangan lupa reboot

Cek apakah Ubuntu membutuhkan reboot:

bash

```bash
test -f /var/run/reboot-required \
  && echo "REBOOT REQUIRED" \
  || echo "SYSTEM OK"
```

Jika diperlukan, jadwalkan reboot melalui maintenance window.

### D. Perhatikan Ubuntu Pro

Untuk server yang membutuhkan cakupan security maintenance lebih luas, pertimbangkan Ubuntu Pro.

Ubuntu Pro menyediakan ESM untuk paket-paket tertentu di luar cakupan standard security maintenance.

Dokumentasi: [Ubuntu Security Updates](<https://documentation.ubuntu.com/security/security-updates/?utm_source=chatgpt.com>).


## 7. Referensi dokumentasi resmi

Saya sarankan menyimpan dokumentasi berikut sebagai referensi ketika mengelola Ubuntu:

1. [Ubuntu – Automatic Updates](<https://ubuntu.com/server/docs/how-to/software/automatic-updates/?utm_source=chatgpt.com>) — konfigurasi dan cara kerja unattended-upgrades.
2. [Ubuntu – Security Updates](<https://documentation.ubuntu.com/security/security-updates/?utm_source=chatgpt.com>) — security patch dan Ubuntu Pro.
3. [Ubuntu Manpage – unattended-upgrade](<https://manpages.ubuntu.com/manpages/resolute/man8/unattended-upgrades.8.html?utm_source=chatgpt.com>) — dokumentasi teknis parameter dan opsi.
4. [Ubuntu – Security Suggestions](<https://ubuntu.com/server/docs/explanation/security/security_suggestions/?utm_source=chatgpt.com>) — panduan keamanan server.

## PUDIN SAEPUDIN
