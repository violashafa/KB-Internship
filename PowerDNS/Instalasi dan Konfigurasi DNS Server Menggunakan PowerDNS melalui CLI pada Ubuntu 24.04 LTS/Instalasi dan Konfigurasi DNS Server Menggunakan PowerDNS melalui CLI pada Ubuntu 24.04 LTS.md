


# Instalasi dan Konfigurasi DNS Server Menggunakan PowerDNS melalui CLI pada Ubuntu 24.04 LTS.

Pelajari cara instalasi dan konfigurasi DNS Server menggunakan BIND di Kilat VM 2.0 berbasis Ubuntu 18.04. Panduan lengkap mulai dari persiapan, setup Glue Record, hingga konfigurasi zona domain.
***

Halo, Kawan Belajar!

DNS atau Domain Name System merupakan sistem yang digunakan untuk menerjemahkan nama domain menjadi IP Address. Dengan adanya DNS, pengguna dapat mengakses layanan menggunakan nama domain tanpa harus mengingat IP Address dari server.

Salah satu aplikasi yang dapat digunakan sebagai DNS Server adalah **PowerDNS**. PowerDNS merupakan DNS Server yang digunakan untuk menangani permintaan DNS dan mengelola informasi domain melalui DNS Zone dan DNS Record.

Pada panduan kali ini, akan dilakukan instalasi dan konfigurasi **PowerDNS pada Kilat VM 2.0** dengan sistem operasi Ubuntu. Konfigurasi dilakukan menggunakan domain sendiri dengan nameserver ns1 dan ns2 sehingga diperlukan Glue Record.

