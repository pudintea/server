# Github Dua Akun
Kalau menggunakan **GitHub Desktop**, mengelola dua akun (kantor dan pribadi) bisa dilakukan dengan rapi. Kuncinya adalah memisahkan **akun GitHub**, **identitas Git (nama/email)**, dan **SSH key** (atau HTTPS login).

Berikut pendekatan yang paling umum dan minim masalah.

### Opsi 1 (Paling direkomendasikan): Satu akun GitHub Desktop + SSH untuk kedua akun

GitHub Desktop hanya mendukung satu akun GitHub yang aktif untuk sinkronisasi fitur tertentu, tetapi Git sendiri bisa menggunakan SSH yang berbeda untuk setiap akun.

#### 1. Buat dua SSH key

Misalnya:

```bash
ssh-keygen -t ed25519 -C "email-pribadi@example.com"
```

Simpan sebagai:

```
~/.ssh/id_ed25519_personal
```

Lalu buat lagi untuk kantor:

```bash
ssh-keygen -t ed25519 -C "email-kantor@company.com"
```

Simpan sebagai:

```
~/.ssh/id_ed25519_work
```

---

#### 2. Tambahkan ke SSH config

Buat atau edit:

```text
~/.ssh/config
```

Isi seperti ini:

```text
# Personal
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal

# Work
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
```

---

#### 3. Tambahkan public key ke masing-masing akun GitHub

Masukkan isi file:

```
id_ed25519_personal.pub
```

ke akun GitHub pribadi.

Masukkan:

```
id_ed25519_work.pub
```

ke akun GitHub kantor.

---

#### 4. Clone repository menggunakan host yang sesuai

Repository pribadi:

```bash
git clone git@github-personal:username/repo.git
```

Repository kantor:

```bash
git clone git@github-work:company/repo.git
```

GitHub Desktop tetap bisa membuka repository tersebut melalui **File → Add Local Repository**.

---

### 5. Atur identitas Git per repository

Masuk ke folder project.

Untuk project kantor:

```bash
git config user.name "Nama Kantor"
git config user.email "email-kantor@company.com"
```

Untuk project pribadi:

```bash
git config user.name "Nama Pribadi"
git config user.email "email-pribadi@example.com"
```

Karena menggunakan `git config` tanpa `--global`, pengaturan ini hanya berlaku untuk repository tersebut.

Cek dengan:

```bash
git config --list
```

---

## Struktur yang saya rekomendasikan

Misalnya:

```
Projects/
│
├── Work/
│   ├── ProjectA
│   ├── ProjectB
│
└── Personal/
    ├── Portfolio
    ├── Blog
```

Setiap repository memiliki:

* `user.name` yang sesuai
* `user.email` yang sesuai
* Remote SSH sesuai akun (`github-work` atau `github-personal`)

Dengan cara ini Anda tidak perlu mengubah akun setiap kali membuka GitHub Desktop.

---

### Jika Anda menggunakan Windows

Kombinasi berikut biasanya paling nyaman:

* GitHub Desktop untuk antarmuka visual.
* Git for Windows dengan OpenSSH.
* Dua SSH key (`personal` dan `work`).
* Konfigurasi `~/.ssh/config`.
* `git config user.name` dan `git config user.email` per repository.

Konfigurasi ini membuat Anda dapat membuka proyek kantor dan pribadi secara bersamaan di GitHub Desktop tanpa perlu login/logout atau khawatir commit menggunakan identitas yang salah.


## Memasukan Key ke Akun Github
Tentu. Bagian ini adalah langkah untuk **mendaftarkan SSH public key ke akun GitHub** agar GitHub mengenali komputer Anda. Yang ditambahkan adalah **public key** (`.pub`), **bukan** private key.

Misalnya Anda membuat key seperti ini:

```bash
ssh-keygen -t ed25519 -C "email-pribadi@example.com"
```

dan menyimpannya sebagai:

```
~/.ssh/id_ed25519_personal
```

Maka akan ada dua file:

```
~/.ssh/id_ed25519_personal       <- Private key (JANGAN dibagikan)
~/.ssh/id_ed25519_personal.pub   <- Public key (ditambahkan ke GitHub)
```

## Langkah 1: Lihat isi file `.pub`

### Windows (PowerShell)

```powershell
Get-Content $HOME\.ssh\id_ed25519_personal.pub
```

atau

```powershell
type $HOME\.ssh\id_ed25519_personal.pub
```

### macOS / Linux

```bash
cat ~/.ssh/id_ed25519_personal.pub
```

Outputnya akan terlihat seperti ini (contoh):

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIK8L... email-pribadi@example.com
```

Salin **seluruh isi** baris tersebut.

---

## Langkah 2: Login ke akun GitHub pribadi

Masuk ke akun GitHub pribadi melalui browser.

Kemudian buka **Settings**.

---

## Langkah 3: Masuk ke menu SSH Keys

Di menu sebelah kiri pilih:

```
SSH and GPG keys
```

Lalu klik tombol:

```
New SSH key
```

---

## Langkah 4: Isi form

Contohnya:

**Title**

```
Laptop ASUS
```

atau

```
Office Laptop
```

Nama ini hanya sebagai penanda agar Anda tahu key tersebut berasal dari perangkat mana.

**Key type**

Biarkan **Authentication Key**.

**Key**

Paste seluruh isi file:

```
id_ed25519_personal.pub
```

yang tadi Anda salin.

Lalu klik **Add SSH key**.

GitHub mungkin akan meminta Anda memasukkan password atau kode verifikasi untuk konfirmasi.

---

## Langkah 5: Lakukan hal yang sama untuk akun kantor

Misalnya Anda punya:

```
~/.ssh/id_ed25519_work.pub
```

Jalankan:

```bash
cat ~/.ssh/id_ed25519_work.pub
```

atau di Windows:

```powershell
Get-Content $HOME\.ssh\id_ed25519_work.pub
```

Salin isinya.

Kemudian login ke **akun GitHub kantor**, buka **Settings → SSH and GPG keys → New SSH key**, lalu tempel isi `id_ed25519_work.pub`.

---

## Cara memastikan sudah berhasil

Jalankan:

```bash
ssh -T git@github-personal
```

Jika berhasil, Anda akan melihat pesan seperti:

```text
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

Untuk akun kantor:

```bash
ssh -T git@github-work
```

Outputnya akan serupa, tetapi dengan username akun kantor.

---




## Pudin Saepudin
