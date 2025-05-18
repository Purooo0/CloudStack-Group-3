---
title: Dokumentasi Cloudstack Installation

---

# Install and Configure Apache Cloudstack Private Cloud

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


## Introduction

### Apache CloudStack

Apache CloudStack adalah perangkat lunak cloud computing sumber terbuka yang dirancang untuk menerapkan dan mengelola jaringan besar mesin virtual (VM). Sebagai platform Infrastructure-as-a-Service (IaaS), CloudStack menyediakan seperangkat fitur lengkap untuk membangun dan mengelola lingkungan cloud yang skalabel. CloudStack mendukung berbagai hypervisor, kemampuan API yang luas, dan serangkaian alat yang kaya bagi administrator cloud.

---

## Computing Environment Setup
```
CPU: Intel i5-10210U (8) @ 4.200GHz
RAM : 32 GB
Storage : 250GB
Network : Ethernet 100GB/s
Operating System : Ubuntu Server 24.04
```

### Persiapkan Sistem Operasi

Menggunakan **Ubuntu 22.04 LTS**

### Perbarui Sistem

Perbarui paket-paket di sistem terlebih dahulu untuk memastikan semua sudah terbaru:

```bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
```

---

## Configure Network

> ⚠️ **WAJIB menggunakan koneksi kabel Ethernet (bukan Wi-Fi)** untuk memastikan koneksi yang stabil. Hal ini sangat penting terutama saat proses pembuatan **Zone** dalam CloudStack, karena konfigurasi jaringan dan virtualisasi sangat sensitif terhadap kestabilan koneksi.

---

## CloudStack Installation (Controller and Compute Node at the Same Host)

### Import Kunci Repository

```bash
sudo -i
mkdir -p /etc/apt/keyrings 
wget -O- http://packages.shapeblue.com/release.asc | gpg --dearmor | sudo tee /etc/apt/keyrings/cloudstack.gpg > /dev/null
echo deb [signed-by=/etc/apt/keyrings/cloudstack.gpg] http://packages.shapeblue.com/cloudstack/upstream/debian/4.18 / > /etc/apt/sources.list.d/cloudstack.list
```

### Mengecek Repository yang ditambahkan

```bash
nano /etc/apt/sources.list.d/cloudstack.list
```

Pastikan baris berikut ada di bagian akhir:

```bash
deb [signed-by=/etc/apt/keyrings/cloudstack.gpg] http://packages.shapeblue.com/cloudstack/upstream/debian/4.18 /
```

---

## Configure Cloudstack Host with KVM Hypervisor

### Install KVM dan Cloudstack Agent

```bash
apt-get install qemu-kvm cloudstack-agent -y
```

### Konfigurasi KVM Virtualization Management

#### Ganti beberapa line

```bash
sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf

# Pada Ubuntu 22.04, tambahkan argumen berikut ke file /etc/default/libvirtd
sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd
```

**Penjelasan**:

* Mengaktifkan akses VNC dari IP manapun
* Mengatur `libvirtd` agar mendengarkan koneksi melalui TCP

#### Tambahkan beberapa line

```bash
echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
```

#### Restart libvirtd

```bash
systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket
systemctl restart libvirtd
```

---

### Konfigurasi untuk Mendukung Docker dan Service Lain

```bash
echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
sysctl -p
```

---

### Generate Host ID Unik

```bash
apt-get install uuid -y
UUID=$(uuid)
echo host_uuid = \"$UUID\" >> /etc/libvirt/libvirtd.conf
systemctl restart libvirtd
```

---

### Konfigurasi Iptables Firewall

```bash
NETWORK=192.168.106.0/24
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

apt-get install iptables-persistent
# Ketika ditanya, jawab "yes" untuk menyimpan konfigurasi
```

---

## Multi-host Cloud Infrastructure

## Managing Cloud Resource Through CLI

## Complete Video Tutorial