# Apache CloudStack Private Cloud Setup

## Contributors
- Adhelia Putri Maylani (2206814816)
- Beres Bakti Parsaoran Siagian (2206817585)
- Nahl Syareza Rahidra (2206830340)
- Nakita Rahma Dinanti (2206059401)
- Tanto Efrem Lesmana (2206031391)

## Project Overview
Tujuan dari proyek ini adalah untuk membangun dan mengonfigurasi **Private Cloud** menggunakan **Apache CloudStack**, sebuah platform open-source yang memungkinkan penerapan **Infrastructure-as-a-Service (IaaS)**. Proyek ini bertujuan untuk memberikan pemahaman yang lebih baik tentang cara mengelola cloud pribadi serta memberikan pengalaman praktis dalam pengaturan cloud computing, dengan menggunakan **Ubuntu Server** dan **KVM Hypervisor**.

## Objective
1. Menggunakan Apache CloudStack untuk membuat dan mengelola cloud pribadi yang dapat meng-host mesin virtual (VM).
2. Menerapkan KVM (Kernel-based Virtual Machine) sebagai hypervisor untuk virtualisasi mesin dalam lingkungan cloud.
3. Memahami dan mengonfigurasi CloudStack sebagai alat untuk mengelola berbagai sumber daya dalam cloud, seperti server, storage, dan network.
4. Meningkatkan pemahaman tentang cara menambah node, mengonfigurasi jaringan, dan mengelola sumber daya cloud di berbagai lingkungan fisik.

## Setup Guide
Untuk panduan detail instalasi dan konfigurasi, silakan merujuk ke [Cloudstack Installation and Configuration](Cloudstack%20Installation%20and%20Configuration) untuk langkah-langkah teknis yang lebih mendalam.

## Technologies Used
1. **Apache CloudStack**
Apache CloudStack adalah platform open-source untuk membangun dan mengelola cloud pribadi dan publik. Sebagai penyedia **Infrastructure-as-a-Service (IaaS)**, CloudStack memungkinkan administrator untuk mengelola mesin virtual (VM), storage, dan jaringan dalam skala besar. Dengan fitur seperti multi-tenancy, automasi, dan manajemen sumber daya yang mudah, CloudStack memungkinkan penyedia layanan cloud untuk mengelola dan mengoptimalkan lingkungan virtual mereka secara efektif. CloudStack mendukung berbagai hypervisor dan menyediakan API serta antarmuka pengguna yang intuitif untuk mempermudah administrasi cloud.
2. **Ubuntu Server 24.04**
Ubuntu Server adalah sistem operasi berbasis Linux yang populer digunakan dalam pengelolaan server dan infrastruktur cloud. Dalam proyek ini, **Ubuntu Server 24.04** digunakan sebagai dasar sistem operasi untuk host cloud pribadi. Ubuntu dikenal karena kestabilannya, kemudahan penggunaan, serta komunitas besar yang menyediakan banyak dokumentasi dan dukungan. Ubuntu Server juga cocok untuk digunakan dalam cloud computing karena skalabilitas dan kemampuannya untuk mendukung berbagai aplikasi dan layanan.
3. **KVM Hypervisor**
**KVM (Kernel-based Virtual Machine)** adalah solusi virtualisasi berbasis kernel Linux yang memungkinkan eksekusi mesin virtual di server fisik. KVM mendukung berbagai sistem operasi tamu, termasuk Linux, Windows, dan lainnya, menjadikannya pilihan yang sangat baik untuk mengelola beban kerja cloud. Dalam proyek ini, KVM digunakan sebagai hypervisor untuk menyediakan virtualisasi mesin di cloud yang dibangun dengan CloudStack. KVM mengintegrasikan langsung dengan kernel Linux, memberikan kinerja yang efisien dan skalabilitas yang baik dalam lingkungan virtual.
4. **iptables**
**iptables** adalah alat untuk mengonfigurasi firewall di sistem berbasis Linux, termasuk Ubuntu. Alat ini digunakan untuk memfilter dan mengontrol aliran lalu lintas jaringan pada server yang menjalankan CloudStack. Dalam proyek ini, **iptables** digunakan untuk membuka dan memblokir port tertentu untuk memastikan keamanan dan integritas sistem cloud pribadi yang sedang dibangun. Konfigurasi firewall yang tepat penting untuk melindungi sistem dari akses yang tidak sah dan mengatur komunikasi antar node dalam cloud.
5. **UUID (Universally Unique Identifier)**
UUID adalah standar pengidentifikasi yang digunakan untuk menghasilkan ID unik pada sistem komputasi. Dalam proyek ini, UUID digunakan untuk menghasilkan ID unik untuk setiap host dalam konfigurasi CloudStack. Dengan menggunakan UUID, kita dapat memastikan bahwa setiap node dalam cloud memiliki identitas yang dapat dibedakan, yang sangat penting untuk manajemen sumber daya dan pengelolaan cluster dalam lingkungan cloud yang besar.
6. **qemu-kvm**
**qemu-kvm** adalah paket yang memungkinkan KVM untuk digunakan dengan perangkat keras virtualisasi penuh. QEMU (Quick Emulator) adalah emulator mesin yang dapat menjalankan berbagai sistem operasi dalam mode virtual. Dalam konteks proyek ini, **qemu-kvm** digunakan untuk mengonfigurasi dan menjalankan mesin virtual yang terisolasi di dalam server fisik yang meng-host cloud pribadi. Integrasi KVM dengan QEMU memungkinkan penggunaan sumber daya perangkat keras secara lebih efisien dan mengoptimalkan kinerja VM.

## License

