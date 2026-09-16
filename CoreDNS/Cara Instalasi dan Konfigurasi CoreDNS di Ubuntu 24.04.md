# Cara Instalasi dan Konfigurasi CoreDNS di Ubuntu 24.04

Halo, Kawan Belajar! Pernah membutuhkan DNS server sendiri untuk mengelola domain pada Kilat VM?

Pada panduan ini, kita akan membahas cara menginstal dan mengonfigurasi **CoreDNS** pada Ubuntu 24.04 LTS hingga dapat digunakan sebagai **authoritative DNS server** untuk mengelola DNS record pada domain.

## Apa Itu CoreDNS?

CoreDNS merupakan DNS server **open source** yang digunakan untuk menangani permintaan DNS (*Domain Name System*). CoreDNS menggunakan sistem berbasis plugin sehingga dapat dikonfigurasi sesuai dengan kebutuhan, seperti melayani DNS record, melakukan DNS forwarding, dan menyediakan layanan DNS lainnya.

CoreDNS cocok digunakan pada **Kilat VM** ketika pengguna membutuhkan DNS server yang ringan, fleksibel, dan dapat dikonfigurasi secara mandiri.

### Fungsi CoreDNS

CoreDNS dapat digunakan untuk:

* Menjadi **authoritative DNS server** untuk sebuah domain.
* Mengelola dan melayani **DNS Zone** serta **DNS Record**.
* Menjawab permintaan DNS dari client.
* Mendukung berbagai jenis DNS Record seperti `A`, `AAAA`, `CNAME`, `MX`, `NS`, dan `TXT`.
* Meneruskan permintaan DNS ke DNS server lain melalui konfigurasi plugin yang sesuai.

### Kelebihan CoreDNS

Beberapa kelebihan CoreDNS antara lain:

* **Ringan**, sehingga dapat berjalan dengan penggunaan resource yang relatif rendah.
* **Fleksibel**, karena menggunakan sistem plugin yang dapat disesuaikan dengan kebutuhan.
* **Open source**, sehingga dapat digunakan dan dikembangkan secara bebas.
* **Mudah dikonfigurasi**, menggunakan file konfigurasi `Corefile`.
* **Mendukung berbagai kebutuhan DNS**, termasuk authoritative DNS dan DNS forwarding.

### Kekurangan CoreDNS

CoreDNS juga memiliki beberapa hal yang perlu diperhatikan:

* Konfigurasi DNS memerlukan pemahaman dasar mengenai DNS, seperti zone, record, dan nameserver.
* Pengelolaan DNS dilakukan secara mandiri pada server sehingga konfigurasi dan keamanannya perlu diperhatikan.
* Untuk kebutuhan **high availability**, diperlukan lebih dari satu DNS server agar tidak bergantung pada satu server.

Pada panduan ini, CoreDNS akan dikonfigurasi sebagai **authoritative DNS server** pada Kilat VM dengan sistem operasi Ubuntu 24.04 LTS.

## Persiapan Awal

Sebelum memulai instalasi dan konfigurasi CoreDNS, pastikan kamu sudah memiliki:

1. **Kilat VM** dengan sistem operasi Ubuntu 24.04 LTS.
2. Akses **Root** atau user dengan hak akses `sudo`.
3. **IP Address publik** pada Kilat VM.
4. Domain yang akan digunakan untuk konfigurasi DNS.
5. Akses untuk melakukan konfigurasi DNS pada registrar domain.
6. Port `53/UDP` dan `53/TCP` dapat diakses dari internet.

Pada panduan ini, kita akan menggunakan contoh konfigurasi berikut:

| Konfigurasi    | Nilai                |
| -------------- | -------------------- |
| Sistem Operasi | Ubuntu 24.04 LTS     |
| Domain         | `domainkamu.com`     |
| Nameserver     | `ns1.domainkamu.com` |
| Nameserver     | `ns2.domainkamu.com` |

> **Catatan:** `domainkamu.com` hanya digunakan sebagai contoh. Silakan sesuaikan dengan domain dan IP Address yang digunakan.

