
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
<img width="779" height="405" alt="apt update   apt upgrade -y" src="https://github.com/user-attachments/assets/bd2093f0-cc0c-42f8-8069-eddcfcba04ec" />
  <br>
  <em>Gambar 1: Update Paket Ubuntu Server</em>
</p>

Kemudian install beberapa utilitas yang akan digunakan:
```
apt install -y curl wget gnupg ca-certificates lsb-release software-properties-common unzip git nano dnsutils
```
<p align="center">
<img width="849" height="427" alt="install beberapa utilitas" src="https://github.com/user-attachments/assets/8fead86c-af8b-4137-8b05-10679a14f48f" />
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
<img width="846" height="430" alt="Menonaktifkan DNS Stub Listener" src="https://github.com/user-attachments/assets/520ad9f7-741a-4dd9-9d74-9a185ba1b300" />
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
<img width="849" height="430" alt="Buat repository PowerDNS" src="https://github.com/user-attachments/assets/0f2af243-7c02-4331-891e-f9e76ea6ccc6" />
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
<img width="847" height="428" alt="konfigurasi pinning" src="https://github.com/user-attachments/assets/cb46d491-259a-414e-9b7e-d3f31b251e3c" />
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
<img width="846" height="429" alt="veris power dns" src="https://github.com/user-attachments/assets/fa3936a3-517e-4592-8d43-962c7e87d067" />
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
<img width="850" height="338" alt="installll mariadb" src="https://github.com/user-attachments/assets/c11585f1-4102-4a93-bd09-077417ba2ab1" />
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
<img width="850" height="324" alt="install backend mariadbbb" src="https://github.com/user-attachments/assets/ebc69948-8560-4fd3-86fb-a418c878cf25" />
  <br>
  <em>Gambar 8: Instal Backend MariaDB</em>
</p>

## 7. Membuat Database PowerDNS

Masuk atau *login* ke MariaDB sebagai pengguna *root* dengan menjalankan perintah berikut:
```
mysql -u root -p
```

Kemudian, buat sebuah database baru untuk PowerDNS menggunakan perintah:
```
CREATE DATABASE powerdns;
```

Buat user baru untuk mengakses database tersebut 
```
CREATE USER 'powerdns'@'localhost' IDENTIFIED BY 'PASSWORD';
```
> **Catatan:** Ganti 'PASSWORD' dengan kata sandi yang kuat dan aman (kombinasi huruf besar, huruf kecil, angka, dan simbol) untuk menghindari risiko keamanan pada database Anda.

Berikan hak akses penuh kepada user tersebut pada database PowerDNS:
```
GRANT ALL PRIVILEGES ON powerdns.* TO 'powerdns'@'localhost';
```

Terakhir, perbarui hak istimewa (privileges) dengan menjalankan perintah:
```
FLUSH PRIVILEGES;
```
> **Catatan:** Apabila proses konfigurasi database telah selesai dan Anda ingin keluar dari prompt MariaDB, gunakan perintah `exit;`.

## 8. Import Database Schema PowerDNS

Setelah *backend* MariaDB PowerDNS berhasil diinstal, cari file *schema* yang tersedia pada sistem dengan menjalankan perintah berikut:
```
ls /usr/share/doc/pdns-backend-mysql/
```

Kalau ingin mencari file SQL secara lebih spesifik, Anda bisa menggunakan perintah:
```
find /usr/share -type f -iname "*.sql" | grep -i pdns
```

Kemudian lakukan import menggunakan path tersebut
```
mysql -u powerdns -p powerdns < /usr/share/pdns-backend-mysql/schema/schema.mysql.sql
```
<p align="center">
<img width="847" height="134" alt="Impor File" src="https://github.com/user-attachments/assets/f4dde935-4e6f-473a-94f7-37f21f81a9bb" />
  <br>
  <em>Gambar 9: Impor File</em>
</p>

> **Catatan:** Saat diminta password, masukkan password user powerdns yang sudah Anda buat sebelumnya

Setelah proses selesai, silakan login kembali ke database untuk memastikan tabel-tabelnya sudah terbuat:
```
mysql -u powerdns -p powerdns
```

