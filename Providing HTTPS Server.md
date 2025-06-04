### Providing HTTP/HTTPS Server
[Link YouTube]() 
---

#### Aktifkan MagicDNS dan HTTPS di Konsol Admin
Buka Tailscale Admin Console, kemudian pergi ke tab 'DNS'.

![image](https://hackmd.io/_uploads/B16vKUoWxe.png)

Aktifkan MagicDNS jika belum diaktifkan. MagicDNS memungkinkan penamaan perangkat yang lebih mudah di jaringan Tailscale.

![image](https://hackmd.io/_uploads/ry6CK8iWlg.png)

Di bagian HTTPS Certificates, klik Enable HTTPS. Nantinya akan diminta untuk mengonfirmasi bahwa nama perangkat dan tailnet akan dipublikasikan di ledger publik.

![image](https://hackmd.io/_uploads/Sy-VcIs-lg.png)

--- 

#### Pilih Mesin untuk Mengonfigurasi HTTPS
Kembali buka Tailscale Admin Console. Pilih perangkat (machine) yang ingin diatur untuk HTTPS. Pilih alamat perangkat yang akan digunakan, contohnya: `2020-pc-0018.taila69d63.ts.net`. Nama di belakang `.ts.net` adalah nama Tailscale perangkat, yang berfungsi sebagai alamat unik dalam jaringan Tailscale.
![image](https://hackmd.io/_uploads/H13_yvjbxg.png)

--- 

#### Akses Remote untuk Pengaturan HTTPS
1. Akses server melalui SSH:
`ssh root@2020-pc-0018.taila69d63.ts.net`

2. Jalankan perintah berikut untuk menghasilkan sertifikat TLS:
`tailscale cert <hostname>ts.net` contohnya
`tailscale cert 2020-pc-0018.taila69d63.ts.net`
dan mendapatkan output:
``` c
Wrote public cert to 2020-pc-0018.taila69d63.ts.net.crt
Wrote private key to 2020-pc-0018.taila69d63.ts.net.key
```
> ✅ Tailscale telah berhasil membuat file sertifikat HTTPS (public key) dan private key untuk hostname `2020-pc-0018.taila69d63.ts.net`.

Secara default, file tersebut ditulis ke lokasi direktori tempat dijalankannya perintah. Maka file-nya berada di:
`/root/2020-pc-0018.taila69d63.ts.net.crt`
`/root/2020-pc-0018.taila69d63.ts.net.key`

3. Pastikan modul Apache yang dibutuhkan sudah aktif:
``` c
sudo a2enmod proxy proxy_http ssl headers
sudo systemctl restart apache2
```
4. Pasang sertifikat Tailscale ke folder yang mudah diakses Apache:
``` c
sudo mkdir -p /etc/apache2/ssl/
sudo cp /root/2020-pc-0018.taila69d63.ts.net.crt /etc/apache2/ssl/
sudo cp /root/2020-pc-0018.taila69d63.ts.net.key /etc/apache2/ssl/
sudo chmod 600 /etc/apache2/ssl/*
```
5. Buat konfigurasi virtual host HTTPS baru di Apache:
``` c
sudo nano /etc/apache2/sites-available/cloudstack-ssl.conf
```
Isi file tersebut dengan:
``` c
<VirtualHost *:443>
 ServerName 2020-pc-0018.taila69d63.ts.net
 SSLEngine on
 SSLCertificateFile /etc/apache2/ssl/2020-pc-0018.taila69d63.ts.net.crt
 SSLCertificateKeyFile /etc/apache2/ssl/2020-pc-0018.taila69d63.ts.net.key
 ProxyPreserveHost On
 ProxyRequests Off
 ProxyPass / http://localhost:8080/
 ProxyPassReverse / http://localhost:8080/
 # Optional: supaya header aman dan tidak caching
 Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
 Header always set X-Frame-Options DENY
 Header always set X-Content-Type-Options nosniff
 ErrorLog ${APACHE_LOG_DIR}/cloudstack_ssl_error.log
 CustomLog ${APACHE_LOG_DIR}/cloudstack_ssl_access.log combined
</VirtualHost>
```

6.  Aktifkan konfigurasi dan SSL:
``` c 
sudo a2ensite cloudstack-ssl.conf
sudo systemctl reload apache2
```

7. Pastikan firewall izinkan port 443:
``` c
sudo ufw allow 443
```
---

#### Sekarang akses
![image](https://hackmd.io/_uploads/rJHruPsbeg.png)
