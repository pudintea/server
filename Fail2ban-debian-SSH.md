# Konfigurasi Fail2ban SSH aman – Debian 13

Rekomendasi untuk productionUntuk server Debian 13 (Trixie), port SSH 22, dan autentikasi SSH Key, saya sarankan menggunakan:

- Fail2ban dengan backend `systemd`.
- Firewall action `nftables`.
- Maksimal 3 percobaan gagal dalam 10 menit.
- Blokir awal 1 jam.
- Progressive ban untuk IP yang berulang kali menyerang.
- Recidive jail untuk pelaku yang terus mengulangi serangan.

Debian 13 sudah menggunakan nftables sebagai default action Fail2ban, sehingga kita tidak perlu memaksakan iptables.

fail2ban.dev

+1

Penting: Karena firewall server belum diketahui, kita akan memeriksa konfigurasi yang ada terlebih dahulu. Jangan mengubah atau menghapus rules firewall yang sudah berjalan.

## 1. Persiapan dan pemeriksaan

Jalankan:

bash

```bash
cat /etc/os-release
sudo sshd -T | grep -E 'port|passwordauthentication|pubkeyauthentication|permitrootlogin'
sudo systemctl status ssh
```

Periksa firewall:

bash

```bash
sudo nft list ruleset
sudo ufw status verbose
sudo iptables -S
```

Perintah di atas hanya untuk pemeriksaan.

Install paket:

bash

```bash
sudo apt update
sudo apt install fail2ban nftables python3-systemd -y
```

Periksa versi:

bash

```bash
fail2ban-client --version
nft --version
```

## 2. Buat konfigurasi Fail2ban

Buat file:

bash

```bash
sudo nano /etc/fail2ban/jail.d/sshd.local
```

Masukkan konfigurasi:

ini

```ini
[DEFAULT]

# IP yang tidak boleh diblokir
ignoreip = 127.0.0.1/8 ::1

# Waktu blokir awal
bantime = 1h

# Rentang penghitungan percobaan
findtime = 10m

# Maksimal kegagalan
maxretry = 3

# Progressive ban
bantime.increment = true
bantime.factor = 2
bantime.maxtime = 1w
bantime.rndtime = 10m

# Database penyimpanan ban
dbfile = /var/lib/fail2ban/fail2ban.sqlite3


[sshd]

enabled = true
port = 22
filter = sshd
backend = systemd

maxretry = 3
findtime = 10m
bantime = 1h

# Gunakan nftables
banaction = nftables
```

Keterangan progressive ban:

| Kondisi | Durasi |
| --- | --- |
| Ban pertama | 1 jam |
| Ban berikutnya | 2 jam |
| Ban berikutnya | 4 jam |
| Ban berikutnya | 8 jam |
| Maksimum | 1 minggu |

Durasi aktual bergantung pada riwayat pelanggaran dan pengaturan incremental ban.

## 3. Aktifkan perlindungan terhadap pelaku berulang (Recidive)

Recidive merupakan jail tambahan. Fungsinya memblokir IP yang sudah beberapa kali terkena ban Fail2ban.

Buat file:

bash

```bash
sudo nano /etc/fail2ban/jail.d/recidive.local
```

Isi:

ini

```ini
[recidive]

enabled = true
filter = recidive

logpath = /var/log/fail2ban.log
backend = polling

findtime = 1d
maxretry = 5
bantime = 1w

banaction = nftables[type=allports]
```

Artinya, IP yang tercatat terkena ban sebanyak 5 kali dalam 24 jam akan diblokir selama 7 hari pada seluruh port.

Catatan: Recidive bergantung pada file `/var/log/fail2ban.log`. Pastikan file tersebut tersedia dan Fail2ban mencatat aktivitasnya:

bash

```bash
ls -l /var/log/fail2ban.log
```

Jika file tidak tersedia, jangan aktifkan jail recidive dahulu. Konfigurasi SSH utama tetap bisa berjalan tanpa recidive.

## 4. Validasi dan jalankan Fail2ban

Periksa konfigurasi:

bash

```bash
sudo fail2ban-client -t
```

Jika hasilnya:

text

```text
OK
```

Restart:

bash

```bash
sudo systemctl enable fail2ban
sudo systemctl restart fail2ban
```

Periksa:

bash

```bash
sudo systemctl status fail2ban
sudo fail2ban-client ping
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

Hasil yang diharapkan:

- Fail2ban aktif.
- SSH jail aktif.
- Backend membaca systemd journal.
- Tidak ada error pada action nftables.

## 5. Pengujian firewall dan ban

Periksa apakah Fail2ban membuat tabel nftables:

bash

```bash
sudo nft list table inet f2b-table
```

Untuk menguji mekanisme ban tanpa melakukan serangan login, gunakan alamat dokumentasi khusus:

bash

```bash
sudo fail2ban-client set sshd banip 192.0.2.123
```

Periksa:

bash

```bash
sudo nft list table inet f2b-table
sudo fail2ban-client status sshd
```

Kemudian hapus ban pengujian:

bash

```bash
sudo fail2ban-client set sshd unbanip 192.0.2.123
```

Jangan menguji dengan IP publik milik sendiri karena bisa menyebabkan koneksi SSH terputus.

## 6. Pengujian log SSH

Jalankan:

bash

```bash
sudo journalctl -u ssh -f
```

Buka terminal lain dan periksa log Fail2ban:

bash

```bash
sudo journalctl -u fail2ban -f
```

Untuk menguji kemampuan filter membaca log:

bash

```bash
sudo fail2ban-regex systemd-journal sshd
```

Perintah ini memeriksa apakah filter SSH mengenali pola kegagalan autentikasi dari journal.

fail2ban.dev

+1

## 7. Hardening SSH tambahan

Karena server menggunakan SSH Key, periksa konfigurasi:

bash

```bash
sudo nano /etc/ssh/sshd_config
```

Pastikan pengaturan berikut aktif:

ini

```ini
Port 22
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
MaxAuthTries 3
MaxSessions 3
X11Forwarding no
AllowTcpForwarding no
```

Catatan:

- `AllowTcpForwarding no` hanya jika server tidak membutuhkan SSH tunneling atau port forwarding.
- Jika masih menggunakan root untuk administrasi, pastikan user sudo sudah dapat login sebelum mengaktifkan `PermitRootLogin no`.
- Debian dapat memiliki konfigurasi tambahan di `/etc/ssh/sshd_config.d/`. Periksa hasil akhirnya dengan `sshd -T`.

Validasi:

bash

```bash
sudo sshd -t
sudo sshd -T | grep -E 'permitrootlogin|passwordauthentication|pubkeyauthentication|kbdinteractiveauthentication|maxauthtries'
```

Jika sudah benar:

bash

```bash
sudo systemctl reload ssh
```

Jangan tutup sesi SSH yang sedang aktif sebelum berhasil login kembali menggunakan SSH Key dari terminal baru.

## 8. Monitoring keamanan harian

Beberapa perintah penting:

| Kebutuhan | Perintah |
| --- | --- |
| Status Fail2ban | `sudo fail2ban-client status` |
| Status SSH jail | `sudo fail2ban-client status sshd` |
| Lihat IP diblokir | `sudo fail2ban-client get sshd banip` |
| Lihat log SSH | `sudo journalctl -u ssh` |
| Lihat log Fail2ban | `sudo journalctl -u fail2ban` |
| Lihat firewall | `sudo nft list ruleset` |

Untuk melihat IP yang diblokir secara langsung:

bash

```bash
sudo fail2ban-client status sshd
```

### PUDIN SAEPUDIN