Kemudian cek tabel untuk memastikan schema berhasil di-import dan tabel-tabel yang dibutuhkan PowerDNS muncul:
```
SHOW TABLES;
```
<p align="center">
<img width="848" height="318" alt="showw tabless" src="https://github.com/user-attachments/assets/ad616fea-eae9-4379-acb5-1c08844eb4ca" />
  <br>
  <em>Gambar 10: Show Tables</em>
</p>

## 9. Konfigurasi Backend PowerDNS

Nah, setelah *schema* masuk ke *database*, baru kita beri tahu PowerDNS bahwa data DNS disimpan di dalam *database* MariaDB.

Edit file konfigurasi utama PowerDNS dengan menggunakan editor teks `nano`:
```
nano /etc/powerdns/pdns.conf
```

Tambahkan atau sesuaikan konfigurasi backend database di dalam file tersebut seperti berikut:
```
launch=gmysql 
gmysql-host=127.0.0.1 
gmysql-port=3306 
gmysql-dbname=powerdns 
gmysql-user=powerdns 
gmysql-password=PASSWORD_ANDA 
gmysql-dnssec=yes
```
<p align="center">
<img width="848" height="429" alt="isi nano pdns" src="https://github.com/user-attachments/assets/a88d2ea1-5872-454a-b84a-32ecb947768e" />
  <br>
  <em>Gambar 11: File Pdns</em>
</p>

Setelah konfigurasi disimpan, restart service PowerDNS untuk menerapkan perubahan:
```
systemctl restart pdns
```

Kemudian cek kembali status service PowerDNS untuk memastikan semuanya berjalan dengan normal:
```
#systemctl status pdns
● pdns.service - PowerDNS Authoritative Server
     Loaded: loaded (/usr/lib/systemd/system/pdns.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-09-19 14:47:50 WIB; 9s ago
       Docs: man:pdns_server(1)
             man:pdns_control(1)
             https://doc.powerdns.com
   Main PID: 5508 (pdns_server)
      Tasks: 8 (limit: 1094)
     Memory: 46.6M (peak: 46.9M)
        CPU: 161ms
     CGroup: /system.slice/pdns.service
             └─5508 /usr/sbin/pdns_server --guardian=no --daemon=no --disable-syslog --log-timestamp=no>
```

> **Catatan:** Pastikan statusnya menunjukkan keterangan active (running):

## 10. Konfigurasi PowerDNS API
**PowerDNS-Admin** membutuhkan **PowerDNS API** untuk berkomunikasi dengan _PowerDNS Authoritative Server_. PowerDNS menyediakan REST API melalui webserver internalnya. API menggunakan API Key melalui header `X-API-Key`.

Membuat Kunci API yang Kuat
```
openssl rand -hex 32
```
> **Catatan:** Salin (copy) hasil string acak yang dihasilkan oleh perintah di atas untuk digunakan sebagai `api-key.`

Buka konfigurasi:
```
nano /etc/powerdns/pdns.conf
```

Tambahkan:
```
api=yes
api-key=ISI_DENGAN_HASIL_OPENSSL_RANDOM_TADI

webserver=yes
webserver-address=127.0.0.1
webserver-port=8081
```

<p align="center">
<img width="847" height="403" alt="Isin file pdns API" src="https://github.com/user-attachments/assets/837dd78f-9832-4a06-8b6b-2d2810060e9f" />
  <br>
  <em>Gambar 12: File Pdns API</em>
</p>

Restart PowerDNS:
```
systemctl restart pdns
```

Periksa port API:
```
ss -lntup | grep 8081
```

Kemudian lakukan pengujian API:
```
curl -H "X-API-Key: API_KEY_ANDA" http://127.0.0.1:8081/api/v1/servers/localhost
```

Jika berhasil, PowerDNS akan mengembalikan informasi server dalam format JSON.
Contoh:
```
{ 
  "type": "Server", 
  "id": "localhost", 
  "daemon_type": "authoritative", 
  "version": "5.1.x" 
}
```

> **Catatan:** Pada konfigurasi ini API hanya menerima koneksi dari server lokal melalui `127.0.0.1.` Hal ini lebih aman karena PowerDNS-Admin akan berjalan pada server yang sama.

