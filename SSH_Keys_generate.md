Menggunakan SSH Keys memungkinkan kita masuk ke server tanpa menggunakan password, panduan ini akan menjelaskan bagaimana cara membuat SSK key pada linux dan mac dalam format OpenSSH.

Generate SSH keys dengan OpenSSH
Paket OpenSSH secara default sudah terinstall pada sistem operasi linux dan mac, ikuti langkah-langkah di bawah ini untuk membuat SSH keys dengan menggunakan perkakas OpenSSH.


<h2>1. Generate SSH Key</h2>
<pre>
  <code>$ ssh-keygen -t rsa -b 4096 -C "Sysadmin"</code>
</pre>
<ul>
  <li>parameter  "-t rsa" akan menghasilkan kunci dengan format RSA</li>
  <li>parameter "-b 4096" akan menghasilkan panjang kunci 4096-bit</li>
  <li>parameter "-C [comment]" digunakan untuk membuat deskripsi</li>
</ul>

<h2>2. Kemudian tekan Enter untuk menyimpan kunci. Secara default kunci akan tersimpan pada direktori ~/.ssh dengan nama id_rsa sebagai private key dan id_rsa.pub sebagai public key.</h2>

Generating public/private rsa key pair.
Enter file in which to save the key (/home/sysadmin/.ssh/id_rsa):
<h2>3. Selanjutnya akan muncul permintaan memasukkan password untuk SSH keys kita, kamu bisa melewati step ini dengan menekan tombol enter</h2>

Enter passphrase (empty for no passphrase):
Enter same passphrase again:
<h2>4. SSH keys sudah di hasilkan dan tersimpan.</h2>

Your identification has been saved in /home/example_user/.ssh/id_rsa.
Your public key has been saved in /home/sysadmin/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Ku7dQu5GcqB8/9UXwhjYGXJrFSrb1jJJTEBwpauz/p0 Sysadmin
The key's randomart image is:
+---[RSA 4096]----+
|      .o=o= o.   |
|       . O *     |
|    . . + X      |
| . . = o * *     |
|  o + O S O + .  |
|   o O o * + . . |
|    o + . . . .  |
|     o o o   .   |
|      . E        |
+----[SHA256]-----+

<p>Selesai...</p>
