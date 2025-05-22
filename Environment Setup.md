## Computing Environment Setup

```
Network Address : 192.168.0.0/24
Host IP address : 192.168.0.242/24
Gateway : 192.168.0.1
Public IP : 100.73.78.15 (Menggunakan Tailscale)
```

### Konfigurasi Jaringan

**Ubah file yang terdapat di dalam folder /etc/netplan. Jika belum ada file, silakan dibuat sendiri dengan format <nama-file>.yaml. Di sini kita akan menggunakan/membuat file 01-netcfg.yaml**
```
cd /etc/netplan
sudo nano 01-netcfg.yaml
```

**Jika file sudah ada, kemungkinan akan terlihat seperti ini:**
**NOTE: TIDAK BISA MENGGUNAKAN WIFI, HARUS MENGGUNAKAN ETHERNET. MAKA DARI ITU SEDIAKAN KABEL LAN DAN KEMUDIAN HUBUNGKAN DENGAN ROUTER DI RUMAH KALIAN**

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

**Jika baru dibuat, maka file akan kosong.**

**Ubah file menjadi seperti ini.** </br>
**Nama port untuk ethernets disesuaikan dengan nama port yang ada pada laptop masing-masing. Port bisa dilihat dengan cara menjalankan command**</br>
```
ip a
```
**IP Address juga dapat dilihat dengan menjalankan command tersebut (jika sudah terhubung dengan kabel ethernet dan WAJIB MENGGUNAKAN KABEL ETHERNET)**

**Cari port yang berawalan dengan en (contoh di sini misalnya enp1s0)**


![image](https://hackmd.io/_uploads/SysWX7jWex.png)

### Terapkan Konfigurasi Jaringan

```
sudo -i  #open new shell with root privileges
netplan generate #generate config file for the renderer
netplan apply  #applies network configuration to the system
reboot #reboot the system
```

> **Catatan:** Kamu mungkin akan menemui error pada langkah ini. Pastikan kamu menggunakan **spasi** dan **bukan tab** saat mengubah file konfigurasi jaringan.
Dan ntuk mengecek apakah konfigurasi jaringan sudah diterapkan atau belum, gunakan perintah `ifconfig` dan cari interface bernama **"br0"**. Pastikan alamat IP-nya sesuai dengan yang sudah kamu atur sebelumnya.


### Uji Jaringan, Pastikan Konfigurasi yang Diterapkan
```
ifconfig     #check the ip address and existing interface
ping -c 5 google.com  #make sure you could connect to the internet
```

> **Catatan:** Jika kamu tidak bisa melakukan ping ke google.com, coba ping ke gateway kamu dan ke 8.8.8.8. Langkah ini akan membantu mengetahui letak masalah koneksi antara komputermu dan internet. Pastikan tidak ada masalah dengan koneksi internet karena kamu akan menginstal beberapa paket dari internet.


### Masuk ke Sistem Sebagai Pengguna Root
```
su -
```

### Menginstal Alat Pemantauan Sumber Daya Perangkat Keras
```
apt update -y
apt upgrade -y
apt install htop lynx duf -y
apt install bridge-utils
```

> **htop** adalah alat untuk memantau penggunaan CPU.
> **duf** adalah alat untuk memantau penggunaan disk.
> **lynx** adalah browser web berbasis CLI (Command Line Interface).


### Konfigurasi LVM (optional)
```
#not required unless logical volume is used
lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
resize2fs /dev/ubuntu-vg/ubuntu-lv
```

### Menginstal layanan jaringan dan editor teks
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


### Aktifkan login root SSH
```
sed -i '/#PermitRootLogin prohibit-password/a PermitRootLogin yes' /etc/ssh/sshd_config
#restart ssh service
service ssh restart
#or
systemctl restart sshd.service
```

### Periksa Konfigurasi SSH
```
nano /etc/ssh/sshd_config
```

> Find the line 'PermitRootLogin' make sure it set to 'yes' like this
> ![image](https://hackmd.io/_uploads/HyHUH7sWll.png)

---