## 11. Instalasi Docker
Pada panduan ini PowerDNS-Admin dijalankan menggunakan Docker karena dokumentasi resmi PowerDNS-Admin merekomendasikan Docker sebagai cara cepat untuk menjalankan aplikasi.

Install Docker:
```
apt install -y docker.io docker-compose-v2
```

Aktifkan Docker:
```
systemctl enable --now docker
```

Periksa versi:
```
docker --version
```
<p align="center">
<img width="837" height="128" alt="docker --version" src="https://github.com/user-attachments/assets/15df1076-7f29-4bcd-b4c3-60a38f63f4ee" />
  <br>
  <em>Gambar 13: versi Docker</em>
</p>

Periksa Docker Compose:
```
docker compose version
```
<p align="center">
<img width="836" height="109" alt="docker compose version" src="https://github.com/user-attachments/assets/84f08c70-1914-468f-92e3-22a94ec7afba" />
  <br>
  <em>Gambar 14: versi Docker Compose</em>
</p>

> **Catatan:** Perbedaan versi Docker yang muncul di terminal adalah hal wajar karena adanya pembaruan paket repositori. Yang terpenting, pastikan kedua perintah tersebut menampilkan informasi versi dengan sukses tanpa ada _error_.

## 12. Instalasi PowerDNS-Admin
Buat direktori untuk PowerDNS-Admin:
```
mkdir -p /opt/powerdns-admin
```

```
cd /opt/powerdns-admin
```
Membuat Kunci Rahasia (Secret Key) yang Kuat
```
openssl rand -hex 32
```
> **Catatan:** Salin (copy) hasil string acak yang dihasilkan untuk digunakan pada konfigurasi Docker Compose di bawah.

Buat file Docker Compose:
```
nano docker-compose.yml
```

Isi:
```
services:
  powerdns-admin:
    image: powerdnsadmin/pda-legacy:latest
    container_name: powerdns-admin
    restart: unless-stopped
    environment:
      SECRET_KEY: "ISI_DENGAN_HASIL_SECRETKEY_RANDOM_TADI"
      SQLALCHEMY_DATABASE_URI: "sqlite:////data/powerdns-admin.db"
    volumes:
      - pda-data:/data
    ports:
      - "9191:80"

volumes:
  pda-data:
```
<p align="center">
<img width="844" height="399" alt="File Docker Compose" src="https://github.com/user-attachments/assets/4182aebf-4534-4663-9646-19705527fcd1" />
  <br>
  <em>Gambar 15: File Docker Compose</em>
</p>

Jalankan container di latar belakang (detached mode) menggunakan Docker Compose:
```
docker compose up -d
```

Periksa container:
```
docker ps
```

Contoh:
```
CONTAINER ID   IMAGE                             STATUS
xxxxxxxxxxxx   powerdnsadmin/pda-legacy:latest   Up About a minute

```

> **Catatan:** PowerDNS-Admin menggunakan SQLite untuk database aplikasinya pada contoh ini. Database SQLite tersebut hanya menyimpan data PowerDNS-Admin, sedangkan data DNS tetap berada pada database PowerDNS/MariaDB.

## 13. Konfigurasi Koneksi PowerDNS API
Pada tahap ini PowerDNS-Admin berjalan dalam container, sedangkan PowerDNS API berjalan pada host Ubuntu.

Karena PowerDNS API dikonfigurasi pada `127.0.0.1`, container tidak dapat langsung menggunakan `127.0.0.1` untuk mengakses API host.

Agar PowerDNS-Admin dapat berkomunikasi dengan PowerDNS API, buat konfigurasi Docker agar container dapat mengakses host.

Edit file:
```
nano docker-compose.yml
```

Ubah menjadi:
```
services:
  powerdns-admin:
    image: powerdnsadmin/pda-legacy:latest
    container_name: powerdns-admin
    restart: unless-stopped
    environment:
      SECRET_KEY: "ISI_DENGAN_HASIL_SECRETKEY_RANDOM_YANG_PERTAMA"
      SQLALCHEMY_DATABASE_URI: "sqlite:////data/powerdns-admin.db"
    volumes:
      - pda-data:/data
    ports:
      - "9191:80"
    extra_hosts:
      - "host.docker.internal:host-gateway"

volumes:
  pda-data:
```