## Rangkuman Versi yang Digunakan

Panduan ini menggunakan konfigurasi berikut:

| Komponen       | Versi            |
| -------------- | ---------------- |
| Sistem Operasi | Ubuntu 24.04 LTS |
| CoreDNS        | 1.14.6           |
| DNS Utilities  | `dig`            |

> **Catatan:** Versi CoreDNS dapat berubah seiring adanya release terbaru. Versi yang digunakan pada panduan ini adalah CoreDNS 1.14.6.

## 1. Update Sistem

Sebelum melakukan instalasi CoreDNS, lakukan update package pada Ubuntu dengan menjalankan perintah berikut:

```bash
apt update && apt upgrade -y
```

Kemudian install package yang diperlukan:

```bash
apt install -y curl tar dnsutils
```
<p align="center">
  <img width="853" height="233" alt="Instalasi package dnsutils" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/instalasi-package-dnsutils.png" style="border-radius: 10px;" />
  <br>
  Gambar 1: Instalasi paket
</p>

Package `dnsutils` digunakan untuk melakukan pengujian DNS menggunakan perintah `dig`.

> **Catatan:** Perintah pada panduan ini diasumsikan dijalankan menggunakan user `root`. Jika menggunakan user biasa, tambahkan `sudo` pada perintah yang membutuhkan hak akses administratif.

## 2. Instalasi CoreDNS

CoreDNS menyediakan binary yang dapat digunakan pada berbagai sistem operasi dan arsitektur Linux.

Pada panduan ini, binary yang digunakan adalah CoreDNS versi **1.14.6** untuk Linux dengan arsitektur **AMD64**.

Masuk ke direktori `/tmp`:

```bash
cd /tmp
```

Download binary CoreDNS:

```bash
curl -LO https://github.com/coredns/coredns/releases/download/v1.14.6/coredns_1.14.6_linux_amd64.tgz
```
<p align="center">
  <img width="868" height="114" alt="Download CoreDNS" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/instalasi-coredns.png" style="border-radius: 10px;" />
  <br>
  Gambar 2: Download CoreDNS
</p>

Ekstrak file yang telah di-download:

```bash
tar -xzf coredns_1.14.6_linux_amd64.tgz
```

Pindahkan binary CoreDNS ke direktori `/usr/local/bin`:

```bash
mv coredns /usr/local/bin/coredns
```

Kemudian berikan permission agar binary dapat dijalankan:

```bash
chmod +x /usr/local/bin/coredns
```

Pastikan CoreDNS telah berhasil di-install dengan menjalankan:

```bash
coredns -version
```
<p align="center">
  <img width="359" height="78" alt="Verifikasi versi CoreDNS" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/verifikasi-versi-coredns.png" style="border-radius: 10px;" />
  <br>
  Gambar 3: Verifikasi versi CoreDNS
</p>

Jika instalasi berhasil, versi CoreDNS akan ditampilkan pada terminal.

## 3. Membuat User dan Direktori CoreDNS

Untuk meningkatkan keamanan, CoreDNS akan dijalankan menggunakan user khusus bernama `coredns`.

Buat user `coredns` dengan perintah berikut:

```bash
useradd --system --no-create-home --shell /usr/sbin/nologin coredns
```

Kemudian buat direktori untuk menyimpan konfigurasi dan zone file:

```bash
mkdir -p /etc/coredns/zones
```

Ubah ownership direktori tersebut agar dapat diakses oleh user `coredns`:

```bash
chown -R coredns:coredns /etc/coredns
```

## 4. Membuat Zone File

Zone file digunakan untuk menyimpan DNS record yang akan dilayani oleh CoreDNS.

Buat zone file untuk domain `domainkamu.com`:

```bash
nano /etc/coredns/zones/db.domainkamu.com
```

Kemudian tambahkan konfigurasi berikut:

