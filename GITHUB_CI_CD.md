Baik, kasus Anda **sangat umum di perusahaan** 👍
Intinya: **server tidak public**, berada di **jaringan lokal kantor**, dan ingin **CI/CD dari GitHub** dengan **branch dev & main**.

Saya akan jelaskan **pelan-pelan, bertahap, dari konsep → arsitektur → langkah teknis**, tanpa lompat.

---

## 1️⃣ Gambaran Besar Solusi (Supaya Tidak Bingung)

Karena **server Anda tidak bisa diakses dari internet**, maka:

❌ **GitHub tidak bisa “push” ke server Anda**
✅ **Server Anda yang “menarik” instruksi dari GitHub**

### Metode PALING TEPAT untuk kondisi ini:

👉 **GitHub Actions + Self-Hosted Runner**

> Artinya:
>
> * Server lokal Anda menjalankan agent GitHub
> * Saat ada push ke GitHub, server Anda **sendiri yang mengeksekusi CI/CD**

Ini **AMAN**, **STANDAR INDUSTRI**, dan **tanpa expose server ke publik**.

---

## 2️⃣ Arsitektur CI/CD yang Akan Kita Bangun

```
Developer Push Code
        │
        ▼
     GitHub Repo
    (branch dev / main)
        │
        ▼
 GitHub Actions Workflow
        │
        ▼
Self-Hosted Runner (di Server Lokal)
        │
        ▼
Deploy ke:
- Folder dev (branch dev)
- Folder production (branch main)
```

---

## 3️⃣ Struktur Branch & Folder di Server

### Di GitHub

* `dev` → untuk developer
* `main` → untuk production

### Di Server Lokal

Misalnya:

```bash
/var/www/
├── app-dev/        # hasil deploy branch dev
└── app-prod/       # hasil deploy branch main
```

---

## 4️⃣ Tahap 1 — Persiapan Server Lokal

### 4.1 OS & Tools Wajib

Pastikan server memiliki:

* Linux (Ubuntu/CentOS)
* `git`
* `curl`
* `bash`

```bash
sudo apt update
sudo apt install git curl -y
```

---

## 5️⃣ Tahap 2 — Setup Self-Hosted Runner GitHub

### 5.1 Masuk ke GitHub Repository

1. Buka repo Anda
2. **Settings → Actions → Runners**
3. Klik **New self-hosted runner**
4. Pilih OS (Linux)

GitHub akan menampilkan **command khusus** → JANGAN DISALIN SEMBARANGAN
Jalankan **langsung di server lokal**

---

### 5.2 Install Runner di Server

Contoh (kurang lebih seperti ini):

```bash
mkdir actions-runner
cd actions-runner

curl -o actions-runner-linux-x64.tar.gz -L https://github.com/actions/runner/releases/download/...
tar xzf actions-runner-linux-x64.tar.gz
```

Lalu konfigurasi:

```bash
./config.sh --url https://github.com/USERNAME/REPO --token TOKEN_DARI_GITHUB
```

Jalankan runner:

```bash
./run.sh
```

👉 Jika sukses, server Anda **akan muncul di GitHub sebagai Runner ACTIVE**

---

### 5.3 Jadikan Runner Service (Auto Start)

Supaya tidak mati saat reboot:

```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

---

## 6️⃣ Tahap 3 — Siapkan Folder Deploy di Server

```bash
mkdir -p /var/www/app-dev
mkdir -p /var/www/app-prod

chown -R $USER:$USER /var/www
```

---

## 7️⃣ Tahap 4 — Workflow CI/CD GitHub Actions

Di repo GitHub buat file:

```
.github/workflows/deploy.yml
```

---

## 8️⃣ Contoh Workflow CI/CD (Sederhana & Aman)

```yaml
name: CI/CD Local Server

on:
  push:
    branches:
      - dev
      - main

jobs:
  deploy:
    runs-on: self-hosted

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to DEV
        if: github.ref == 'refs/heads/dev'
        run: |
          rsync -av --delete ./ /var/www/app-dev

      - name: Deploy to PRODUCTION
        if: github.ref == 'refs/heads/main'
        run: |
          rsync -av --delete ./ /var/www/app-prod
```

📌 Artinya:

* Push ke `dev` → deploy ke `app-dev`
* Push ke `main` → deploy ke `app-prod`

---

## 9️⃣ Tahap 5 — Alur Kerja Developer (Best Practice)

### Developer

```bash
git checkout dev
git commit -m "fitur baru"
git push origin dev
```

➡️ Otomatis deploy ke **DEV**

---

### Release ke Production

```bash
git checkout main
git merge dev
git push origin main
```

➡️ Otomatis deploy ke **PRODUCTION**

---

## 🔐 Keamanan (Penting)

✅ Server **tidak perlu IP publik**
✅ Tidak perlu SSH dari GitHub
✅ Token hanya tersimpan lokal
✅ Bisa pakai user non-root

---

## 🧠 Alternatif (Jika Tidak Mau GitHub Actions)

| Metode          | Cocok? | Catatan         |
| --------------- | ------ | --------------- |
| Jenkins Lokal   | ✅      | Lebih berat     |
| GitLab CI       | ❌      | Repo di GitHub  |
| Cron + Git Pull | ⚠️     | Kurang aman     |
| SSH dari GitHub | ❌      | Butuh public IP |

👉 **Self-Hosted Runner adalah pilihan TERBAIK**

---

## 10️⃣ Kalau Mau Lebih Lanjut (Next Level)

Nanti bisa dikembangkan:

* Docker build & deploy
* Testing otomatis
* Approval sebelum production
* Rollback otomatis

---

## ❓Pertanyaan Lanjutan (Opsional)

Agar saya bisa menyesuaikan contoh:

1. Aplikasi Anda **apa**? (Laravel, NodeJS, PHP biasa, Java?)
2. Server **Linux apa**?
3. Ingin **pakai Docker atau tidak**?

Kalau mau, saya bisa buatkan **workflow YAML khusus sesuai aplikasi Anda** 🚀