<p align="center">
<img width="841" height="399" alt="Konfigurasi Koneksi PowerDNS API di Docker" src="https://github.com/user-attachments/assets/0c3373d0-455b-47da-989a-e5f193408bb3" />
  <br>
  <em>Gambar 16: Konfigurasi Koneksi PowerDNS API di Docker</em>
</p>

Kemudian jalankan ulang:
```
docker compose down
```

```
docker compose up -d
```


## 14. Akses PowerDNS-Admin melalui Browser
Buka web browser kesayanganmu (seperti Google Chrome, Mozilla Firefox, atau Microsoft Edge), lalu masukkan alamat IP Public VPS-mu diikuti dengan port `9191`:

```
http://IP_SERVER:9191
```

Contoh:
```
http://103.xxx.xxx.xxx:9191
```


Halaman PowerDNS-Admin akan tampil.

<p align="center">
<img width="1366" height="700" alt="Tampilan PowerDNS Login" src="https://github.com/user-attachments/assets/01c34585-010d-4862-bded-f4e5b7919e5c" />
  <br>
  <em>Gambar 17: Tampilan PowerDNS Login</em>
</p>

> **Catatan:** Jika halaman web gagal diakses, pastikan port `9191` pada firewall VPS atau panel cloud provider (seperti Kilat VM 2.0) sudah diizinkan (allow) untuk koneksi TCP.

## 15. Membuat Akun Administrator
Pada halaman awal PowerDNS-Admin, pilih menu atau tautan untuk membuat akun baru (Register / Create Account).

<p align="center">
<img width="1366" height="700" alt="Menu Create Account" src="https://github.com/user-attachments/assets/fa0f18cc-120d-4348-925c-f13f80461bad" />
  <br>
  <em>Gambar 18: Menu Create Account</em>
</p>

Lalu isi data administrator dengan informasi berikut:
```
Username : admin
Email    : email@domainkamu.id
Password : PASSWORD_ADMIN
```
<p align="center">
<img width="1363" height="724" alt="Halaman Registrasi" src="https://github.com/user-attachments/assets/c55bf31a-19c2-46bc-adef-865b61ee809f" />
  <br>
  <em>Gambar 19: Halaman Registrasi</em>
</p>

> **Catatan:** Gunakan password minimal 12–16 karakter yang mengombinasikan huruf besar/kecil, angka, dan simbol, serta hindari kata yang mudah ditebak demi menjaga keamanan penuh panel DNS kamu.

Setelah akun dibuat, login menggunakan akun administrator tersebut.
<p align="center">
<img width="1364" height="665" alt="Halaman Login" src="https://github.com/user-attachments/assets/146665a1-2634-4f8b-8312-ae77f9125aaf" />
  <br>
  <em>Gambar 20: Halaman Login</em>
</p>

## 16. Konfigurasi PowerDNS pada PowerDNS-Admin
Pada halaman utama dasbor PowerDNS-Admin, arahkan pandangan ke menu navigasi (biasanya terletak di bagian atas atau bilah samping).

Pilih menu Settings (Pengaturan).

<p align="center">
<img width="1366" height="729" alt="Halaman Dashboard Terdapat Error" src="https://github.com/user-attachments/assets/f93f44e4-97bb-471e-ba5a-550893bbb1ce" />
  <br>
  <em>Gambar 21: Halaman Setting Terdapat Error</em>
</p>

Isi dengan:

PowerDNS API URL:
```
http://host.docker.internal:8081
```

PowerDNS API Key:
```
API_KEY_ANDA
```

Simpan konfigurasi.

<p align="center">
<img width="1366" height="724" alt="Simpan Konfigurasi" src="https://github.com/user-attachments/assets/bdc55cd4-d16e-4d01-9cfb-c50daf0b838c" />
  <br>
  <em>Gambar 22: Simpan Konfigurasi</em>
</p>

> **Catatan:** Jika integrasi sukses, pesan error koneksi tidak akan muncul. 