> 💡 **Catatan Kawan Belajar:** 
> 
> Sebelum kita masuk ke langkah-langkah instalasi teknis, pastikan kamu sudah memahami dasar-dasar serta alur kerjanya melalui artikel pengenalan sebelumnya:
> 
> 👉 [Apa Itu PowerDNS? Mengenal DNS Server Authoritative yang Fleksibel](https://www.cloudkilat.com/layanan/kilat-domain)

## 1. Persiapan

Untuk melakukan instalasi dan konfigurasi DNS Server menggunakan PowerDNS, beberapa kebutuhan yang perlu disiapkan antara lain:
* Domain dan Kilat VM 2.0 aktif.
* Sistem operasi Ubuntu 24.04.
* Setup Glue Record.
* IP Address publik pada Kilat VM.
  
Jika belum memiliki Kilat VM 2.0, pengguna dapat melakukan pemesanan layanan Kilat VM 2.0 melalui CloudKilat.

## Informasi Versi Sistem
Panduan ini menggunakan komponen dan versi perangkat lunak berikut:

| Komponen  | Versi |
| ------------- |:-------------:|
| Sistem Operasi      | Ubuntu 24.04 LTS     |
| PowerDNS Server      | 4.8.3 (Package Ubuntu 24.04)     |
| Database Backend      | MariaDB Server     |
| DNS Utilities      | `dig`     |

> **Catatan:** Versi PowerDNS dapat berubah seiring adanya *release* terbaru. Versi yang digunakan pada panduan ini adalah PowerDNS 4.8.3

## 2. Setup Glue Record
Sebelum melakukan konfigurasi PowerDNS, pastikan domain telah memiliki **Glue Record** apabila menggunakan nameserver sendiri.

Glue Record merupakan informasi IP Address yang digunakan oleh suatu nameserver dan didaftarkan pada registrar domain.


Adapun panduan cara setup Glue Record seperti berikut ini:

1. [Login Portal Client Area CloudKilat](https://portal.cloudkilat.com/clientarea) terlebih dahulu.
2. Untuk langkah-langkah lengkapnya, Anda dapat mengikuti panduan resmi melalui tautan [Cara Menggunakan Private Name Server pada Domain di Portal Client CloudKilat](https://kb.cloudkilat.id/domain-di-cloudkilat/cara-menggunakan-private-name-server-pada-domain-di-portal-client-cloudkilat).
## 3. Akses SSH & Update Sistem
Selanjutya, untuk melakukan instalasi dan konfigurasi DNS Server, silakan mengikuti langkah-langkah berikut ini:

Masuk ke Kilat VM 2.0 Anda terlebih dahulu, atau Anda juga bisa melakukan *remote* menggunakan SSH. Jika Anda masih belum mengetahui cara *remote* menggunakan SSH, silakan membaca panduannya melalui tautan [Cara Akses Kilat VM Melalui SSH](https://kb.cloudkilat.id/akses-kilat-vm/cara-akses-kilat-vm-melalui-ssh).

Setelah berhasil masuk ke dalam Kilat VM 2.0, perbarui paket sistem ke versi terbaru dengan menjalankan perintah berikut seperti pada Gambar 1:

```
apt update -y
```

<p align="center">
  <img width="1352" height="555" alt="apt update" src="https://github.com/user-attachments/assets/5fa107d5-5933-4f65-b352-50bca91f8a1e" />
  <br>
  <em>Gambar 1: Update Paket Ubuntu Server</em>
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
  <em>Gambar 2: Konfig DNS Stub</em>
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

## 5. Instalasi PowerDNS

Tunggu proses update hingga benar-benar selesai, dan selanjutnya install paket PowerDNS menggunakan perintah : 

```
apt install pdns-server
```

tekan **Y** apabila diminta untuk melanjutkan proses instalasi seperti pada Gambar 3:
<p align="center">
  <img width="1202" height="241" alt="install powerdns (y)" src="https://github.com/user-attachments/assets/222dbb7f-2e98-4efb-ae31-1da58e6d8bd5" />
  <br>
  <em>Gambar 3: Instalasi PowerDNS</em>
</p>

Setelah proses instalasi selesai, Anda dapat memeriksa versi PowerDNS yang terinstal menggunakan perintah sebagai berikut:

```
pdns_server --version
```
<p align="center">
  <img width="1360" height="347" alt="versi powerdns" src="https://github.com/user-attachments/assets/8f2ffb58-6ae8-4c6c-9759-8e52315b8220" />
  <br>
  <em>Gambar 4: Versi PowerDNS</em>
</p>

> **Catatan:** Versi PowerDNS dapat berubah seiring adanya *release* terbaru.

Kemudian, aktifkan dan pastikan service PowerDNS berjalan dengan baik menggunakan perintah status berikut, dan pastikan statusnya bernilai active (running):

```
#systemctl status pdns
● pdns.service - PowerDNS Authoritative Server
     Loaded: loaded (/usr/lib/systemd/system/pdns.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-09-12 21:36:42 WIB; 24h ago
       Docs: man:pdns_server(1)
             man:pdns_control(1)
             https://doc.powerdns.com
   Main PID: 329879 (pdns_server)
      Tasks: 8 (limit: 1094)
     Memory: 47.9M (peak: 48.2M)
        CPU: 4.323s
     CGroup: /system.slice/pdns.service
             └─329879 /usr/sbin/pdns_server --guardian=no --daemon=no --disable-syslog --log-timestamp=no>
```

## 6. Instalasi Database Backend

PowerDNS dapat menggunakan berbagai jenis *backend* untuk menyimpan data DNS. Pada praktik ini, digunakan MariaDB sebagai *database backend*.

Instal MariaDB dengan menjalankan perintah berikut:
```
apt install mariadb-server
```
tekan **Y** apabila diminta untuk melanjutkan proses instalasi seperti pada Gambar 5:
<p align="center">
<img width="1359" height="387" alt="install mariadb" src="https://github.com/user-attachments/assets/fb97b494-bee4-4303-a91a-c1daa909f4b0" />
  <br>
  <em>Gambar 5: Instal MariaDB</em>
</p>

Setelah instalasi selesai, cek service MariaDB untuk memastikan berjalan dengan normal:
```
#systemctl status mariadb
● mariadb.service - MariaDB 10.11.14 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-09-12 12:16:49 WIB; 1 day 10h ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
   Main PID: 293006 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 10 (limit: 7226)
     Memory: 93.7M (peak: 94.9M)
        CPU: 24.211s
     CGroup: /system.slice/mariadb.service
             └─293006 /usr/sbin/mariadbd
```

Selanjutnya, install backend MariaDB untuk PowerDNS menggunakan perintah berikut:
```
apt install pdns-backend-mysql
```
<p align="center">
<img width="1366" height="523" alt="install backend mariadb" src="https://github.com/user-attachments/assets/39bc39ba-dc1a-4e45-bb25-965188e1aa20" />
  <br>
  <em>Gambar 6: Instal Backend MariaDB</em>
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
  <em>Gambar 7: Impor File</em>
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
  <em>Gambar 8: Show Tables</em>
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
gmysql-user=powerdns
gmysql-password=password
gmysql-dbname=powerdns
```
<p align="center">
<img width="1366" height="287" alt="isi nano" src="https://github.com/user-attachments/assets/afff038b-ad69-4d23-b8ff-5fa716e9ff36" />
  <br>
  <em>Gambar 9: File Pdns</em>
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
     Active: active (running) since Sat 2026-09-12 21:36:42 WIB; 1 day 1h ago
       Docs: man:pdns_server(1)
             man:pdns_control(1)
             https://doc.powerdns.com
   Main PID: 329879 (pdns_server)
      Tasks: 8 (limit: 1094)
     Memory: 47.9M (peak: 48.2M)
        CPU: 4.611s
     CGroup: /system.slice/pdns.service
             └─329879 /usr/sbin/pdns_server --guardian=no --daemon=no --disable-syslog --log-timestamp=no>
```
> **Catatan:** Pastikan statusnya menunjukkan keterangan active (running):

## 10. Membuat DNS Zone

Setelah *backend* berhasil dikonfigurasi, langkah selanjutnya adalah membuat DNS *Zone* untuk domain yang akan digunakan.

*Login* kembali ke *database* PowerDNS menggunakan perintah berikut:
```
mysql -u powerdns -p powerdns
```

Kemudian buat zone domain baru dengan memasukkan query SQL berikut
```
INSERT INTO domains (name, type) VALUES ('domainkamu.id', 'NATIVE');
```
> **Catatan:** `domainkamu.id` hanya digunakan sebagai contoh. Silakan sesuaikan dengan domain dan IP Address yang digunakan. 

Setelah itu, cek apakah zone tersebut sudah berhasil tersimpan dengan menjalankan perintah:
```
SELECT * FROM domains;
```
<p align="center">
<img width="980" height="130" alt="show domain" src="https://github.com/user-attachments/assets/c0fa8c6e-057b-440e-a7ea-0e8dd86ccf05" />
  <br>
  <em>Gambar 9: Show Domain</em>
</p>

## 11. Menambahkan DNS Record

Setelah DNS *Zone* berhasil dibuat, langkah berikutnya adalah menambahkan berbagai macam DNS *Record* yang diperlukan ke dalam *database*.
#### A Record
*A Record* digunakan untuk mengarahkan domain utama ke alamat IP *server* Anda:
```
INSERT INTO records (domain_id, name, type, content, ttl) 
VALUES (1, 'domainkamu.id', 'A', 'IP_SERVER', 3600);
```

#### NS Record
Tambahkan nameserver yang akan digunakan oleh domain Anda:
```
INSERT INTO records (domain_id, name, type, content, ttl) 
VALUES 
(1, 'domainkamu.id', 'NS', 'ns1.domainkamu.id', 3600),
(1, 'domainkamu.id', 'NS', 'ns2.domainkamu.id', 3600);
```

#### A Record untuk Nameserver
Karena nameserver yang digunakan merupakan child nameserver, tambahkan A Record untuk masing-masing nameserver tersebut:
```
INSERT INTO records (domain_id, name, type, content, ttl) 
VALUES 
(1, 'ns1.domainkamu.id', 'A', 'IP_SERVER', 3600),
(1, 'ns2.domainkamu.id', 'A', 'IP_SERVER', 3600);
```

#### CNAME Record
Jika Anda ingin mengarahkan subdomain www ke domain utama, tambahkan CNAME Record berikut:
```
INSERT INTO records (domain_id, name, type, content, ttl) 
VALUES 
(1, 'www.domainkamu.id', 'CNAME', 'domainkamu.id', 3600);
```
> **Catatan:** `domainkamu.id` hanya digunakan sebagai contoh. Silakan sesuaikan dengan domain dan IP Address yang digunakan.


## 12. Mengecek DNS Zone dan Record
Setelah seluruh *record* ditambahkan ke dalam *database*, langkah terakhir adalah memeriksa kembali seluruh data yang telah dibuat untuk memastikan semuanya sudah terkonfigurasi dengan benar.

Jalankan *query* SQL berikut di dalam MariaDB:
```
SELECT name, type, content, ttl FROM records;
```

Pastikan record yang muncul sudah sesuai dengan konfigurasi.
```
domainkamu.id          A       IP_SERVER
domainkamu.id          NS      ns1.domainkamu.id
domainkamu.id          NS      ns2.domainkamu.id
ns1.domainkamu.id      A       IP_SERVER
ns2.domainkamu.id      A       IP_SERVER
www.domainkamu.id      CNAME   domainkamu.id
```
> **Catatan:** Ganti `IP_SERVER` dengan IP Address publik Kilat VM yang digunakan. Pastikan nilai `ns1`, `ns2`, `www`, dan domain utama disesuaikan dengan kebutuhan konfigurasi DNS Anda.

Pada konfigurasi tersebut terdapat beberapa DNS *record*:

| Record / Entri | Fungsi |
| :--- | :--- |
| domainkamu.id  | Mengarahkan domain utama ke IP Address *server* |
| domainkamu.id NS | Menentukan *nameserver* yang bertanggung jawab terhadap domain |
| ns1.domainkamu.id A | Mengarahkan *nameserver* pertama ke IP Address VPS |
| ns2.domainkamu.id A | Mengarahkan *nameserver* kedua ke IP Address VPS |
| www.domainkamu.id CNAME | Mengarahkan `www.domainkamu.id` agar merujuk ke domain utama (`domainkamu.id`) |

## Restart PowerDNS
Setelah seluruh konfigurasi dan penambahan record selesai, lakukan restart terakhir pada service PowerDNS untuk menerapkan semua pembaruan secara sempurna, lalu pastikan kembali bahwa service telah berjalan dengan normal dan stabil:
```
#systemctl restart pdns
#systemctl status pdns
● pdns.service - PowerDNS Authoritative Server
     Loaded: loaded (/usr/lib/systemd/system/pdns.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-09-12 21:36:42 WIB; 7s ago
       Docs: man:pdns_server(1)
             man:pdns_control(1)
             https://doc.powerdns.com
   Main PID: 329879 (pdns_server)
      Tasks: 8 (limit: 1094)
     Memory: 47.8M (peak: 48.0M)
        CPU: 171ms
     CGroup: /system.slice/pdns.service
```

## 13. Verifikasi
## Mengecek Port DNS
Layanan DNS menggunakan port `53` (baik protokol UDP maupun TCP) untuk menerima setiap *query* atau permintaan DNS yang masuk dari klien.

Untuk mengecek apakah port tersebut sudah digunakan dan aktif oleh PowerDNS, jalankan perintah berikut di terminal:
```
#ss -lntup | grep :53
udp   UNCONN 0      0            0.0.0.0:53        0.0.0.0:*    users:(("pdns_server",pid=329879,fd=5))
udp   UNCONN 0      0               [::]:53           [::]:*    users:(("pdns_server",pid=329879,fd=6))
tcp   LISTEN 0      128          0.0.0.0:53        0.0.0.0:*    users:(("pdns_server",pid=329879,fd=7))
tcp   LISTEN 0      128             [::]:53           [::]:*    users:(("pdns_server",pid=329879,fd=8))
```

## Pengujian DNS Server Menggunakan dig

Setelah layanan PowerDNS aktif dan seluruh konfigurasi selesai, lakukan pengujian fungsionalitas DNS menggunakan utilitas `dig`.

Pertama, lakukan pengujian *query* terhadap DNS yang telah dibuat dengan perintah:
#### Cek DNS Zone
```
#dig @IP_SERVER domainkamu.id
```
Jika berhasil, akan muncul bagian **ANSWER SECTION** yang berisi IP Address domain.

#### Cek NS Record
```
#dig @IP_SERVER domainkamu.id NS
domainkamu.id.    NS    ns1.domainkamu.id.
domainkamu.id.    NS    ns2.domainkamu.id.
```
#### Pengujian Nameserver dari Domain
```
#dig domainkamu.id NS
ns1.domainkamu.id
ns2.domainkamu.id
```

#### Pengujian Domain
```
#dig domainkamu.id +short
IP_SERVER
```

## Verifikasi Menggunakan DNS Checker
Apabila hasil *output* IP Address sudah mengarah ke IP Address *server* yang digunakan, maka hasil *pointing* domain sudah *resolved*.

Selain itu, Anda juga dapat memeriksa hasil *pointing* lebih lanjut menggunakan *tools* berbasis web seperti [DNS Checker](https://dnschecker.org/). Jika sudah resolved semua, maka akan ditandai dengan centang warna hijau secara keseluruhan pada tool DNS Checker seperti pada Gambar 10.

<p align="center">
<img width="1163" height="645" alt="dns checker" src="https://github.com/user-attachments/assets/6984dd71-721e-49a7-8e96-056246bc5022" />
  <br>
  <em>Gambar 10: DNS CHECKER</em>
</p>

Apabila dari hasil verifikasi, hasil pointing domain masih belum mengarah ke IP Address server yang digunakan atau masih belum terdapat tanda centang hijau secara keseluruhan pada tool DNS Checker, biasanya hal tersebut masih berada dalam proses propagasi.

> _Propagasi adalah waktu yang dibutuhkan oleh internet atau ISP untuk mengenali record-record DNS yang baru pada sebuah domain (biasanya dibutuhkan ketika terjadi perubahan pada record-record DNS). Pada saat proses propagasi berlangsung, domain terkadang akan mengalami anomali ketika diakses._
>
>> _Proses propagasi ini dipengaruhi oleh beberapa faktor, yaitu pengaturan TTL (Time to Live), jaringan ISP, serta pihak Registry domain. Waktu yang dibutuhkan untuk proses propagasi ini biasanya memakan waktu kurang lebih hingga 48 jam._

## 14.  Troubleshooting

### PowerDNS tidak berjalan
Jika *service* PowerDNS gagal berjalan atau mengalami *stop*, periksa status *service* terlebih dahulu:
```
systemctl status pdns
````

> **Verifikasi:** Pastikan status *service* menunjukkan *active (running)*. Jika terdapat galat *(error)*, periksa detail log *service* menggunakan perintah:
>

```
journalctl -u pdns -n 50 --no-pager
````

### PowerDNS gagal terhubung ke 
Jika PowerDNS tidak dapat mengakses atau terhubung ke database MariaDB, periksa file konfigurasi utama PowerDNS:
```
nano /etc/powerdns/pdns.conf
````

Pastikan parameter koneksi database berikut sudah terkonfigurasi dengan benar:
```
launch=gmysql
gmysql-host=127.0.0.1
gmysql-user=powerdns
gmysql-password=PASSWORD_ANDA
gmysql-dbname=powerdns
````

> **Catatan:** Ganti PASSWORD_ANDA dengan kata sandi yang sesuai dengan konfigurasi user database Anda.
>

Setelah memastikan konfigurasi sudah benar, restart service PowerDNS:
```
systemctl restart pdns
````

> **Verifikasi:** Cek kembali status PowerDNS untuk memastikan ia berhasil berjalan dengan backend database.
>

### Port *53* Sudah Digunakan
Jika port *53* sudah digunakan oleh layanan lain (seperti `systemd-resolved`), PowerDNS tidak akan bisa berjalan. Periksa penggunaan port *53* pada server:
```
ss -lntup | grep ':53'
````

> **Verifikasi:** Jika port *53* terpakai oleh proses lain, nonaktifkan DNS Stub Listener pada `systemd-resolved` melalui file `/etc/systemd/resolved.conf` dengan mengubah baris `DNSStubListener=no`, lalu restart service terkait.
>

### DNS tidak memberikan response
Jika DNS tidak dapat diakses atau tidak memberikan respon dari luar:
1. Lakukan pengujian query langsung ke IP server:
```
dig @IP_SERVER domainkamu.id
````

2. Pastikan hal-hal berikut dalam kondisi normal:
    * Status service PowerDNS dalam keadaan active (running).
    * PowerDNS mendengarkan (listen) pada port 53.
    * Port 53 (TCP/UDP) sudah dibuka pada firewall.
    * Konfigurasi DNS Zone, DNS Record, database, dan nameserver sudah sesuai.

### Domain belum dapat diakses
Jika domain masih belum dapat diakses atau perubahan record belum terlihat:
1. Periksa nameserver domain:
```
dig domainkamu.id NS
````

2. Periksa A Record domain:
```
dig domainkamu.id A
````

> **Verifikasi:** Pastikan hasil output sudah mengarah ke alamat IP publik Kilat VM Anda.
>

Jika konfigurasi sudah dipastikan benar tetapi perubahan belum terlihat dari jaringan luar atau resolver tertentu, kemungkinan besar domain Anda masih berada dalam proses propagasi (waktu yang dibutuhkan oleh internet/ISP untuk mengenali perubahan record DNS yang baru, yang biasanya membutuhkan waktu hingga 48 jam).

## Kesimpulan
Dengan memahami konsep dan cara kerja PowerDNS, kamu bisa membangun layanan DNS Authoritative pada VPS secara lebih fleksibel dan terkelola. Dengan dukungan MariaDB sebagai backend, konfigurasi zone dan record DNS dapat disimpan serta dikelola dengan lebih terstruktur sesuai kebutuhan.

Setelah PowerDNS berhasil dikonfigurasi, kamu dapat mengelola domain, nameserver, serta DNS record melalui database dan melakukan pengecekan untuk memastikan layanan DNS berjalan dengan baik.

## Referensi

* [PowerDNS Official Website](https://www.powerdns.com/)
* [PowerDNS GitHub Releases](https://github.com/PowerDNS/pdns)
* [KB - Cara Menggunakan Private Name Server pada Domain di Portal Client CloudKilat](https://kb.cloudkilat.id/domain-di-cloudkilat/cara-menggunakan-private-name-server-pada-domain-di-portal-client-cloudkilat)

Sekian, dan semoga bermanfaat.
