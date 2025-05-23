## VM Creation and Configuration

### Menginstal Cloudmonkey
```
snap install cloudmonkey
cloudmonkey --version
```

### Konfigurasi Cloudmonkey

```
cloudmonkey set url [IP Address:8080]/client/api
#example: cloudmonkey set url http://192.168.104.24:8080/client/api
cloudmonkey set apikey [apikey]
cloudmonkey set secretkey [secretkey]
cloudmonkey sync
```

API key and secret key generated in user page



### Membuat Instance
```
cloudmonkey list serverofferings | grep id
cloudmonkey list zones |grep id
cloudmonkey list templates templatefilter=all | grep id

cloudmonkey deploy virtualmachine templateid=[TEMPLATE-ID] serviceofferingid=[SERVICE-ID] zoneid=[ZONE-ID]

```

> ![image](https://hackmd.io/_uploads/S1XDO4iWel.png)

---

### Mengembangkan Virtual Machine di CloudStack dan Konfigurasi Tailscale Advertise Routes untuk Akses Jaringan Lokal

Menyebarkan Mesin Virtual di CloudStack dan Mengonfigurasi Rute Iklan Tailscale untuk Akses Jaringan Lokal

```bash
sudo tailscale up --advertise-routes=192.168.0.0/24 --operator=pc-2020-0018
```

> ![image](https://hackmd.io/_uploads/B15CtVo-le.png)

---

#### Mengakses Konsol VM dari GUI Apache CloudStack

Setelah menjalankan perintah Tailscale di atas, kamu sekarang bisa mengakses **Console/CLI** dari VM langsung melalui GUI Apache CloudStack.

> ![image](https://hackmd.io/_uploads/rJJc9VoZgl.png)
> ![image](https://hackmd.io/_uploads/ryCciVjbgx.png)

---

#### Mengaktifkan Akses Internet dari VM

Tambahkan **egress rule** berikut agar VM dapat mengakses internet:

> ![image](https://hackmd.io/_uploads/r1uLUSj-gx.png)

> ![image](https://hackmd.io/_uploads/ByVdUHjZxx.png)

* **Source CIDR:** `10.1.1.0/24`
* **Destination CIDR:** `0.0.0.0/0`
* **Protocol:** `All`
* **ICMP type / Start port:** `All`
* **ICMP code / End port:** `All`

> Pengaturan ini mengizinkan semua lalu lintas keluar dari jaringan sumber `10.1.1.0/24` menuju seluruh alamat tujuan di internet (`0.0.0.0/0` berarti semua alamat IP), untuk semua protokol, port, dan tipe ICMP.

> ![image](https://hackmd.io/_uploads/S1Ou3Nobxx.png)

---

#### Akses SSH ke VM

Untuk bisa melakukan **SSH** ke VM dari laptop kamu atau perangkat lain, kamu bisa menggunakan **port forwarding** seperti berikut:

> ![image](https://hackmd.io/_uploads/ryNhLSiWge.png)

> ![image](https://hackmd.io/_uploads/BkkJPBiZle.png)


* **Private port:** `22 - 22`
* **Public port:** `22 - 22`
* **Protocol:** `TCP`
* **VM:** `QVM-07ee70f9-755c-4031-9989-b4355dbe0147 (10.1.1.11)`

> ![image](https://hackmd.io/_uploads/B1TgDHiZeg.png)

> Pengaturan ini meneruskan koneksi SSH yang masuk pada port 22 di IP publik (gateway) ke port 22 milik IP privat VM, sehingga kamu dapat melakukan SSH ke VM secara remote dari mana saja.

> ![image](https://hackmd.io/_uploads/Bk6U1BjWlg.png)

> **Alternatif:** Jika laptop kamu yang digunakan untuk SSH berada di jaringan yang sama (misalnya melalui Tailscale yang sudah mengiklankan subnet lokal menggunakan `--advertise-routes`), maka kamu bisa langsung SSH ke IP privat VM **tanpa perlu port forwarding**.

> ![image](https://hackmd.io/_uploads/B1DjkBibgl.png)

---

#### Konfigurasi Firewall untuk SSH dan Ping
![image](https://hackmd.io/_uploads/ry-VvBjZlg.png)

Agar VM dapat menerima koneksi **SSH** dan juga dapat **melakukan dan menerima ping (ICMP)**, atur **firewall** seperti berikut:

1. Untuk SSH:

   * **Source CIDR:** `0.0.0.0/0`
   * **Protocol:** `TCP`
   * **Start port:** `22`
   * **End port:** `22`

2. Untuk ICMP (ping):

   * **Source CIDR:** `0.0.0.0/0`
   * **Protocol:** `ICMP`
   * **ICMP type:** `-1` (semua tipe)
   * **ICMP code:** `-1` (semua kode)

---

> Pengaturan ini memungkinkan semua sumber IP melakukan SSH ke VM dan juga mengizinkan semua jenis komunikasi ICMP (termasuk ping), baik masuk maupun keluar, tergantung arah trafik dan kebijakan lainnya.

> Namun, perlu diperhatikan bahwa **tidak semua perangkat bisa langsung melakukan ping ke VM**, meskipun aturan firewall telah mengizinkan ICMP. Agar ping (dan koneksi lainnya) berhasil, ada beberapa syarat yang harus dipenuhi:

> 1. **Perangkat harus berada dalam jaringan yang sama dengan VM**, contohnya berada dalam subnet privat yang sama seperti `10.1.1.0/24`, atau memiliki jalur routing yang mengarah ke subnet tersebut.

> 2. Jika tidak berada di jaringan yang sama, maka perangkat harus **terhubung melalui jaringan overlay seperti Tailscale**, dan:
>
>    * VM berada dalam subnet yang di-*advertise* oleh perangkat (laptop/server) yang menjalankan Tailscale dengan opsi `--advertise-routes`.
>    * Perangkat yang ingin melakukan ping harus juga berada di jaringan Tailscale dan memiliki akses ke subnet tersebut melalui mekanisme routing internal Tailscale.

> 3. **Firewall CloudStack** dan **ACL di Tailscale** juga harus mengizinkan trafik ICMP atau SSH. Walaupun aturan di CloudStack sudah mengizinkan, jika Tailscale atau perangkat jaringan lain (misalnya firewall lokal atau VPN rules) memblokir ICMP atau TCP/22, maka koneksi tetap akan gagal.

> Dengan kata lain, walaupun konfigurasi firewall di CloudStack sudah dibuat permisif (semua IP, semua port/protokol), tetap saja konektivitas tidak akan berhasil **kecuali** perangkat:
>
> * memiliki jalur routing yang valid ke alamat IP privat VM, dan
> * tidak dibatasi oleh konfigurasi firewall lain di luar CloudStack.

> ![image](https://hackmd.io/_uploads/Sy-JXrsZge.png)

>![image](https://hackmd.io/_uploads/H12zSroZxg.png)

---