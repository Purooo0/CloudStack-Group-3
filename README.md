# Apache CloudStack Private Cloud Setup
[Link YouTube]()

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