## 17. Membuat DNS Zone melalui GUI
Setelah PowerDNS-Admin berhasil terhubung dengan PowerDNS, pembuatan zone dapat dilakukan melalui GUI tanpa menjalankan perintah SQL.

Masuk ke menu:
```
Create Zones
```
<p align="center">
<img width="1366" height="725" alt="Menu Zones" src="https://github.com/user-attachments/assets/6658e31c-d96d-4b51-9b48-9ed66fc0ea99" />
  <br>
  <em>Gambar 23: Menu Zones</em>
</p>

Pada bagian Zone Name, masukkan nama domain yang ingin kamu kelola:
```
domainkamu.id
```
Biarkan pengaturan lainnya sesuai default (misalnya Zone Type menggunakan Native atau Primary).

<p align="center">
<img width="1366" height="691" alt="Membuat DNS Zone melalui PowerDNS-Admin" src="https://github.com/user-attachments/assets/43b4a0dd-b667-4f03-baa0-684f9ec93a3a" />
  <br>
</p>

<p align="center">
<img width="1366" height="721" alt="Membuat DNS Zone melalui PowerDNS-Admin 2" src="https://github.com/user-attachments/assets/0de5c617-b1e7-48a8-bce9-fc02a45b12c5" />
  <br>
  <em>Gambar 24: Membuat DNS Zone melalui PowerDNS-Admin</em>
</p>

Klik tombol `Create Zone` di bagian bawah.

Setelah zone berhasil dibuat, domain akan otomatis muncul pada daftar zone (Zone List).

<p align="center">
<img width="1366" height="722" alt="DNS Zone yang berhasil dibuat" src="https://github.com/user-attachments/assets/7294ab2d-0d9e-4a86-a70f-a00d4d32ed82" />
  <br>
  <em>Gambar 25: DNS Zone yang berhasil dibuat</em>
</p>

## 18. Menambahkan DNS Record melalui GUI
Pada tampilan dashboard sudah ada daftar zone (domain) yang sudah dibuat sebelumnya. 

Klik nama domain tersebut, kemudian akan masuk pada pengaturan DNS Record domain anda.

Lakukan penambahan dengan klik `Add Record`:

<p align="center">
<img width="1365" height="296" alt="Klik Add Record" src="https://github.com/user-attachments/assets/94cb75d2-e7fd-49bf-9e14-e139acc669c0" />
  <br>
  <em>Gambar 26: Klik Add Record</em>
</p>

### Nameserver (NS)
Tambahkan NS untuk domain utama.
```
ns1.domainkamu.id 
ns2.domainkamu.id
```

### Record A
Tambahkan record A untuk domain utama.
```
Type : A 
Name : domainkamu.id 
Content : IP_SERVER 
TTL : 3600
```

Tambahkan juga record A untuk nameserver:
```
Type : A 
Name : ns1.domainkamu.id 
Content : IP_SERVER 
TTL : 3600

Type : A 
Name : ns2.domainkamu.id 
Content : IP_SERVER 
TTL : 3600
```

### CNAME
Sebagai contoh, tambahkan record CNAME untuk subdomain `www`.
```
Type : CNAME 
Name : www.domainkamu.id 
Content : domainkamu.id 
TTL : 3600
```

## 19. Memeriksa DNS Record melalui PowerDNS-Admin
Setelah seluruh record ditambahkan, periksa kembali daftar record.
```
SOA domainkamu.id 
NS domainkamu.id 
NS domainkamu.id 
A domainkamu.id 
A ns1.domainkamu.id 
A ns2.domainkamu.id 
CNAME www.domainkamu.id 
TXT domainkamu.id
```
<p align="center">
<img width="1366" height="719" alt="Daftar DNS Record pada PowerDNS-Admin" src="https://github.com/user-attachments/assets/ca8bf376-fe2b-4bfc-8fd6-0ff3b32d3515" />
  <br>
  <em>Gambar 27: Daftar DNS Record pada PowerDNS-Admin</em>
</p>

> **Catatan:** Nama record yang tampil dapat berbeda tergantung zone dan record yang dibuat saat praktik.

