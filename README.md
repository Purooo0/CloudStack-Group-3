# Apache CloudStack Private Cloud Setup
[Link YouTube](https://youtu.be/h7UMVRyZawU)

<div style="background-color: #dfd5d4; padding: 15px; border-radius: 8px; color: #27272a;">
<b>Contributors :</b>
<ul>
  <li style="color: #27272a;">Adhelia Putri Maylani (2206814816)</li>
  <li style="color: #27272a;">Beres Bakti Parsaoran Siagian (2206817585)
</li>
  <li style="color: #27272a;">Nahl Syareza Rahidra (2206830340)</li>
  <li style="color: #27272a;">Nakita Rahma Dinanti (2206059401)</li>
  <li style="color: #27272a;">Tanto Efrem Lesmana (2206031391)</li>
</ul>
</div>

## Project Overview
Tujuan dari proyek ini adalah untuk membangun dan mengonfigurasi **Private Cloud** menggunakan **Apache CloudStack**, sebuah platform open-source yang memungkinkan penerapan **Infrastructure-as-a-Service (IaaS)**. Proyek ini bertujuan untuk memberikan pemahaman yang lebih baik tentang cara mengelola cloud pribadi serta memberikan pengalaman praktis dalam pengaturan cloud computing, dengan menggunakan **Ubuntu Server** dan **KVM Hypervisor**.

## Objective
1. Menggunakan Apache CloudStack untuk membuat dan mengelola cloud pribadi yang dapat meng-host mesin virtual (VM).
2. Menerapkan KVM (Kernel-based Virtual Machine) sebagai hypervisor untuk virtualisasi mesin dalam lingkungan cloud.
3. Memahami dan mengonfigurasi CloudStack sebagai alat untuk mengelola berbagai sumber daya dalam cloud, seperti server, storage, dan network.
4. Meningkatkan pemahaman tentang cara menambah node, mengonfigurasi jaringan, dan mengelola sumber daya cloud di berbagai lingkungan fisik.

## Computing Environment Setup

### Network Address

```
Network Address : 192.168.0.0/24
Host IP address : 192.168.0.242/24
Gateway : 192.168.0.1
Public IP : 100.73.78.15
```

### Configure Network

**Edit the network configuration file under /netplan directory**
```
cd /etc/netplan
sudo nano ./0*.yaml
```

**The opened file will look like this:**

```
# This is the network config written by 'subiquity'
network:
  version: 2
  ethernets:
    enp0s3:                             # Interface Name
      addresses: [10.1.1.8/24]          # Ip Address
      gateway4: 10.1.1.1                # Gateway
      nameservers:
        addresses: [10.1.1.1,8.8.8.8]   # DNS Server
```

**Edit the file**

![image](https://hackmd.io/_uploads/SysWX7jWex.png)

### Apply Network Configuration

```
sudo -i  #open new shell with root privileges
netplan generate #generate config file for the renderer
netplan apply  #applies network configuration to the system
reboot #reboot the system
```

> **Catatan:** Kamu mungkin akan menemui error pada langkah ini. Pastikan kamu menggunakan **spasi** dan **bukan tab** saat mengubah file konfigurasi jaringan.
Dan ntuk mengecek apakah konfigurasi jaringan sudah diterapkan atau belum, gunakan perintah `ifconfig` dan cari interface bernama **"br0"**. Pastikan alamat IP-nya sesuai dengan yang sudah kamu atur sebelumnya.


### Test the Network, make sure the configuration applied
```
ifconfig     #check the ip address and existing interface
ping -c 5 google.com  #make sure you could connect to the internet
```

> **Catatan:** Jika kamu tidak bisa melakukan ping ke google.com, coba ping ke gateway kamu dan ke 8.8.8.8. Langkah ini akan membantu mengetahui letak masalah koneksi antara komputermu dan internet. Pastikan tidak ada masalah dengan koneksi internet karena kamu akan menginstal beberapa paket dari internet.


### Login to the system as root user
```
su -
```

### Installing Hardware resource monitoring tools
```
apt update -y
apt upgrade -y
apt install htop lynx duf -y
apt install bridge-utils
```

> **htop** adalah alat untuk memantau penggunaan CPU.
> **duf** adalah alat untuk memantau penggunaan disk.
> **lynx** adalah browser web berbasis CLI (Command Line Interface).


### Configure LVM (optional)
```
#not required unless logical volume is used
lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
resize2fs /dev/ubuntu-vg/ubuntu-lv
```

### Installing Network services and text editor
```
apt-get install openntpd openssh-server sudo vim tar -y
apt-get install intel-microcode -y
passwd root
#change it to Pa$$w0rd
```
> **openntpd** adalah klien NTP untuk menyinkronkan waktu antara host dan seluruh internet.

> **openssh-server** adalah server SSH untuk mengaktifkan akses jarak jauh.

> **vim** adalah editor teks.

> **tar** adalah alat untuk mengompres dan mengekstrak file. Tar sering digunakan untuk mengekstrak file hasil unduhan.

> **intel-microcode** adalah sekumpulan prosedur yang ditulis dalam assembly x86 untuk meningkatkan modularitas pada level rendah.


### Enable SSH root login
```
sed -i '/#PermitRootLogin prohibit-password/a PermitRootLogin yes' /etc/ssh/sshd_config
#restart ssh service
service ssh restart
#or
systemctl restart sshd.service
```

### Check the SSH Configuration
```
nano /etc/ssh/sshd_config
```

> Find the line 'PermitRootLogin' make sure it set to 'yes' like this
> ![image](https://hackmd.io/_uploads/HyHUH7sWll.png)

---
## Cloudstack Installation (Controller and Compute Node at the Same Host)
---
### Importing cloudstack repositories key
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

### Check the added repositories
```
nano /etc/apt/sources.list.d/cloudstack.list
```
> make sure you see this line at the last line
> ![image](https://hackmd.io/_uploads/HyZ0BQsZel.png)

### Installing Cloudstack and Mysql Server

```
apt-get update -y
apt-get install cloudstack-management mysql-server
```
> Note : Proses ini lumayan lama


### Configure mysql
#### Open mysql config file

```
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

#### Copy these lines under [mysqld] section

```
server-id = 1
sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_ENGINE_SUBSTITUTION"
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=1000
log-bin=mysql-bin
binlog-format = 'ROW'
```

#### Restart mysql service
```
systemctl restart mysql
```

#### Check mysql service status
```
systemctl status mysql
```
> ![image](https://hackmd.io/_uploads/HkmdPQiWxl.png)

### Deploy Database as Root and Then Create "cloud" User with Password "cloud" too

```
cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:Pa$$w0rd -i 192.168.104.24
```

### Configure Primary and Secondary Storage
```
apt-get install nfs-kernel-server quota
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
mkdir -p /export/primary /export/secondary
exportfs -a
```

### Configure NFS Server
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

### Install KVM and Cloudstack Agent
```
apt-get install qemu-kvm cloudstack-agent -y
```

### Configure KVM Virtualization Management
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

#### Add some lines
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

### Configuration to Support Docker and Other Services
```
echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
sysctl -p
```

Explanation
> ARP and IP packet will not processed by arptable and iptable

### Generate Unique Host ID

```
apt-get install uuid -y
UUID=$(uuid)
echo host_uuid = \"$UUID\" >> /etc/libvirt/libvirtd.conf
systemctl restart libvirtd
```

### Configure Iptables Firewall and Make it persistent

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


### Disable apparmour on libvirtd
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



### Launch management Server
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

### Open web browser and type

```
http://<YOUR_IP_ADDRESS>:8080
```

Example:
```
http://192.168.0.242:8080
```

### You should see the cloudstack dashboard

> ![image](https://hackmd.io/_uploads/ByD0YXsbgx.png)

#### Login with default 
```
username : admin 
password : password
```

#### setup Zone
> ![image](https://hackmd.io/_uploads/SJkzpmsbxe.png)

> ![image](https://hackmd.io/_uploads/By2E6Qo-lx.png)

> ![image](https://hackmd.io/_uploads/Hk4npQsWge.png)

> ![image](https://hackmd.io/_uploads/rJW06XiWge.png)

> ![image](https://hackmd.io/_uploads/H1eGCQjWex.png)

> ![image](https://hackmd.io/_uploads/SkB9CQo-xe.png)

>![image](https://hackmd.io/_uploads/rJ_-yEiWle.png)

>![image](https://hackmd.io/_uploads/rJSDJVsbxl.png)

>![image](https://hackmd.io/_uploads/rkfYyVoWgl.png)

>![image](https://hackmd.io/_uploads/rko31VjZgg.png)

>![image](https://hackmd.io/_uploads/H1rCyNi-le.png)

>![image](https://hackmd.io/_uploads/SkikgNjbee.png)

> **After setup the Zone, you will see dashbord page**
> ![image](https://hackmd.io/_uploads/HkJBeNobel.png)

> **Install ubuntu 22.04 ISO**
> ![image](https://hackmd.io/_uploads/B1iqgNsbxe.png)


---
## Multi-host Cloud Infrastructure
---
### Importing cloudstack repositories key
```
sudo -i
mkdir -p /etc/apt/keyrings
wget -O- http://packages.shapeblue.com/release.asc | gpg --dearmor | sudo tee /etc/apt/keyrings/cloudstack.gpg > /dev/null
echo deb [signed-by=/etc/apt/keyrings/cloudstack.gpg] http://packages.shapeblue.com/cloudstack/upstream/debian/4.18 / > /etc/apt/sources.list.d/cloudstack.list
```

### Installing KVM host and Cloudstack-Agent
```
apt-get install qemu-kvm cloudstack-agent
```

### Configure Qemu KVM Virtualisation Management (libvirtd

```
sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf
# On Ubuntu 22.04, add LIBVIRTD_ARGS="--listen" to /etc/default/libvirtd instead.

sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd

#configure default libvirtd configuration

echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf

systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket
systemctl restart libvirtd
```

### Configuration to Support Docker Services
```
#on certain hosts where you may be running docker and other services, 
#you may need to add the following in /etc/sysctl.conf
#and then run sysctl -p: --> /etc/sysctl.conf

echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
sysctl -p
```

### Generate Unique Host ID
```
apt-get install uuid -y
UUID=$(uuid)
echo host_uuid = \"$UUID\" >> /etc/libvirt/libvirtd.conf
systemctl restart libvirtd
```

### Disable Firewall

```
systemctl ufw disable
```

### Disable Apparmour on libvirtd
```
ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
```

### STORAGE SETUP for Additional PRIMARY and SECONDARY

```
apt-get install nfs-kernel-server quota
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
mkdir -p /export/primary /export/secondary
exportfs -a
```

### Configure NFS Server
```
sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server
sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common
echo "NEED_STATD=yes" >> /etc/default/nfs-common
sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota
service nfs-kernel-server restart
```

### Permit SSH root login

```
sed -i '/#PermitRootLogin prohibit-password/a PermitRootLogin yes' /etc/ssh/sshd_config
#restart ssh service
service ssh restart
```

--- 
## Managing Cloud Resource Through CLI

### Installing Cloudmonkey
```
snap install cloudmonkey
cloudmonkey --version
```

### Configure Cloudmonkey

```
cloudmonkey set url [IP Address:8080]/client/api
#example: cloudmonkey set url http://192.168.104.24:8080/client/api
cloudmonkey set apikey [apikey]
cloudmonkey set secretkey [secretkey]
cloudmonkey sync
```

API key and secret key generated in user page



### Create an instance
```
cloudmonkey list serverofferings | grep id
cloudmonkey list zones |grep id
cloudmonkey list templates templatefilter=all | grep id

cloudmonkey deploy virtualmachine templateid=[TEMPLATE-ID] serviceofferingid=[SERVICE-ID] zoneid=[ZONE-ID]

```

> ![image](https://hackmd.io/_uploads/S1XDO4iWel.png)

---

### Deploying a Virtual Machine in CloudStack and Configuring Tailscale Advertise Routes for Local Network Access

```bash
sudo tailscale up --advertise-routes=192.168.0.0/24 --operator=pc-2020-0018
```

> ![image](https://hackmd.io/_uploads/B15CtVo-le.png)

---

#### Accessing the VM Console from Apache CloudStack GUI

Setelah menjalankan perintah Tailscale di atas, kamu sekarang bisa mengakses **Console/CLI** dari VM langsung melalui GUI Apache CloudStack.

> ![image](https://hackmd.io/_uploads/rJJc9VoZgl.png)
> ![image](https://hackmd.io/_uploads/ryCciVjbgx.png)

---

#### Enabling Internet Access from the VM

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

#### SSH Access to the VM

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

#### Firewall Configuration for SSH and Ping
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



## Common Troubleshooting dan Concern
Berikut terdapat beberapa list permasalahan yang mungkin muncul ketika melakukan setup **SETELAH** bisa mengakses dashboard
- Setup Netplan (Concern)
Untuk setup networknya, **WAJIB** menggunakan kabel ethernet dan **TIDAK BISA** menggunakan WiFi. Solusinya tinggal bawa aja laptop ke salah satu rumah temen kalian dan pasang aja langsung ke kabel ethernet. Untuk melihat port untuk ethernet, lakukan command 'sudo ip a', dan cari port yang dengan 'en' (misalnya enp0s3, enp1s0, eno1, enx00e04cda0182, dll).
- Zone Creation Failed: Cannot deploy with paramaeters... (Troubleshooting)
Ini biasanya dikarenakan kita salah setup di netplan sebelumnya. Hal ini kami jumpai ketika masih menggunakan WiFi dan kemudian mencoba mengonfigurasikan zone pada dashboard. Solusinya terdapat di atas. 
- Zone Creation Failed: Authentication Error (Troubleshooting)
Permasalahan ini terjadi ketika password root yang diberikan ketika melakukan command cloudstack-setup-databases tidak sesuai dengan password root pada laptop tersebut. Solusinya adalah mengganti password root menggunakan command 'sudo passwd root', dan kemudian akan diminta password. Masukkan password dan simpan baik-baik. Intinya, password root pada command cloudstack-setup-databases dan password root milik laptop harus sama
- Ingin mengulang setup Cloudstack (Concern)
Jika terdapat kesalahan konfigurasi, misalnya ingin mengubah dari advanced network ke basic network, dan ingin mengulang setup Cloudstack, kalian bisa mencoba melakukan ini. Masukkan command 'sudo systemctl stop cloudstack-management cloudstack-agent'. Masuk ke MySQL dengan menggunakan command 'sudo mysql -u root'. Kemudian jalankan query 'DROP DATABASE cloud; DROP DATABASE cloud_usage;', tunggu sampai selesai, kemudian exit. Jalankan lagi perintah 'sudo cloudstack-setup-databases...'. Setelah itu jalankan perintah 'sudo cloudstack-setup-management'. Tunggu beberapa saat, atau buka log file nya dengan command 'tail -f /var/log/cloudstack/management/management-server.log'. Jika log file sudah mulai terlihat normal, atau di-spam dengan SSL, berarti kita sudah bisa masuk ke Cloudstack Dashboard dan melakukan konfigurasi.
- (KHUSUS Pengguna Tailscale) Tidak bisa mengakses console dari consoleproxy atau secondarystoragevm pada network yang berbeda dari laptop server.
Pertama-tama pastikan kalian sudah mengaktifkan advertise routes, masukkan command 'sudo tailscale up --advertise-routes=<IP network laptop server>/<Subnet network laptop server> --accept-routes'. Kemudian buka dashboard Tailscale dan klik laptop server kalian. Dan cari bagian subnet, seharusnya terdapat satu entry yang terdapat dalam 'Awaiting approval'. Jika belum, maka tunggu sesaat. Jika sudah ada, klik 'edit' dan ceklis IP network, kemudian klik 'Save'.
Jika sudah setup subnet pada Tailscale dan masih belum bisa, terdapat dua cara yang dapat dilakukan, cara pertama yaitu menjalankan command 'sudo systemctl restart tailscaled'. Jika masih belum bisa, maka restart laptop server dengan command 'reboot' dan tunggu sampai laptop menyala kembali. Setelah login ulang, tunggu beberapa saat dan masukkan command 'sudo systemctl start tailscaled' dilanjut dengan 'sudo systemctl restart tailscaled'. Langkah ini seharusnya memperbolehkan kita untuk mengakses console dari consoleproxy ataupun secondarystoragevm, dan bahkan bisa mengakses VM jika berada pada konfigurasi Basic.

## Technologies Used
1. **Apache CloudStack**
   Apache CloudStack adalah platform open-source untuk membangun dan mengelola cloud pribadi dan publik. Sebagai penyedia **Infrastructure-as-a-Service (IaaS)**, CloudStack memungkinkan administrator untuk mengelola mesin virtual (VM), storage, dan jaringan dalam skala besar. Dengan fitur seperti multi-tenancy, automasi, dan manajemen sumber daya yang mudah, CloudStack memungkinkan penyedia layanan cloud untuk mengelola dan mengoptimalkan lingkungan virtual mereka secara efektif. CloudStack mendukung berbagai hypervisor dan menyediakan API serta antarmuka pengguna yang intuitif untuk mempermudah administrasi cloud.
   
2. **Ubuntu Server 24.04**
   Ubuntu Server adalah sistem operasi berbasis Linux yang populer digunakan dalam pengelolaan server dan infrastruktur cloud. Dalam proyek ini, **Ubuntu Server 24.04** digunakan sebagai dasar sistem operasi untuk host cloud pribadi.
   
3. **KVM Hypervisor**
   **KVM (Kernel-based Virtual Machine)** adalah solusi virtualisasi berbasis kernel Linux yang memungkinkan eksekusi mesin virtual di server fisik. KVM mendukung berbagai sistem operasi tamu, termasuk Linux, Windows, dan lainnya, menjadikannya pilihan yang sangat baik untuk mengelola beban kerja cloud. Dalam proyek ini, KVM digunakan sebagai hypervisor untuk menyediakan virtualisasi mesin di cloud yang dibangun dengan CloudStack. KVM mengintegrasikan langsung dengan kernel Linux, memberikan kinerja yang efisien dan skalabilitas yang baik dalam lingkungan virtual.
   
4. **iptables**
   **iptables** adalah alat untuk mengonfigurasi firewall di sistem berbasis Linux, termasuk Ubuntu. Alat ini digunakan untuk memfilter dan mengontrol aliran lalu lintas jaringan pada server yang menjalankan CloudStack. Dalam proyek ini, **iptables** digunakan untuk membuka dan memblokir port tertentu untuk memastikan keamanan dan integritas sistem cloud pribadi yang sedang dibangun.

5. **UUID (Universally Unique Identifier)**
    UUID adalah standar pengidentifikasi yang digunakan untuk menghasilkan ID unik pada sistem komputasi. Dalam proyek ini, UUID digunakan untuk menghasilkan ID unik untuk setiap host dalam konfigurasi CloudStack.
   
6. **qemu-kvm**
    **qemu-kvm** adalah paket yang memungkinkan KVM untuk digunakan dengan perangkat keras virtualisasi penuh. QEMU (Quick Emulator) adalah emulator mesin yang dapat menjalankan berbagai sistem operasi dalam mode virtual. Dalam konteks proyek ini, **qemu-kvm** digunakan untuk mengonfigurasi dan menjalankan mesin virtual yang terisolasi di dalam server fisik yang meng-host cloud pribadi.

## System Architecture

### Overview
Arsitektur sistem dari cloud pribadi ini dibangun untuk mendukung berbagai mesin virtual (VM) dan aplikasi yang dapat diakses secara efisien. Proyek ini menggunakan **Apache CloudStack** sebagai platform untuk mengelola sumber daya cloud, dengan **KVM** sebagai hypervisor untuk virtualisasi mesin.

### Components
1. **CloudStack Management Server**
   Bertindak sebagai pengendali utama dari seluruh sistem cloud. Server ini bertanggung jawab untuk pengelolaan semua VM, penyimpanan, dan jaringan melalui antarmuka pengguna atau API. CloudStack Management Server dapat dijalankan di server terpisah atau diintegrasikan dengan node pengendali lainnya.

2. **KVM Hypervisor**
   Digunakan untuk menjalankan mesin virtual di atas host fisik. Setiap VM yang dijalankan pada cloud menggunakan KVM sebagai hypervisor untuk menyediakan lingkungan virtualisasi yang efisien. KVM terintegrasi langsung dengan kernel Linux untuk kinerja yang optimal.

3. **Storage System**
   Sistem penyimpanan yang digunakan untuk menyimpan data VM dan snapshot mereka. CloudStack menyediakan opsi untuk penyimpanan lokal atau terdistribusi, memungkinkan pengguna untuk memilih metode penyimpanan yang sesuai dengan kebutuhan skalabilitas dan keandalan.

4. **Networking**
   CloudStack juga memungkinkan pengelolaan jaringan virtual, memungkinkan pembuatan dan pengaturan jaringan virtual yang menghubungkan VM di seluruh cluster. Pengaturan jaringan ini melibatkan pembuatan jaringan pribadi, VPN, dan penyesuaian pengaturan NAT dan firewall menggunakan **iptables**.

5. **Multi-host Configuration**
   Untuk mendukung skalabilitas dan redundansi, beberapa host server dapat dikonfigurasi untuk bekerja sama dalam satu cloud. Host-host ini dapat berbagi beban dan menyediakan failover untuk memastikan ketersediaan yang tinggi.

### Architecture Diagram

## License

