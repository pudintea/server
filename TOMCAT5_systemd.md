Java Anda sudah cocok untuk Apache Tomcat lama — OpenJDK 8 biasanya masih bisa jalan.

Berarti tinggal pastikan `JAVA_HOME` benar.

Di Ubuntu 20 dengan Java Anda, biasanya:

```bash id="vt57n4"
readlink -f $(which java)
```

hasilnya kira-kira:

```text id="12doh7"
/usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java
```

Maka `JAVA_HOME` yang benar adalah:

```bash id="e4utpi"
/usr/lib/jvm/java-8-openjdk-amd64
```

Jadi file service tadi sudah sesuai.

---

Supaya lebih stabil, saya sarankan pakai versi service ini:

```bash id="7z2p2x"
sudo nano /etc/systemd/system/tomcat5.service
```

Isi:

```ini id="m1c2kr"
[Unit]
Description=Apache Tomcat 5
After=network.target

[Service]
Type=forking

User=root
Group=root

Environment="JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64"
Environment="CATALINA_HOME=/usr/share/tomcat5"
Environment="CATALINA_BASE=/usr/share/tomcat5"
Environment="CATALINA_PID=/usr/share/tomcat5/temp/tomcat.pid"

ExecStart=/usr/share/tomcat5/bin/startup.sh
ExecStop=/usr/share/tomcat5/bin/shutdown.sh

Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Lalu jalankan:

```bash id="qf4zsu"
sudo systemctl daemon-reload
sudo systemctl enable tomcat5
sudo systemctl start tomcat5
```

Cek:

```bash id="s8wdj6"
sudo systemctl status tomcat5
```

Kalau muncul:

```text id="mgo6fh"
active (running)
```

berarti sudah otomatis start saat reboot.

Tes dengan:

```bash id="k0h25k"
sudo reboot
```

Setelah server hidup lagi:

```bash id="xx0y6v"
sudo systemctl status tomcat5
```

Jika ada error setelah reboot, kirim hasil:

```bash id="5g4l9t"
sudo systemctl status tomcat5
```

dan:

```bash id="v6f1q4"
journalctl -u tomcat5 -n 50
```

nanti saya bantu analisa.
