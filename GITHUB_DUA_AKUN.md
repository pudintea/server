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