```dns
$ORIGIN domainkamu.com.
$TTL 3600

@   IN  SOA ns1.domainkamu.com. admin.domainkamu.com. (
        2026091201 ; Serial
        3600       ; Refresh
        600        ; Retry
        86400      ; Expire
        3600       ; Minimum TTL
)

    IN  NS  ns1.domainkamu.com.
    IN  NS  ns2.domainkamu.com.

ns1 IN  A   IP-VPS
ns2 IN  A   IP-VPS

www IN  A   IP-VPS
@   IN  A   IP-VPS
```

Simpan konfigurasi tersebut.

> **Catatan:** Ganti `IP-VPS` dengan IP Address publik Kilat VM yang digunakan. Pastikan nilai `ns1`, `ns2`, `www`, dan `@` disesuaikan dengan kebutuhan konfigurasi DNS.

Pada konfigurasi tersebut terdapat beberapa DNS record:

| Record | Fungsi                                                       |
| ------ | ------------------------------------------------------------ |
| `SOA`  | Menentukan informasi utama dari zone                         |
| `NS`   | Menentukan nameserver yang bertanggung jawab terhadap domain |
| `A`    | Mengarahkan hostname ke IP Address                           |
| `ns1`  | Mengarahkan nameserver pertama ke IP Address VPS             |
| `ns2`  | Mengarahkan nameserver kedua ke IP Address VPS               |
| `www`  | Mengarahkan `www.domainkamu.com` ke IP Address VPS           |
| `@`    | Mengarahkan domain utama `domainkamu.com` ke IP Address VPS  |

> **Catatan:** Nilai `Serial` perlu dinaikkan setiap kali terdapat perubahan pada zone file agar perubahan dapat dikenali oleh DNS server.

## 5. Konfigurasi CoreDNS

CoreDNS menggunakan file `Corefile` sebagai konfigurasi utama. File ini digunakan untuk menentukan zone yang dilayani serta plugin yang digunakan.

Buat file `Corefile`:

```bash
nano /etc/coredns/Corefile
```

Masukkan konfigurasi berikut:

```text
domainkamu.com:53 {
    file /etc/coredns/zones/db.domainkamu.com
    log
    errors
}
```
<p align="center">
  <img width="828" height="189" alt="Konfigurasi Corefile CoreDNS" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/konfigurasi-corefile-coredns.png" style="border-radius: 10px;" />
  <br>
  Gambar 4: Konfigurasi Corefile
</p>

Konfigurasi tersebut membuat CoreDNS melayani query DNS untuk `domainkamu.com` pada port `53` menggunakan zone file `/etc/coredns/zones/db.domainkamu.com`.

Simpan konfigurasi tersebut, kemudian ubah ownership file:

```bash
chown coredns:coredns /etc/coredns/Corefile
```

## 6. Menonaktifkan DNS Stub Listener Ubuntu

Pada Ubuntu 24.04, terdapat service `systemd-resolved` yang membantu sistem melakukan koneksi ke DNS. Service ini dapat menggunakan port `53` melalui fitur **DNS Stub Listener**.

CoreDNS juga membutuhkan port `53` untuk menerima permintaan DNS. Jika port tersebut sudah digunakan oleh `systemd-resolved`, CoreDNS tidak dapat menggunakan port yang sama.

Periksa terlebih dahulu penggunaan port `53`:

```bash
ss -lntup | grep ':53'
```
<p align="center">
  <img width="940" height="97" alt="Pengecekan penggunaan port 53" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/cek-penggunaan-port53.jpeg" style="border-radius: 10px;" />
  <br>
  Gambar 5: Pemeriksaan port 53
</p>

Jika terdapat proses `systemd-resolved` yang menggunakan port `53`, ubah konfigurasi `systemd-resolved`:

```bash
nano /etc/systemd/resolved.conf
```

Pada bagian `[Resolve]`, tambahkan atau ubah konfigurasi menjadi:

```ini
[Resolve]
DNSStubListener=no
```
<p align="center">
  <img width="940" height="544" alt="Konfigurasi DNSStubListener" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/konfigurasi-dnsstublistener.png" style="border-radius: 10px;" />
  <br>
  Gambar 6: Konfigurasi DNSStubListener
