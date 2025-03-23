Masalah yang kamu hadapi terjadi karena server DNS yang kamu gunakan (192.168.3.3) belum dikonfigurasi dengan benar untuk memberikan informasi nama hostnya sendiri. Biasanya, ini terjadi karena ada yang kurang dalam konfigurasi **reverse DNS lookup** atau **hostname-nya tidak terdaftar dengan benar** di file konfigurasi BIND9.

---

### **Solusi yang Bisa Dicoba**:

#### **1. Pastikan File `/etc/hosts` Sudah Dikonfigurasi dengan Benar**
Cek apakah IP server DNS sudah memiliki hostname yang sesuai. Jalankan perintah berikut di server Ubuntu yang menjalankan BIND9:

```bash
cat /etc/hosts
```
Pastikan ada baris seperti ini:
```
192.168.3.3    dns-server.local
```

Jika belum ada, tambahkan baris tersebut, lalu simpan.

---

#### **2. Pastikan Konfigurasi Zona Reverse DNS Sudah Ada**
Untuk memastikan bahwa BIND9 bisa merespons permintaan reverse lookup (`PTR` record), kamu harus memiliki file konfigurasi zona reverse DNS. 

- Cek apakah ada konfigurasi zona reverse di file BIND9:

```bash
cat /etc/bind/named.conf.local
```

Jika belum ada, tambahkan konfigurasi berikut:

```bash
zone "3.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192";
};
```
> *Sesuaikan dengan jaringanmu. Jika IP server DNS-mu `192.168.3.3`, maka format zona reverse adalah `3.168.192.in-addr.arpa`.*

---

#### **3. Buat File Konfigurasi Reverse Lookup**
Buat file baru `/etc/bind/db.192` dan isi dengan:

```bash
$TTL 604800
@       IN      SOA     dns-server.local. root.dns-server.local. (
                        2         ; Serial
                        604800    ; Refresh
                        86400     ; Retry
                        2419200   ; Expire
                        604800 )  ; Negative Cache TTL
;

@       IN      NS      dns-server.local.
3       IN      PTR     dns-server.local.
```
> *Pastikan hostname-nya sesuai dengan yang ada di `/etc/hosts`.*

Simpan lalu restart BIND9:
```bash
sudo systemctl restart bind9
```

---

#### **4. Tes Konfigurasi**
Coba jalankan perintah ini dari klien:
```bash
nslookup 192.168.3.3
```
Jika konfigurasi berhasil, seharusnya output menunjukkan hostname yang benar.

---

#### **5. Pastikan Forwarders Sudah Dikonfigurasi dengan Benar (Opsional)**
Jika kamu ingin BIND9 bisa meneruskan permintaan ke DNS publik (misalnya Google DNS atau Cloudflare), pastikan di file `/etc/bind/named.conf.options` ada bagian berikut:

```bash
options {
    directory "/var/cache/bind";
    
    forwarders {
        8.8.8.8;
        8.8.4.4;
    };

    allow-query { any; };
    recursion yes;
};
```
Simpan, lalu restart:
```bash
sudo systemctl restart bind9
```

---

### **Kesimpulan**
Masalah **"Server: UnKnown"** biasanya terjadi karena BIND9 tidak memiliki informasi **reverse DNS lookup** (`PTR` record). Dengan menambahkan zona reverse dan memastikan hostname sudah dikonfigurasi dengan benar, masalah ini bisa teratasi. 

Coba langkah-langkah di atas dan beri tahu saya hasilnya! 🚀
