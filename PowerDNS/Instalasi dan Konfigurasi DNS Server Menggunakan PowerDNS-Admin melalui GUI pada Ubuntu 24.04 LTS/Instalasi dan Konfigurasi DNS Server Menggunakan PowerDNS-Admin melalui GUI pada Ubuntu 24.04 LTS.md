# Instalasi dan Konfigurasi DNS Server Menggunakan PowerDNS-Admin melalui GUI pada Ubuntu 24.04 LTS

Pelajari cara instalasi dan konfigurasi PowerDNS Server berbasis GUI di Kilat VM 2.0 dengan Ubuntu 24.04. Panduan lengkap ini membahas persiapan sistem, setup Glue Record, hingga manajemen zona domain secara visual dan mudah melalui panel PowerDNS-Admin.

***

Halo, Kawan Belajar!

DNS atau Domain Name System merupakan sistem yang digunakan untuk menerjemahkan nama domain menjadi IP Address. Dengan adanya DNS, pengguna dapat mengakses layanan menggunakan nama domain tanpa harus mengingat IP Address dari server.

Salah satu aplikasi open-source berkinerja tinggi yang dapat digunakan sebagai DNS Server adalah **PowerDNS**. Berbeda dengan DNS konvensional, PowerDNS memiliki keunggulan fleksibilitas yang tinggi karena dapat diintegrasikan dengan panel web modern untuk mempermudah pengelolaan domain.

Pada panduan kali ini, kita akan melakukan instalasi dan konfigurasi **PowerDNS Authoritative Server berbasis GUI (Graphical User Interface) menggunakan PowerDNS-Admin di Kilat VM 2.0 dengan sistem operasi Ubuntu**. Kita akan mengonfigurasi domain sendiri menggunakan nameserver ns1 dan ns2, menyiapkan Glue Record, serta melakukan manajemen record secara visual melalui dashboard panel web yang ramah pengguna.


