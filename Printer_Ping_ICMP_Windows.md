Di Windows 10 dan Windows 11, **Windows Defender Firewall** secara default memblokir beberapa koneksi masuk, termasuk **ICMP (ping)** dan **File and Printer Sharing**. Kalau Anda ingin membolehkan **ping** dan **printer sharing**, ada dua cara: melalui **Control Panel (GUI)** dan melalui **Command Prompt / PowerShell**.

---

## 1. Allow Ping (ICMP Echo Request)

### Cara GUI (Windows Defender Firewall):

1. Tekan `Win + R`, ketik `wf.msc`, lalu tekan **Enter** → ini akan membuka *Windows Defender Firewall with Advanced Security*.
2. Klik **Inbound Rules** (aturan koneksi masuk).
3. Cari rule bernama:

   * **File and Printer Sharing (Echo Request - ICMPv4-In)**
   * (Jika perlu juga aktifkan yang ICMPv6).
4. Klik kanan rule tersebut → pilih **Enable Rule**.

Sekarang komputer sudah bisa menerima ping.

### Cara Command Line (lebih cepat):

Buka **Command Prompt (Run as Administrator)**, lalu jalankan:

```cmd
netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=Yes
```

Atau kalau hanya untuk ping (ICMPv4):

```cmd
netsh advfirewall firewall add rule name="ICMPv4 Allow Incoming Echo Request" protocol=icmpv4:8,any dir=in action=allow
```

---

## 2. Allow Printer Sharing (File and Printer Sharing)

### Cara GUI:

1. Masuk ke **Control Panel** → **Network and Sharing Center** → **Advanced sharing settings**.
2. Pastikan opsi **Turn on file and printer sharing** aktif pada profil jaringan yang digunakan (Private/Public/Domain).
3. Lalu, kembali ke **Windows Defender Firewall with Advanced Security (wf.msc)**.
4. Cari rule **File and Printer Sharing (SMB-In)** dan **File and Printer Sharing (Spooler Service – RPC)**.
5. Klik kanan → **Enable Rule**.

### Cara Command Line:

Di **CMD (Admin)** jalankan:

```cmd
netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=Yes
```

Ini akan mengaktifkan semua rule yang terkait dengan printer sharing.

---

## Ringkasan:

* **Ping** → aktifkan rule **Echo Request ICMPv4-In**.
* **Printer Sharing** → aktifkan rule di grup **File and Printer Sharing**.
* Bisa dilakukan lewat GUI (`wf.msc`) atau cepat via CMD/PowerShell dengan `netsh`.

-----
Pudin Saepudin : italazhar.com
