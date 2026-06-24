# Cara Install Cockpit (Ubuntu 22.04 / 24.04)

**Cockpit** adalah panel administrasi server berbasis web untuk Linux yang memudahkan pengelolaan server melalui browser.
Dengan Cockpit, Anda bisa memantau penggunaan CPU, RAM, disk, jaringan, mengelola layanan (services), user, storage, container, log sistem, dan banyak tugas administrasi lainnya tanpa harus selalu menggunakan terminal.
Proyek ini dikembangkan oleh Cockpit dan bersifat open source. Dokumentasi resminya tersedia di [Cockpit Project](https://cockpit-project.org?utm_source=chatgpt.com).


## Step 1: Update Sistem

```bash
sudo apt update
sudo apt upgrade -y
```

---

## Step 2: Install Cockpit

```bash
sudo apt install cockpit -y
```

Cek apakah berhasil:

```bash
cockpit --version
```

---

## Step 3: Aktifkan Service

```bash
sudo systemctl enable --now cockpit.socket
```

Cek status:

```bash
sudo systemctl status cockpit.socket
```

Harus muncul status:

```text
active (listening)
```

---

## Step 4: Buka Firewall

Jika menggunakan UFW:

```bash
sudo ufw allow 9090/tcp
sudo ufw reload
```

Cek:

```bash
sudo ufw status
```

---

## Step 5: Akses dari Browser

Buka:

```text
https://IP_SERVER:9090
```

Contoh:

```text
https://192.168.1.100:9090
```

atau

```text
https://serverku.com:9090
```

Karena sertifikat bawaan bersifat self-signed, browser biasanya menampilkan peringatan keamanan. Pilih **Advanced → Continue**.

---

## Step 6: Login

Gunakan akun Linux yang sudah ada.

Contoh:

```text
Username: root
Password: ********
```

atau

```text
Username: ubuntu
Password: ********
```

---

# Install di Debian 12

```bash
sudo apt update
sudo apt install cockpit -y

sudo systemctl enable --now cockpit.socket
```

Akses:

```text
https://IP_SERVER:9090
```

---

# Fitur yang Bisa Ditambahkan

## Manajemen Podman Container

```bash
sudo apt install cockpit-podman
```

atau

```bash
sudo dnf install cockpit-podman
```

---

## Manajemen Mesin Virtual (KVM)

```bash
sudo apt install cockpit-machines
```

---

## Manajemen Storage

```bash
sudo apt install cockpit-storaged
```

---

## File Browser

Pada beberapa distro:

```bash
sudo apt install cockpit-files
```

Jika paket tidak tersedia, fitur file manager bisa dipasang melalui repository tambahan atau plugin komunitas.

---

# Cek Port Cockpit

Pastikan port 9090 aktif:

```bash
sudo ss -tulpn | grep 9090
```

Output normal:

```text
LISTEN 0 4096 *:9090
```

---

# Cara Uninstall

Ubuntu/Debian:

```bash
sudo apt remove cockpit -y
sudo apt autoremove -y
```

Rocky/AlmaLinux:

```bash
sudo dnf remove cockpit -y
```

---

# Kelebihan Cockpit

✅ Gratis dan open source
✅ Ringan dibanding panel hosting besar
✅ Monitoring server real-time
✅ Kelola service systemd dengan GUI
✅ Kelola user dan group
✅ Kelola Podman dan VM
✅ Mudah dipasang

# Kekurangan Cockpit

❌ Bukan panel hosting (tidak seperti cPanel)
❌ Tidak ada fitur otomatis membuat website/email hosting
❌ Fitur tambahan memerlukan plugin
