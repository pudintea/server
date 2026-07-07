# Catatan KOnfigurasi Server Oracle
Per Juli 2026, Oracle telah mengurangi alokasi Always Free Ampere A1. Sebelumnya banyak orang mengenal jatah gratisnya sebagai 4 OCPU + 24 GB RAM,
tetapi dokumentasi resmi Oracle sekarang menunjukkan batas baru 2 OCPU + 12 GB RAM.

## Dokumentasi
* [Daftar dan Create Instance](https://www.youtube.com/watch?v=TAZfDdQha3U&t=364s)
* [Firewall HTTP/HTTPS](https://www.youtube.com/watch?v=8avXWgUz6ac)

## MEmbuat VCN Sebelum Install Server/Instance
Menu > Networking > Overview > Start VCN Wizard > Isi pada VCN Name > Biarkan semuanya devault > Create > View VCN

## Create Server/Instance VPS
Menu > Compute > Instance > Create Instance Pilih OS, atur Ram dan CPU atur storage download sshkey Create

## Cara Buka Firewall Port 80/443
Compute > Instance > Klik Name Server > Networking > Subnet > Security > Name (Default) > Security Rules
```
No
0.0.0.0/0
TCP
All
80
TCP traffic for ports: 80
HTTP
```
```
No
0.0.0.0/0
TCP
All
443
TCP traffic for ports: 443 HTTPS
HTTPS
```

## IP Tables Port 80/443
Buka config IPtables
```
sudo nano /etc/iptables/rules.v4
```
Silahkan sesuaikan kode nya :
```
# CLOUD_IMG: This file was created/modified by the Cloud Image build process
# iptables configuration for Oracle Cloud Infrastructure

# See the Oracle-Provided Images section in the Oracle Cloud Infrastructure
# documentation for security impact of modifying or removing these rule

*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [463:49013]
:InstanceServices - [0:0]
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
```
Jangan lupa terapkan konfigurasi yg sudah kita ubah 
```
sudo iptables-restore < /etc/iptables/rules.v4
```

## Hubungi Saya
* Pudin Saepudin
* [Telegram](https://t.me/pudin_ira)