## 20. Memeriksa Service 
Setelah konfigurasi selesai, periksa kembali service PowerDNS:
```
#systemctl status pdns
● pdns.service - PowerDNS Authoritative Server
     Loaded: loaded (/usr/lib/systemd/system/pdns.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-09-20 17:00:48 WIB; 4h 16min ago
       Docs: man:pdns_server(1)
             man:pdns_control(1)
             https://doc.powerdns.com
   Main PID: 23371 (pdns_server)
      Tasks: 10 (limit: 1094)
     Memory: 50.7M (peak: 59.5M)
        CPU: 1.869s
     CGroup: /system.slice/pdns.service
             └─23371 /usr/sbin/pdns_server --guardian=no --daemon=no --disable-syslog --log-timestamp=no --write-pid=no
```

Periksa port DNS:
```
#ss -lntup | grep :53
udp   UNCONN 0      0            0.0.0.0:53         0.0.0.0:*    users:(("pdns_server",pid=23371,fd=5))
udp   UNCONN 0      0               [::]:53            [::]:*    users:(("pdns_server",pid=23371,fd=6))
tcp   LISTEN 0      128          0.0.0.0:53         0.0.0.0:*    users:(("pdns_server",pid=23371,fd=7))
tcp   LISTEN 0      128             [::]:53            [::]:*    users:(("pdns_server",pid=23371,fd=8))
```

Periksa port API:
```
#ss -lntup | grep :8081
tcp   LISTEN 0      10           0.0.0.0:8081       0.0.0.0:*    users:(("pdns_server",pid=23371,fd=9))
```

## 21. Verifikasi DNS Menggunakan dig
Pengujian pertama dilakukan dari server menggunakan `dig`.

### Verifikasi Record A
Jalankan:
```
dig domainkamu.id A
```

Kemudian:
```
dig www.domainkamu.id A
```

Jika ingin melihat hasil yang lebih singkat:
```
dig +short domainkamu.id A
```

```
dig +short www.domainkamu.id A
```
> **Hasil yang diharapkan:** Perintah `dig` menampilkan IP Address VPS yang telah dimasukkan pada record A.

### Verifikasi NS Record
Selanjutnya lakukan pengecekan nameserver:
```
dig domainkamu.id NS
```

Hasil yang diharapkan menampilkan:
```
ns1.domainkamu.id. 
ns2.domainkamu.id.
```
> **Catatan:** Pengujian ini dilakukan untuk memastikan zone memiliki nameserver sesuai konfigurasi.

### Pengujian Resolusi DNS melalui PowerDNS
Untuk memastikan PowerDNS memberikan respons terhadap query DNS, jalankan:
```
dig @127.0.0.1 domainkamu.id A
```

Kalau PowerDNS menerima query dan zone sudah benar, pada bagian **ANSWER SECTION** akan muncul record A yang telah dibuat.

Contohnya:
```
;; ANSWER SECTION:
domainkamu.id.    3600    IN    A    IP_PUBLIC_VPS
```

> **Hasil yang diharapkan:** PowerDNS berhasil memberikan jawaban DNS berdasarkan record yang tersimpan pada zone.

### Pengujian melalui DNS Resolver
Setelah pengujian lokal berhasil, lakukan pengujian menggunakan resolver DNS dengan:
```
dig @8.8.8.8 domainkamu.id A
```

atau:
```
dig @1.1.1.1 domainkamu.id A
```

> **Hasil yang diharapkan:** Jika domain dan nameserver sudah terdelegasi dengan benar, resolver akan mendapatkan record A dari domain tersebut.



## 22. Pengujian menggunakan DNS Checker
Apabila hasil *output* IP Address sudah mengarah ke IP Address *server* yang digunakan, maka hasil *pointing* domain sudah *resolved*.

