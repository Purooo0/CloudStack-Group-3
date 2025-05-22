## Cloudstack Installation (Controller and Compute Node at the Same Host)

### Mengimpor Kunci Repositori Cloudstack
```
sudo -i
mkdir -p /etc/apt/keyrings 
wget -O- http://packages.shapeblue.com/release.asc | gpg --dearmor | sudo tee /etc/apt/keyrings/cloudstack.gpg > /dev/null
echo deb [signed-by=/etc/apt/keyrings/cloudstack.gpg] http://packages.shapeblue.com/cloudstack/upstream/debian/4.18 / > /etc/apt/sources.list.d/cloudstack.list
```

> Baris pertama digunakan untuk membuat direktori tempat menyimpan public key milik CloudStack.
> `wget -O` digunakan untuk mengunduh URL yang diberikan dan mengarahkan output-nya ke perintah `gpg --dearmor`.
> Perintah `gpg --dearmor` akan mengubah format ASCII armored menjadi format biner.
> Perintah `sudo tee` akan mengarahkan hasil dari `gpg --dearmor` ke file `/etc/apt/keyrings/cloudstack.gpg`.

### Periksa Repositori yang Ditambahkan
```
nano /etc/apt/sources.list.d/cloudstack.list
```
> make sure you see this line at the last line
> ![image](https://hackmd.io/_uploads/HyZ0BQsZel.png)

### Menginstal Cloudstack dan Server Mysql

```
apt-get update -y
apt-get install cloudstack-management mysql-server
```
> Note : Proses ini lumayan lama


### Konfigurasi MySQL
#### Buka File Konfigurasi MySQL

```
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

#### Salin Baris Ini di Bawah Bagian [mysqld]

```
server-id = 1
sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_ENGINE_SUBSTITUTION"
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=1000
log-bin=mysql-bin
binlog-format = 'ROW'
```

#### Mulai Ulang Layanan MySQL
```
systemctl restart mysql
```

#### Periksa Status Layanan MySQL
```
systemctl status mysql
```
> ![image](https://hackmd.io/_uploads/HkmdPQiWxl.png)

### Deploy Database Sebagai Root dan Kemudian Buat Pengguna "cloud" dengan Kata Sandi "cloud" Juga. Pastikan Kata Sandi Akun Root Laptop Anda Cocok

**To change your laptop's root account, insert this command**

```
sudo passwd root
# It will then ask for a password, in this case we will use, so type it in Pa$$w0rd
# It will then ask you to re-enter the password again.
```

```
cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:Pa$$w0rd -i 192.168.104.24
```

### Konfigurasikan Penyimpanan Primer dan Sekunder
```
apt-get install nfs-kernel-server quota
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
mkdir -p /export/primary /export/secondary
exportfs -a
```

### Konfigurasi Server NFS 
```
sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server
sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common
echo "NEED_STATD=yes" >> /etc/default/nfs-common
sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota
service nfs-kernel-server restart
```

**Penjelasan:**

> `sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server`
> Perintah ini menggunakan `sed` (stream editor) untuk mencari baris `RPCMOUNTDOPTS="--manage-gids"` dalam file `/etc/default/nfs-kernel-server` dan menggantinya dengan `RPCMOUNTDOPTS="-p 892 --manage-gids"`. Opsi `-i` digunakan agar perubahan langsung diterapkan ke file asli.

> `sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common`
> Perintah ini mencari baris `STATDOPTS=` dalam file `/etc/default/nfs-common` dan menggantinya dengan `STATDOPTS="--port 662 --outgoing-port 2020"`.

> `echo "NEED_STATD=yes" >> /etc/default/nfs-common`
> Perintah ini menambahkan baris `NEED_STATD=yes` ke akhir file `/etc/default/nfs-common`. Tanda `>>` digunakan untuk menambahkan (append) teks ke file.

> `sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota`
> Perintah ini mencari baris `RPCRQUOTADOPTS=` dalam file `/etc/default/quota` dan menggantinya dengan `RPCRQUOTADOPTS="-p 875"`.

> `service nfs-kernel-server restart`
> Perintah ini digunakan untuk me-restart layanan NFS (Network File System).

---
## Configure Cloudstack Host with KVM Hypervisor

### Instal KVM dan Cloudstack Agent
```
apt-get install qemu-kvm cloudstack-agent -y
```

### Konfigurasi KVM Virtualization Management
Ubah beberapa line
```
sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf

# On Ubuntu 22.04, add LIBVIRTD_ARGS="--listen" to /etc/default/libvirtd instead.

sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd
```

**Penjelasan:**

> `sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf`
> Perintah ini menggunakan `sed` (stream editor) untuk mencari baris dalam file `/etc/libvirt/qemu.conf` yang dimulai dengan `#vnc_listen` dan menggantinya dengan `vnc_listen = "0.0.0.0"`. Opsi `-i` berarti perubahan dilakukan langsung di file aslinya. Alamat IP `0.0.0.0` merupakan alamat khusus yang digunakan untuk menunjukkan bahwa layanan akan menerima koneksi dari semua alamat IP yang tersedia pada mesin lokal.

> `sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd`
> Perintah ini menggunakan `sed` untuk mencari baris yang dimulai dengan `LIBVIRTD_ARGS=` dalam file `/etc/default/libvirtd` dan menggantinya dengan `LIBVIRTD_ARGS="--listen"`. Opsi `-i.bak` berarti file asli akan diedit langsung dan salinannya akan disimpan sebagai file backup dengan ekstensi `.bak`.

#### Tambahkan Beberapa Baris
```
echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
```

**Penjelasan:**

```
echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
# menonaktifkan pendengaran (listening) melalui TLS pada daemon libvirt

echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
# mengaktifkan pendengaran (listening) melalui TCP pada daemon libvirt

echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
# menentukan port yang akan digunakan daemon untuk menerima koneksi TCP

echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
# menonaktifkan multicast DNS, sehingga layanan libvirtd tidak akan ditemukan melalui mDNS

echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
# menonaktifkan autentikasi untuk koneksi TCP ke daemon libvirt
```

#### Restart libvirtd

```
systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket
systemctl restart libvirtd
```

**Penjelasan Perintah:**

> `systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket`
> Perintah ini menggunakan `systemctl`, yaitu manajer sistem dan layanan di Linux, untuk *masking* beberapa unit socket yang berkaitan dengan layanan `libvirtd`. Melakukan *mask* pada unit di `systemd` berarti menonaktifkannya sepenuhnya, sehingga unit tersebut tidak bisa dijalankan secara manual maupun otomatis oleh layanan lain. Dalam kasus ini, perintah tersebut menonaktifkan beberapa socket yang digunakan `libvirtd` untuk berkomunikasi dengan proses lain. Hal ini mungkin dilakukan untuk mencegah `libvirtd` menerima koneksi melalui socket tersebut.

> `systemctl restart libvirtd`
> Perintah ini digunakan untuk me-*restart* layanan `libvirtd`. Ini biasanya dilakukan setelah melakukan perubahan pada konfigurasi layanan atau unit terkait (seperti socket), agar perubahan tersebut bisa diterapkan.
 
>`libvirtd` adalah daemon yang menyediakan manajemen mesin virtual (VM), jaringan virtual, dan penyimpanan untuk berbagai teknologi virtualisasi seperti KVM, QEMU, Xen, dan lainnya. Daemon ini merupakan bagian dari proyek libvirt yang menyediakan antarmuka API yang konsisten dan aman untuk mengelola platform virtualisasi, tanpa bergantung pada teknologi virtualisasi yang digunakan.

### Konfigurasi untuk Mendukung Docker dan Layanan Lainnya
```
echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
sysctl -p
```

Explanation
> ARP and IP packet will not processed by arptable and iptable

### Hasilkan Unique Host ID

```
apt-get install uuid -y
UUID=$(uuid)
echo host_uuid = \"$UUID\" >> /etc/libvirt/libvirtd.conf
systemctl restart libvirtd
```

### Konfigurasikan Firewall Iptables dan Jadikan Persisten

```
NETWORK=192.168.0.0/24
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 111 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 111 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 2049 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 32803 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 32769 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 892 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 875 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 662 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8250 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8080 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8443 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 9090 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 16514 -j ACCEPT
#iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 3128 -j ACCEPT
#iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 3128 -j ACCEPT

apt-get install iptables-persistent
#jawab saja yes yes
```

Langkah ini memastikan semua port layanan yang digunakan oleh CloudStack tidak diblokir oleh firewall dan dapat diakses oleh jaringan.

**Penjelasan:**

> `-A INPUT` akan menambahkan (*append*) aturan ke rantai aturan `INPUT`
> `-s $NETWORK` menentukan sumber paket, dalam hal ini berasal dari jaringan 192.168.0.0/24
> `-m state` menggunakan modul *state* untuk mencocokkan status paket, `--state NEW` berarti aturan hanya berlaku untuk paket yang memulai koneksi baru
> `-p udp/tcp --dport [NOMOR_PORT]` menerapkan aturan untuk protokol tertentu dan nomor port tujuan
> `-j ACCEPT` berarti menerima paket yang cocok dengan aturan tersebut


### Nonaktifkan Apparmour di libvirtd
```
ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
```

**Penjelasan:**

> `ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/`
> Perintah ini membuat symbolic link (tautan simbolik), yaitu jenis file yang menunjuk ke file atau direktori lain, dari `/etc/apparmor.d/usr.sbin.libvirtd` ke `/etc/apparmor.d/disable/`. Ini secara efektif menonaktifkan profil AppArmor untuk `usr.sbin.libvirtd`.

> `ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/`
> Perintah ini membuat symbolic link dari `/etc/apparmor.d/usr.lib.libvirt.virt-aa-helper` ke `/etc/apparmor.d/disable/`, sehingga menonaktifkan profil AppArmor untuk `usr.lib.libvirt.virt-aa-helper`.

> `apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd`
> Perintah ini menggunakan utilitas `apparmor_parser` untuk menghapus profil AppArmor `usr.sbin.libvirtd` dari kernel. Opsi `-R` memberitahu `apparmor_parser` untuk menghapus profil.

> `apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper`
> Perintah ini menghapus profil AppArmor `usr.lib.libvirt.virt-aa-helper` dari kernel.



### Luncurkan Server manajemen
```
cloudstack-setup-management
systemctl status cloudstack-management
tail -f /var/log/cloudstack/management/management-server.log #if you want to troubleshoot 
#wait until all services (components) running successfully
```

**Penjelasan:**

> `cloudstack-setup-management`
> Perintah ini digunakan untuk mengatur management server Apache CloudStack, sebuah perangkat lunak open-source untuk membuat, mengelola, dan menjalankan layanan infrastruktur cloud. Perintah ini mengonfigurasi koneksi database, mengatur alamat IP management server, dan menjalankan management server.

> `systemctl status cloudstack-management`
> Perintah ini menggunakan systemctl, manajer sistem dan layanan di Linux, untuk menampilkan status layanan cloudstack-management. Perintah ini menunjukkan apakah layanan sedang berjalan atau tidak, serta menampilkan entri log terbaru. Kamu dapat menggunakan perintah ini untuk memeriksa apakah management server CloudStack berjalan dengan baik.

> `tail -f /var/log/cloudstack/management/management-server.log #if you want to troubleshoot`
> Perintah ini menampilkan bagian akhir file log server management CloudStack dan kemudian menampilkan data tambahan secara real-time saat file tersebut bertambah. Ini berguna untuk memantau file log secara langsung dan sangat membantu saat melakukan troubleshooting jika ada masalah dengan management server CloudStack. Opsi `-f` membuat `tail` terus membuka file dan menampilkan baris baru yang ditambahkan.

### Buka Web Browser dan Masukkan

```
http://<YOUR_IP_ADDRESS>:8080
```

Example:
```
http://192.168.0.242:8080
```

### Kamu Dapat Melihat dashboard Cloudstack 

> ![image](https://hackmd.io/_uploads/ByD0YXsbgx.png)

#### Login dengan Default
```
username : admin 
password : password
```