> 💡 **Catatan Kawan Belajar:** 
> 
> Sebelum kita masuk ke langkah-langkah instalasi teknis, pastikan kamu sudah memahami dasar-dasar serta alur kerjanya melalui artikel pengenalan sebelumnya:
> 
> 👉 [Apa Itu PowerDNS? Mengenal DNS Server Authoritative yang Fleksibel](https://www.cloudkilat.com/layanan/kilat-domain)
> 
> 🖥️ Lebih suka menggunakan terminal?
> 
> 👉 [Instalasi dan Konfigurasi DNS Server Menggunakan PowerDNS melalui CLI pada Ubuntu 24.04 LTS](https://www.cloudkilat.com/layanan/kilat-domain)


## 1. Persiapan
Untuk melakukan instalasi dan konfigurasi DNS Server menggunakan PowerDNS, beberapa kebutuhan yang perlu disiapkan antara lain:

* Domain dan Kilat VM 2.0 aktif.
* Sistem operasi Ubuntu 24.04.
* Setup Glue Record.
* IP Address publik pada Kilat VM.
* Browser untuk mengakses PowerDNS-Admin
  
Jika belum memiliki Kilat VM 2.0, pengguna dapat melakukan pemesanan **layanan Kilat VM 2.0** melalui **CloudKilat**.

## Informasi Versi Sistem
Panduan ini menggunakan komponen dan versi perangkat lunak berikut:

| Komponen  | Keterangan |
| ------------- |:-------------:|
| VPS      | Kilat VM 2.0     |
| Sistem Operasi      | Ubuntu Server 24.04 LTS     |
| PowerDNS Server      | 4.8.3 (Package Ubuntu 24.04)     |
| Database Backend      | MariaDB Server     |
| DNS Management      | PowerDNS-Admin     |
| Akses      | SSH     |
| Domain      | `domainkamu.id`     |
| Public IP      | `IP_SERVER`     |
| Nameserver      | `ns1.domainkamu.id` dan `ns2.domainkamu.id`     |

> **Catatan:** Pada praktik ini digunakan satu VPS untuk menjalankan PowerDNS Authoritative Server, MariaDB, PowerDNS API, dan PowerDNS-Admin.

> **Catatan:** Ganti seluruh nilai `domainkamu.id` dan `IP_SERVER` sesuai server yang digunakan saat praktik.

## 2. Setup Glue Record
Sebelum melakukan konfigurasi PowerDNS, pastikan domain telah memiliki **Glue Record** apabila menggunakan nameserver sendiri.

Glue Record merupakan informasi IP Address yang digunakan oleh suatu nameserver dan didaftarkan pada registrar domain.


Adapun panduan cara setup Glue Record seperti berikut ini:

1. [Login Portal Client Area CloudKilat](https://portal.cloudkilat.com/clientarea) terlebih dahulu.
2. Untuk langkah-langkah lengkapnya, Anda dapat mengikuti panduan resmi melalui tautan [Cara Menggunakan Private Name Server pada Domain di Portal Client CloudKilat](https://kb.cloudkilat.id/domain-di-cloudkilat/cara-menggunakan-private-name-server-pada-domain-di-portal-client-cloudkilat).

> **Catatan:** Proses propagasi **Glue Record** dan perubahan nameserver bergantung pada registrar dan registry domain. Pengujian DNS publik dilakukan setelah konfigurasi server selesai.

## 3. Akses SSH & Update Sistem
Selanjutya, untuk melakukan instalasi dan konfigurasi DNS Server, silakan mengikuti langkah-langkah berikut ini:

Masuk ke Kilat VM 2.0 Anda terlebih dahulu, atau Anda juga bisa melakukan *remote* menggunakan SSH. Jika Anda masih belum mengetahui cara *remote* menggunakan SSH, silakan membaca panduannya melalui tautan [Cara Akses Kilat VM Melalui SSH](https://kb.cloudkilat.id/akses-kilat-vm/cara-akses-kilat-vm-melalui-ssh).

Setelah berhasil masuk ke dalam Kilat VM 2.0, perbarui repository dan paket sistem dengan menjalankan perintah berikut seperti pada Gambar 1:

```
apt update && apt upgrade -y
```

<p align="center">
  <img width="1352" height="555" alt="apt update" src="https://github.com/user-attachments/assets/5fa107d5-5933-4f65-b352-50bca91f8a1e" />
  <br>
  <em>Gambar 1: Update Paket Ubuntu Server</em>
</p>

Kemudian install beberapa utilitas yang akan digunakan:
```
apt install -y curl wget gnupg ca-certificates lsb-release software-properties-common unzip git nano dnsutils
```
<p align="center">
  <img width="1352" height="555" alt="apt update" src="https://github.com/user-attachments/assets/5fa107d5-5933-4f65-b352-50bca91f8a1e" />
  <br>
  <em>Gambar 2: Install utilitas</em>
</p>

## 4. Menonaktifkan DNS Stub Listener Ubuntu
Pada Ubuntu 24.04, terdapat *service* `systemd-resolved` yang membantu sistem melakukan koneksi ke DNS. *Service* ini menggunakan port 53 melalui fitur *DNS Stub Listener* (`127.0.0.53:53`).

PowerDNS juga membutuhkan port 53 untuk menerima permintaan DNS. Jika port tersebut sudah digunakan oleh `systemd-resolved`, PowerDNS tidak dapat berjalan karena mengalami konflik pada port yang sama.

Periksa terlebih dahulu penggunaan port 53 pada *server* Anda:
```
#ss -lntup | grep ':53'
udp UNCONN 0 0 127.0.0.54:53 0.0.0.0:* users:(("systemd-resolve",pid=468,fd=16))
udp UNCONN 0 0 127.0.0.53%lo:53 0.0.0.0:* users:(("systemd-resolve",pid=468,fd=14))
```
> **Catatan:** Jika terdapat proses `systemd-resolved` yang menggunakan port `53`, Anda perlu mengubah konfigurasi `systemd-resolved`.

Buka file konfigurasi `resolved.conf` menggunakan editor teks `nano`:
```
nano /etc/systemd/resolved.conf
```

Cari bagian `[Resolve]`, lalu tambahkan atau ubah konfigurasi baris berikut menjadi:
```
[Resolve]
DNSStubListener=no
```

<p align="center">
<img width="916" height="565" alt="isi systemd-resolved" src="https://github.com/user-attachments/assets/a629e7d9-c257-452b-8a66-985c2ba99f31" />
  <br>
  <em>Gambar 3: Menonaktifkan DNS Stub Listener</em>
</p>

Simpan konfigurasi dengan menekan tombol **Ctrl + O**, lalu Enter, dan keluar dengan **Ctrl + X**.

Setelah itu, _restart service_ `systemd-resolved` agar perubahan diterapkan:
```
systemctl restart systemd-resolved
```

Periksa kembali penggunaan port `53` untuk memastikan konflik telah teratasi:
```
ss -lntup | grep ':53'
```

> **Catatan:** Langkah ini hanya mematikan fungsi DNS stub listener pada `systemd-resolved` tanpa menghentikan service tersebut secara total, sehingga koneksi internet serta resolver pada VPS Anda dipastikan tetap berjalan dengan normal.

## 5. Instalasi PowerDNS Authoritative Server

### Menambahkan repository resmi PowerDNS
PowerDNS menyediakan repository resmi untuk Ubuntu 24.04 dengan PowerDNS Authoritative Server 5.1.x.

Buat direktori keyring:
```
install -d /etc/apt/keyrings
```

Download public key repository:
```
curl https://repo.powerdns.com/FD380FBB-pub.asc \ | tee /etc/apt/keyrings/auth-51-pub.asc
```

Buat repository PowerDNS:
```
nano /etc/apt/sources.list.d/pdns.list
```

Isi dengan:
```
deb [signed-by=/etc/apt/keyrings/auth-51-pub.asc] http://repo.powerdns.com/ubuntu noble-auth-51 main
```
<p align="center">
<img width="916" height="565" alt="isi systemd-resolved" src="https://github.com/user-attachments/assets/a629e7d9-c257-452b-8a66-985c2ba99f31" />
  <br>
  <em>Gambar 4: Membuat file repository PowerDNS</em>
</p>

Simpan konfigurasi dengan menekan tombol **Ctrl + O**, lalu Enter, dan keluar dengan **Ctrl + X**.

Kemudian buat konfigurasi pinning untuk mengunci versi paket **PowerDNS** agar tetap stabil dan mencegah pembaruan otomatis yang bisa merusak integrasi dengan panel GUI **PowerDNS-Admin**.
```
nano /etc/apt/preferences.d/auth-51
```

Isi:
```
Package: pdns-* 
Pin: origin repo.powerdns.com 
Pin-Priority: 600
```
<p align="center">
<img width="916" height="565" alt="isi systemd-resolved" src="https://github.com/user-attachments/assets/a629e7d9-c257-452b-8a66-985c2ba99f31" />
  <br>
  <em>Gambar 5: Konfigurasi Pinning</em>
</p>

Update repository:
```
apt update
```

### Install PowerDNS

Install PowerDNS Authoritative Server:
```
apt install -y pdns-server
```

Setelah proses instalasi selesai, Anda dapat memeriksa versi PowerDNS yang terinstal menggunakan perintah sebagai berikut:

```
pdns_server --version
```
<p align="center">
  <img width="1360" height="347" alt="versi powerdns" src="https://github.com/user-attachments/assets/8f2ffb58-6ae8-4c6c-9759-8e52315b8220" />
  <br>
  <em>Gambar 6: Versi PowerDNS</em>
</p>

> **Catatan:** Versi PowerDNS dapat berubah seiring adanya *release* terbaru.

Kemudian, aktifkan dan pastikan service PowerDNS berjalan dengan baik menggunakan perintah status berikut, dan pastikan statusnya bernilai active (running):

```
#systemctl status pdns
● pdns.service - PowerDNS Authoritative Server
     Loaded: loaded (/usr/lib/systemd/system/pdns.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-09-19 12:07:15 WIB; 1s ago
       Docs: man:pdns_server(1)
             man:pdns_control(1)
             https://doc.powerdns.com
   Main PID: 4177 (pdns_server)
      Tasks: 8 (limit: 1094)
     Memory: 46.6M (peak: 46.8M)
        CPU: 92ms
     CGroup: /system.slice/pdns.service
             └─4177 /usr/sbin/pdns_server --guardian=no --daemon=no --disable-syslog --log-timestamp=no>
```

## 6. Instalasi Database Backend

PowerDNS dapat menggunakan berbagai jenis *backend* untuk menyimpan data DNS. Pada praktik ini, digunakan MariaDB sebagai *database backend*.

Instal MariaDB dengan menjalankan perintah berikut:
```
apt install -y mariadb-server mariadb-client
```

<p align="center">
<img width="1359" height="387" alt="install mariadb" src="https://github.com/user-attachments/assets/fb97b494-bee4-4303-a91a-c1daa909f4b0" />
  <br>
  <em>Gambar 7: Instal MariaDB</em>
</p>

Setelah instalasi selesai, cek service MariaDB untuk memastikan berjalan dengan normal:
```
#systemctl status mariadb
● mariadb.service - MariaDB 10.11.14 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-09-19 11:26:31 WIB; 44min ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
   Main PID: 888 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 13 (limit: 7226)
     Memory: 112.6M (peak: 116.6M)
        CPU: 1.144s
     CGroup: /system.slice/mariadb.service
             └─888 /usr/sbin/mariadbd
```

Selanjutnya, install backend MariaDB untuk PowerDNS menggunakan perintah berikut:
```
apt install pdns-backend-mysql
```
<p align="center">
<img width="1366" height="523" alt="install backend mariadb" src="https://github.com/user-attachments/assets/39bc39ba-dc1a-4e45-bb25-965188e1aa20" />
  <br>
  <em>Gambar 8: Instal Backend MariaDB</em>
</p>