Selain itu, Anda juga dapat memeriksa hasil *pointing* lebih lanjut menggunakan *tools* berbasis web seperti [DNS Checker](https://dnschecker.org/). Jika sudah resolved semua, maka akan ditandai dengan centang warna hijau secara keseluruhan pada tool DNS Checker seperti pada Gambar 10.

<p align="center">
<img width="1163" height="645" alt="dns checker" src="https://github.com/user-attachments/assets/6984dd71-721e-49a7-8e96-056246bc5022" />
  <br>
  <em>Gambar 28: DNS CHECKER</em>
</p>

Apabila dari hasil verifikasi, hasil pointing domain masih belum mengarah ke IP Address server yang digunakan atau masih belum terdapat tanda centang hijau secara keseluruhan pada tool DNS Checker, biasanya hal tersebut masih berada dalam proses propagasi.

> _Propagasi adalah waktu yang dibutuhkan oleh internet atau ISP untuk mengenali record-record DNS yang baru pada sebuah domain (biasanya dibutuhkan ketika terjadi perubahan pada record-record DNS). Pada saat proses propagasi berlangsung, domain terkadang akan mengalami anomali ketika diakses._
>
>> _Proses propagasi ini dipengaruhi oleh beberapa faktor, yaitu pengaturan TTL (Time to Live), jaringan ISP, serta pihak Registry domain. Waktu yang dibutuhkan untuk proses propagasi ini biasanya memakan waktu kurang lebih hingga 48 jam._

## 23. Troubleshooting
### PowerDNS-Admin menampilkan 400 Bad Request saat menambahkan Zone atau Record
Jika **PowerDNS-Admin** menampilkan `400 Bad Request` saat menambahkan zone atau record, periksa log:
```
docker logs --tail 50 powerdns-admin 2>&1
```

Jika terdapat pesan:
```
Connection to host.docker.internal timed out
```

berarti PowerDNS-Admin masih mencoba mengakses PowerDNS API melalui `host.docker.internal:8081`.

#### Periksa konfigurasi
Cek file konfigurasi PowerDNS:
```
nano /etc/powerdns/pdns.conf
```

Pastikan API aktif:
```
api=yes
api-key=API_KEY_ANDA
webserver=yes
webserver-address=0.0.0.0
webserver-port=8081
```

Kemudian cek konfigurasi API URL pada PowerDNS-Admin:
```
docker exec powerdns-admin python -c "import sqlite3; c=sqlite3.connect('/data/powerdns-admin.db'); print(c.execute(\"SELECT name,value FROM setting WHERE name='pdns_api_url'\").fetchone()); c.close()"
```

Jika masih menggunakan:
```
http://host.docker.internal:8081
```

ubah menjadi:
```
http://172.18.0.1:8081
```

dengan perintah:
```
docker exec powerdns-admin python -c "import sqlite3; c=sqlite3.connect('/data/powerdns-admin.db'); c.execute(\"UPDATE setting SET value='http://172.18.0.1:8081' WHERE name='pdns_api_url'\"); c.commit(); c.close()"
```

Restart PowerDNS-Admin:
```
cd /opt/powerdns-admin
docker compose restart
```

#### Periksa UFW
Jika masih mengalami timeout, izinkan koneksi dari container ke API:
```
ufw allow in on br-cb447a1ab8dd from 172.18.0.2 to 172.18.0.1 port 8081 proto tcp
```

Setelah itu, coba kembali menambahkan zone atau record melalui PowerDNS-Admin.

## Kesimpulan
Berdasarkan pengujian yang dilakukan, **PowerDNS** berhasil digunakan untuk mengelola **zone melalui PowerDNS-Admin**. Zone dapat ditambahkan dan record DNS seperti A dan NS dapat dikonfigurasi melalui antarmuka PowerDNS-Admin.

***
CloudKilat menyediakan layanan **Kilat VM, hosting, serta berbagai layanan pendukung lainnya** dengan performa yang andal. Layanan CloudKilat juga didukung oleh tim support yang siap membantu dengan respons cepat dan pelayanan selama **7x24 jam**.

Untuk informasi lebih lanjut mengenai layanan CloudKilat, silakan kunjungi [website resmi CloudKilat](https://cloudkilat.id/).

Terima kasih, semoga panduan ini bermanfaat.

## Referensi

* [PowerDNS Official Website](https://www.powerdns.com/)
* [PowerDNS GitHub Releases](https://github.com/PowerDNS/pdns)
* [KB - Cara Menggunakan Private Name Server pada Domain di Portal Client CloudKilat](https://kb.cloudkilat.id/domain-di-cloudkilat/cara-menggunakan-private-name-server-pada-domain-di-portal-client-cloudkilat)
