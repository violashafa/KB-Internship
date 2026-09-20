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
<img width="903" height="97" alt="impor schema" src="https://github.com/user-attachments/assets/f6e61714-468c-432e-98b9-e5a4bd888930" />
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
<img width="577" height="261" alt="show tables" src="https://github.com/user-attachments/assets/8c09928b-5e50-4a67-a4d1-23e9f79fa3ba" />
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
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
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
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
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
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
  <br>
  <em>Gambar 13: versi Docker</em>
</p>

Periksa Docker Compose:
```
docker compose version
```
<p align="center">
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
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
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
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
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
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
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
  <br>
  <em>Gambar 17: Tampilan PowerDNS Login</em>
</p>

> **Catatan:** Jika halaman web gagal diakses, pastikan port `9191` pada firewall VPS atau panel cloud provider (seperti Kilat VM 2.0) sudah diizinkan (allow) untuk koneksi TCP.

## 15. Membuat Akun Administrator
Pada halaman awal PowerDNS-Admin, pilih menu atau tautan untuk membuat akun baru (Register / Create Account).

<p align="center">
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
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
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
  <br>
  <em>Gambar 19: Halaman Registrasi</em>
</p>

> **Catatan:** Gunakan password minimal 12–16 karakter yang mengombinasikan huruf besar/kecil, angka, dan simbol, serta hindari kata yang mudah ditebak demi menjaga keamanan penuh panel DNS kamu.

Setelah akun dibuat, login menggunakan akun administrator tersebut.
<p align="center">
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
  <br>
  <em>Gambar 20: Halaman Login</em>
</p>

## 16. Konfigurasi PowerDNS pada PowerDNS-Admin
Pada halaman utama dasbor PowerDNS-Admin, arahkan pandangan ke menu navigasi (biasanya terletak di bagian atas atau bilah samping).

Pilih menu Settings (Pengaturan).

<p align="center">
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
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
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
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
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
  <br>
  <em>Gambar 23: Menu Zones</em>
</p>

Masukkan nama domain:
```
domainkamu.id
```

Kemudian masukkan nameserver:
```
ns1.domainkamu.id 
ns2.domainkamu.id
```

<p align="center">
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
  <br>
  <em>Gambar 24: Membuat DNS Zone melalui PowerDNS-Admin</em>
</p>

Simpan zone.

Setelah zone berhasil dibuat, domain akan muncul pada daftar zone.
<p align="center">
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
  <br>
  <em>Gambar 25: DNS Zone yang berhasil dibuat</em>
</p>

## 18. Menambahkan DNS Record melalui GUI
### Record A
### CNAME
### TXT
## 19. Memeriksa DNS Record melalui PowerDNS-Admin
## 20. Memeriksa Service PowerDNS
## 21. Verifikasi DNS Menggunakan dig
### Verifikasi SOA
### Verifikasi NS Record
### Verifikasi Record A
### Pengujian melalui Public IP
### Pengujian dari DNS Resolver Publik
## 22. Pengujian menggunakan DNS Checker
## 23. Troubleshooting
### PowerDNS gagal berjalan karena port 53 digunakan
### PowerDNS-Admin tidak dapat terhubung ke PowerDNS API
### PowerDNS API tidak aktif
### DNS Record tidak muncul
### DNS belum dapat diakses dari internet
## Kesimpulan
## Referensi
