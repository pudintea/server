Pertanyaan **sangat bagus** 👍
Ini perubahan **aman** dan **umum dilakukan**, tapi memang ada **beberapa titik WAJIB diubah** supaya CI/CD, Nginx, dan CI4 tetap jalan normal.

Saya jelaskan **konsep dulu**, lalu **daftar perubahan satu-per-satu**, lalu **langkah praktisnya**.

---

# 🎯 TUJUAN PERUBAHAN

Dari:

```
/var/www/app-dev
/var/www/app-prod
```

Menjadi:

```
/home/users/app/app-dev
/home/users/app/app-prod
```

Artinya:

* Lokasi code berubah
* Semua komponen yang “menunjuk” ke path lama **HARUS diperbarui**

---

# 🧠 KOMPONEN YANG TERDAMPAK

| Komponen                    | Perlu diubah?           |
| --------------------------- | ----------------------- |
| Folder aplikasi             | ✅                       |
| Nginx config                | ✅                       |
| GitHub Actions (rsync path) | ✅                       |
| Permission user             | ✅                       |
| Cloudflare Tunnel           | ❌                       |
| CodeIgniter 4               | ❌ (asal `public` benar) |

---

# ✅ LANGKAH TERPERINCI (IKUTI URUTAN)

---

## 1️⃣ Buat Folder Baru

```bash
mkdir -p /home/users/app/app-dev
mkdir -p /home/users/app/app-prod
```

Set ownership:

```bash
chown -R www-data:www-data /home/users/app
```

---

## 2️⃣ Pindahkan Data Lama (Jika Sudah Ada)

```bash
mv /var/www/app-dev/* /home/users/app/app-dev/
mv /var/www/app-prod/* /home/users/app/app-prod/
```

(atau `rsync -av` kalau ingin aman)

---

## 3️⃣ UBAH CONFIG NGINX (WAJIB)

### Sebelumnya:

```nginx
root /var/www/app-dev/public;
```

### Menjadi:

```nginx
root /home/users/app/app-dev/public;
```

Dan jika ada prod:

```nginx
root /home/users/app/app-prod/public;
```

📌 **HANYA root yang diubah**, yang lain tetap.

---

## 4️⃣ Reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 5️⃣ UBAH PATH DI GITHUB ACTIONS (WAJIB)

### Workflow sebelumnya:

```yaml
rsync -av --delete ./ /var/www/app-prod
```

### Ubah menjadi:

```yaml
rsync -av --delete ./ /home/users/app/app-prod
```

Dan untuk dev:

```yaml
/home/users/app/app-dev
```

---

## 6️⃣ UBAH PATH DI STEP COMPOSER (Jika Ada)

Jika workflow ada:

```yaml
cd /var/www/app-prod
```

Ganti jadi:

```yaml
cd /home/users/app/app-prod
```

---

## 7️⃣ Permission Folder Writable (ULANGI)

```bash
chmod -R 775 /home/users/app/app-prod/writable
chmod -R 775 /home/users/app/app-dev/writable
```

---

## 8️⃣ Test CI4

```bash
cd /home/users/app/app-prod
php spark
```

Jika muncul CLI → aman.

---

# ❗ HAL YANG TIDAK PERLU DIUBAH

❌ Cloudflare Tunnel
❌ `.env` (selama baseURL tetap sama)
❌ Config CI4 lainnya
❌ PHP-FPM socket

---

# 🧪 CHECKLIST CEPAT

```bash
ls /home/users/app/app-prod/public/index.php
```

Jika ada → OK

---

# 🧠 RINGKASAN SUPER SINGKAT

1. Buat folder baru
2. Pindahkan data
3. Ubah `root` Nginx
4. Update path di GitHub Actions
5. Permission ulang
6. Reload Nginx

---

## 🔐 TIPS KEAMANAN (OPSIONAL TAPI DISARANKAN)

Jika mau lebih rapi:

```bash
chown -R appuser:www-data /home/users/app
chmod -R 750 /home/users/app
```

---

## 🔜 Mau Saya Bantu?

Saya bisa:

* Review workflow YAML terbaru
* Buat struktur multi-user yang rapi
* Pisahkan DEV & PROD tunnel Cloudflare
* Setup rollback directory

Tinggal bilang 👍
