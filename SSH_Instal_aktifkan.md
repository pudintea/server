# INSTALL DAN AKTIFKAN SSH UBUNTU DAN DEBIAN
Tutorial bagi Pemula,

## Install
Buka terminal.
Masukan perintah dibawah ini:
Update Dan Instal
```
sudo apt update && sudo apt install openssh-server -y
```
Enable dan Restart
```
sudo systemctl enable --now ssh && sudo systemctl start ssh
```

Lihat status.
```
sudo systemctl status ssh
```

## Cara Menerapkan Konfigurasi SSH Key yang sudah dibuat di Putty Gen
Pastikan kamu menggunakan user yang ingin di set ssh keynya, misal mau set user userpudin
kalau kamu di root, silahkan pindah dulu ke user yang akan di konfigurasi SSH Key,
```
su - userpudin
```
```
cd ~
```
```
pwd
```
```
mkdir -p ~/.ssh && nano ~/.ssh/authorized_keys
```

Paste public key yang tadi disalin, lalu:
Save file
Set permission:
```
chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys
```
Contoh Public Key :
```
ssh-rsa 
sdsdsssssfsfs-CONTOH-ssnajsajhsauihsuihwjihdqwudghuqwhd
rsa-key-20260415
```

## Konfigurasi setelah mengaktifkan SSH Key
Backup dulu file asli Lanjut Buat file konfigurasi Baru
```
mv /etc/ssh/sshd_config /etc/ssh/asliiiii_sshd_config && nano /etc/ssh/sshd_config
```
Berikut adalah konfigurasi nya :

```
Include /etc/ssh/sshd_config.d/*.conf
Port 22
AddressFamily any

PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no
PubkeyAuthentication yes
PermitRootLogin no
MaxAuthTries 5
LoginGraceTime 30
PermitEmptyPasswords no

KbdInteractiveAuthentication no

X11Forwarding yes
PrintMotd no
UseDNS no

AcceptEnv LANG LC_*
Subsystem sftp  /usr/lib/openssh/sftp-server
```
Lanjut
Penjelasanya ini :
```
# Memasukkan konfigurasi tambahan SSH dari semua file .conf
# yang berada di direktori /etc/ssh/sshd_config.d/
Include /etc/ssh/sshd_config.d/*.conf

# Menentukan port yang digunakan server SSH.
# Port default SSH adalah 22.
Port 22

# Mengizinkan SSH menggunakan IPv4 dan IPv6.
AddressFamily any


# Menonaktifkan autentikasi menggunakan password.
# Pengguna harus menggunakan metode autentikasi lain, seperti SSH key.
PasswordAuthentication no

# Menonaktifkan autentikasi Challenge-Response.
ChallengeResponseAuthentication no

# Menonaktifkan integrasi PAM (Pluggable Authentication Modules).
# Perlu diperhatikan bahwa ini dapat memengaruhi kebijakan autentikasi,
# pembatasan akun, dan pengelolaan sesi pengguna.
UsePAM no

# Mengaktifkan autentikasi menggunakan public key.
# Pengguna dapat login menggunakan pasangan private key dan public key.
PubkeyAuthentication yes

# Melarang login langsung menggunakan akun root melalui SSH.
PermitRootLogin no

# Membatasi maksimal percobaan autentikasi sebanyak 3 kali.
# Jika gagal melebihi batas, koneksi autentikasi akan diputus.
MaxAuthTries 3

# Membatasi maksimal 3 sesi SSH dalam satu koneksi jaringan.
MaxSessions 3

# Menonaktifkan fitur X11 Forwarding.
# Aplikasi GUI dari server tidak dapat diteruskan ke komputer client melalui X11.
X11Forwarding no

# Memberikan waktu maksimal 30 detik untuk menyelesaikan proses login.
LoginGraceTime 30

# Melarang login menggunakan password kosong.
PermitEmptyPasswords no


# Menonaktifkan autentikasi interaktif berbasis keyboard.
# Misalnya, metode autentikasi yang meminta respons melalui keyboard.
KbdInteractiveAuthentication no


# Mengaktifkan X11 Forwarding.
# Memungkinkan aplikasi GUI dari server ditampilkan di komputer client.
X11Forwarding yes

# Menonaktifkan pesan MOTD (Message of the Day) saat pengguna login.
PrintMotd no

# Menonaktifkan pemeriksaan DNS terhadap alamat IP client.
# Dapat mempercepat proses koneksi SSH.
UseDNS no

# Mengizinkan client mengirim variabel lingkungan LANG dan LC_*
# yang berkaitan dengan bahasa dan pengaturan lokal.
AcceptEnv LANG LC_*

# Mengaktifkan layanan SFTP menggunakan program OpenSSH sftp-server.
Subsystem sftp /usr/lib/openssh/sftp-server
```

Jika ingin mengizinkan user tertentu, tambahkan
```
AllowUsers ubuntu user2 user3
```

## Pudin Saepudin : italazhar.com