</p>

Simpan konfigurasi, kemudian restart service:

```bash
systemctl restart systemd-resolved
```

Periksa kembali penggunaan port `53`:

```bash
ss -lntup | grep ':53'
```
<p align="center">
  <img width="445" height="44" alt="verifikasi-port-dns-coredns" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/verifikasi-port-dns-coredns.png" style="border-radius: 10px;" />
  <br>
  Gambar 7: Verifikasi port DNS
</p>

Pastikan port `53` tidak lagi digunakan oleh `systemd-resolved`.

> **Catatan:** Langkah ini hanya menonaktifkan DNS stub listener milik `systemd-resolved`, bukan menghentikan service `systemd-resolved` secara keseluruhan.

## 7. Memberikan Permission Port 53

Karena CoreDNS akan dijalankan menggunakan user `coredns`, berikan capability agar binary CoreDNS dapat menggunakan privileged port `53`:

```bash
setcap 'cap_net_bind_service=+ep' /usr/local/bin/coredns
```

Verifikasi capability yang telah diberikan:

```bash
getcap /usr/local/bin/coredns
```
<p align="center">
  <img width="534" height="69" alt="verifikasi-capability-coredns" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/verifikasi-capability-coredns.png" style="border-radius: 10px;" />
  <br>
  Gambar 8: Verifikasi capability CoreDNS
</p>

Output yang diharapkan:

```text
/usr/local/bin/coredns cap_net_bind_service=ep
```

## 8. Pengujian Konfigurasi CoreDNS

Sebelum membuat CoreDNS sebagai systemd service, lakukan pengujian untuk memastikan konfigurasi dapat berjalan dengan baik.

Jalankan CoreDNS menggunakan user `coredns`:

```bash
sudo -u coredns /usr/local/bin/coredns -conf /etc/coredns/Corefile
```
<p align="center">
  <img width="827" height="147" alt="pengujian-manual-coredns" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/pengujian-manual-coredns.png" style="border-radius: 10px;" />
  <br>
  Gambar 9: Pengujian manual CoreDNS
</p>

Jika CoreDNS berhasil berjalan tanpa pesan error, konfigurasi telah berhasil diterapkan.

Untuk menghentikan proses pengujian, tekan:

```text
Ctrl + C
```

> **Catatan:** Pastikan tidak terdapat pesan error seperti `bind: permission denied` atau `bind: address already in use`.

## 9. Membuat Service CoreDNS

Agar CoreDNS dapat dijalankan sebagai service dan otomatis aktif ketika server melakukan reboot, buat file service systemd:

```bash
nano /etc/systemd/system/coredns.service
```

Tambahkan konfigurasi berikut:

```ini
[Unit]
Description=CoreDNS DNS Server
Documentation=https://coredns.io/
After=network-online.target
Wants=network-online.target

[Service]
User=coredns
Group=coredns
ExecStart=/usr/local/bin/coredns -conf /etc/coredns/Corefile
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
<p align="center">
  <img width="847" height="490" alt="service-coredns" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/service-coredns.png" style="border-radius: 10px;" />
  <br>
  Gambar 10: Service CoreDNS
</p>

Simpan file tersebut.

Kemudian reload konfigurasi systemd:

```bash
systemctl daemon-reload
```

Jalankan service CoreDNS:

```bash
systemctl start coredns
```

Aktifkan service agar CoreDNS otomatis berjalan ketika server melakukan reboot:

```bash
systemctl enable coredns
```

Periksa status CoreDNS:

```bash
systemctl status coredns
```

Jika service berhasil berjalan, status akan menunjukkan:

```text
Active: active (running)
```
<p align="center">
  <img width="631" height="271" alt="status-service-coredns" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/status-service-coredns.png" style="border-radius: 10px;" />
  <br>
  Gambar 11: Status service CoreDNS
</p>

## 10. Konfigurasi Firewall

CoreDNS menggunakan port `53` untuk menerima DNS query. Pastikan port `53/UDP` dan `53/TCP` dapat diakses dari internet.

Jika menggunakan UFW, jalankan:

```bash
ufw allow 53/udp
ufw allow 53/tcp
```

Kemudian periksa status firewall:

```bash
ufw status
```
<p align="center">
  <img width="600" height="375" alt="ufw-status" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/ufw-status.png" style="border-radius: 10px;" />
  <br>
  Gambar 12: Verifikasi UFW
</p>

Pastikan port DNS telah diizinkan pada firewall.

## 11. Verifikasi Port dan DNS

Setelah CoreDNS berhasil dijalankan, pastikan CoreDNS telah listen pada port `53`:

```bash
ss -lntup | grep ':53'
```
<p align="center">
  <img width="850" height="77" alt="verifikasi-port-53-coredns" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/verifikasi-port-53-coredns.png" style="border-radius: 10px;" />
  <br>
  Gambar 13: Verifikasi port 53 CoreDNS
</p>

Pastikan terdapat proses CoreDNS yang menggunakan port tersebut.

Selanjutnya, lakukan pengujian DNS secara langsung ke IP Address Kilat VM:

```bash
dig @IP-VPS domainkamu.com +short
```

Untuk melakukan pengujian terhadap record `www`, gunakan:

```bash
dig @IP-VPS www.domainkamu.com +short
```

Jika konfigurasi berhasil, hasil query akan menampilkan IP Address VPS yang telah dikonfigurasi pada zone file.

> **Catatan:** Pengujian menggunakan `dig @IP-VPS` dilakukan secara langsung ke server CoreDNS sehingga tidak bergantung pada nameserver domain yang sedang digunakan.

## 12. Konfigurasi Nameserver Domain

Setelah CoreDNS berhasil berjalan dan dapat menerima DNS query, domain perlu diarahkan ke **Private NameServer** yang telah dikonfigurasi pada server.

Pada contoh ini, nameserver yang digunakan adalah:

```text
ns1.domainkamu.com
ns2.domainkamu.com
```

Karena domain menggunakan **Private NameServer**, lakukan pendaftaran nameserver dan perubahan nameserver melalui **Portal Client CloudKilat**.

Untuk langkah lengkap mengenai pembuatan Private NameServer dan perubahan nameserver pada domain, silakan mengikuti panduan berikut:

[**Cara Menggunakan Private Name Server pada Domain di Portal Client CloudKilat**](https://kb.cloudkilat.id/domain-di-cloudkilat/cara-menggunakan-private-name-server-pada-domain-di-portal-client-cloudkilat)

Pada panduan tersebut, Private NameServer didaftarkan dengan memasukkan hostname seperti `ns1` dan `ns2` beserta IP Address publik server. Setelah Private NameServer berhasil dibuat, nameserver domain dapat diubah menggunakan opsi **Use custom nameservers**.

> **Catatan:** Setelah nameserver domain diubah, proses propagasi DNS dapat membutuhkan waktu. Pastikan juga seluruh DNS record yang diperlukan, seperti `A`, `MX`, `CNAME`, dan `TXT`, telah dikonfigurasi pada DNS server yang digunakan.

## 13. Verifikasi DNS

Setelah konfigurasi nameserver selesai dan propagasi DNS telah berlangsung, lakukan pengecekan DNS menggunakan `dig`.

Untuk memeriksa nameserver domain:

```bash
dig NS domainkamu.com +short
```

Untuk memeriksa record A:

```bash
dig A domainkamu.com +short
```

Pastikan hasil query menunjukkan nameserver dan DNS record sesuai dengan konfigurasi yang telah dibuat.

Untuk melihat proses delegasi DNS dari root hingga nameserver domain, gunakan:

```bash
dig +trace domainkamu.com
```

## 14. Verifikasi DNS Menggunakan DNS Checker

Setelah nameserver domain dikonfigurasi, lakukan pengecekan untuk memastikan DNS record telah tersebar dan dapat diakses dari berbagai lokasi.

Gunakan layanan [DNS Checker](https://dnschecker.org/) untuk melakukan pengecekan terhadap record DNS.

Masukkan nama domain, kemudian pilih jenis record yang ingin diperiksa, seperti `NS` atau `A`.

<p align="center">
  <img width="1600" height="850" alt="dns-checker" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/dns-checker.png" style="border-radius: 10px;" />
  <br>
  Gambar 14: Verifikasi DNS Checker
</p>

Pastikan hasil pengecekan menunjukkan nameserver dan IP Address yang sesuai dengan konfigurasi pada CoreDNS. Perubahan DNS tidak selalu langsung terlihat di seluruh lokasi karena dipengaruhi oleh proses propagasi DNS dan nilai TTL.

> **Catatan:** Propagasi DNS adalah proses penyebaran perubahan DNS record ke berbagai DNS server di internet. Proses ini membutuhkan waktu karena setiap DNS server dapat menyimpan informasi DNS berdasarkan nilai TTL (*Time to Live*). Selama proses propagasi, hasil pengecekan DNS dapat berbeda-beda di setiap lokasi.

## Troubleshooting

### CoreDNS Tidak Dapat Berjalan

Periksa status service:

```bash
systemctl status coredns
```

Jika terdapat error, periksa log service CoreDNS:

```bash
journalctl -u coredns -n 50 --no-pager
```

Pastikan tidak terdapat kesalahan pada `Corefile` maupun zone file.

### Port 53 Sudah Digunakan

Periksa service yang menggunakan port `53`:

```bash
ss -lntup | grep ':53'
```

Jika port tersebut telah digunakan oleh service DNS lain, pastikan service tersebut tidak berjalan secara bersamaan dengan CoreDNS atau sesuaikan konfigurasi port sesuai kebutuhan.

### DNS Tidak Dapat Diakses dari Internet

Jika DNS tidak dapat diakses dari internet, periksa beberapa hal berikut:

1. Pastikan service CoreDNS dalam kondisi `active (running)`.
2. Pastikan CoreDNS listen pada port `53`.
3. Pastikan port `53/UDP` dan `53/TCP` telah dibuka pada firewall.
4. Pastikan IP Address pada zone file sudah sesuai.
5. Pastikan nameserver domain telah dikonfigurasi dengan benar.
6. Pastikan glue record telah dibuat apabila diperlukan.
7. Pastikan tidak terdapat firewall atau security rule lain yang memblokir koneksi pada port `53`.

### Perubahan DNS Tidak Muncul

Jika perubahan pada DNS record belum terlihat, pastikan nilai `Serial` pada zone file telah dinaikkan.

Contoh:

```dns
2026091202
```

Setelah melakukan perubahan, restart CoreDNS:

```bash
systemctl restart coredns
```

Kemudian lakukan pengecekan kembali:

```bash
dig @IP-VPS domainkamu.com
```

## Kesimpulan

CoreDNS dapat digunakan sebagai DNS server pada Kilat VM dengan sistem operasi Ubuntu 24.04 LTS. Dengan menggunakan `Corefile` dan zone file, pengguna dapat menentukan zone serta DNS record yang akan dilayani oleh CoreDNS.

Dengan mengikuti panduan ini, CoreDNS telah dikonfigurasi sebagai **authoritative DNS server** dan dapat diuji secara langsung menggunakan `dig`. Agar domain dapat menggunakan DNS server tersebut dari internet, pastikan service CoreDNS aktif, port `53/UDP` dan `53/TCP` dapat diakses, serta nameserver domain telah diarahkan ke Private NameServer yang sesuai.

## Referensi

- [CoreDNS Official Website](https://coredns.io/)
- [CoreDNS GitHub Releases](https://github.com/coredns/coredns/releases)
- [KB - Cara Menggunakan Private Name Server pada Domain di Portal Client CloudKilat](https://kb.cloudkilat.id/domain-di-cloudkilat/cara-menggunakan-private-name-server-pada-domain-di-portal-client-cloudkilat)
