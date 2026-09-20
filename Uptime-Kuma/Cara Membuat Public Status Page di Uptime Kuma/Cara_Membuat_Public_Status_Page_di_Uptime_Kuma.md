# Cara Membuat Public Status Page di Uptime Kuma

Halo, Kawan Belajar! Dashboard Uptime Kuma hanya dapat diakses setelah login sebagai admin, sehingga status layanan tidak dapat dilihat langsung oleh pengguna atau klien. Untuk kebutuhan tersebut, Uptime Kuma menyediakan fitur **Public Status Page**, yaitu halaman status yang dapat diakses tanpa login dan menampilkan monitor yang dipilih saja.

Pada panduan ini akan dibahas cara membuat status page, mengelompokkan monitor ke dalam beberapa grup, menyesuaikan tampilannya, membuat pengumuman gangguan (incident), hingga menampilkan status page pada domain sendiri menggunakan reverse proxy Nginx dan SSL Let's Encrypt.

> **Catatan:** Panduan ini merupakan lanjutan dari artikel [Cara Menambahkan dan Mengonfigurasi Monitor di Uptime Kuma](https://kb.cloudkilat.id/uptime-kuma/cara-menambahkan-dan-mengonfigurasi-monitor-di-uptime-kuma). Pastikan Uptime Kuma sudah terinstal dan minimal satu monitor sudah ditambahkan sebelum mengikuti langkah-langkah berikut.

## Persiapan Awal

Sebelum memulai, pastikan sudah memiliki:

1. Uptime Kuma yang sudah terinstal dan dapat diakses melalui dashboard, sesuai artikel instalasi sebelumnya.
2. Akun admin Uptime Kuma yang sudah login ke dashboard.
3. Minimal satu monitor yang sudah ditambahkan, karena status page hanya menampilkan monitor yang sudah ada.
4. (Opsional) File logo yang akan ditampilkan pada status page.
5. (Opsional) Domain atau subdomain beserta akses ke pengaturan DNS-nya, apabila status page ingin diakses melalui domain sendiri seperti pada bagian [Menampilkan Status Page pada Domain Sendiri](#8-opsional-menampilkan-status-page-pada-domain-sendiri).

---

## 1. Membuka Halaman Status Pages

Pada dashboard Uptime Kuma, klik menu **Status Pages** di bagian kanan atas (1), kemudian klik tombol **New Status Page** (2).

<p align="center">
<img alt="Membuka Halaman Status Pages" src="Images/1_akses_status_page.png" />
  <br>
  <em>Gambar 1: Membuka Halaman Status Pages</em>
</p>

Apabila belum pernah membuat status page, halaman akan menampilkan keterangan `No status pages`. Ini adalah kondisi normal.

---

## 2. Menentukan Nama dan Slug Status Page

Pada halaman **Add New Status Page**, isi konfigurasi berikut:

* **Name**: nama status page, contoh `CloudKilat`.
* **Slug**: bagian akhir URL status page, contoh `cloudkilat` sehingga status page dapat diakses pada path `/status/cloudkilat`.

<p align="center">
<img alt="Menentukan Nama dan Slug Status Page" src="Images/2_nama_status_page.png" />
  <br>
  <em>Gambar 2: Menentukan Nama dan Slug Status Page</em>
</p>

Ketentuan penulisan slug:

* Karakter yang diterima hanya `a-z`, `0-9`, dan tanda hubung (`-`).
* Tidak boleh menggunakan tanda hubung berurutan (`--`).
* Slug khusus `default` akan ditampilkan ketika status page diakses tanpa menyertakan slug.

Klik **Next** untuk melanjutkan ke halaman editor status page.

---

## 3. Mengenal Halaman Editor Status Page

Halaman editor terbagi menjadi dua bagian, yaitu panel pengaturan di sisi kiri dan area preview di sisi kanan yang dapat diedit secara langsung.

<p align="center">
<img alt="Halaman Editor Status Page" src="Images/3_halaman_status_page.png" />
  <br>
  <em>Gambar 3: Halaman Editor Status Page</em>
</p>

Bagian yang dapat dikonfigurasi pada area preview:

1. **Logo**: klik ikon unggah pada logo untuk mengganti gambar yang ditampilkan di bagian atas status page.
2. **Judul**: nama status page yang tampil pada halaman, dapat diedit langsung maupun melalui field **Title** pada panel kiri.
3. **Create Incident**: membuat pengumuman gangguan atau pemeliharaan yang ditampilkan sebagai banner di bagian atas status page, dibahas pada langkah 7.
4. **Description**: deskripsi singkat mengenai status page, mendukung format Markdown.
5. **Add Group**: membuat grup/kategori untuk mengelompokkan monitor, contoh `Website`, `Port`, dan `Server`.
6. **Add a monitor**: dropdown untuk memilih monitor yang akan ditampilkan pada status page.
7. **Custom Footer**: teks footer yang ditampilkan di bagian bawah status page, mendukung format Markdown.

Apabila grup maupun monitor belum ditambahkan, area preview akan menampilkan keterangan `Nothing here, please add a group or a monitor`.

---

## 4. Pengaturan pada Panel Kiri Editor

Panel kiri berisi pengaturan tampilan dan perilaku status page. Seluruh perubahan baru tersimpan setelah tombol **Save** di bagian bawah panel ditekan.

<p align="center">
<img alt="Panel Pengaturan Status Page" src="Images/4_konfigurasi_halaman.png" />
  <br>
  <em>Gambar 4: Panel Pengaturan Status Page</em>
</p>

Pengaturan yang tersedia:

* **Slug**: mengubah slug yang sudah dibuat pada langkah sebelumnya.
* **Title**: judul status page.
* **Description**: deskripsi status page, mendukung Markdown.
* **Footer Text**: teks footer status page, mendukung Markdown.
* **Refresh Interval**: jeda refresh otomatis halaman status dalam detik, default `300`.
* **Theme**: tema tampilan status page, yaitu `Auto`, `Light`, atau `Dark`.
* **Show Tags**: menampilkan tag monitor pada status page.
* **Show Powered By**: menampilkan keterangan `Powered by Uptime Kuma` pada bagian bawah halaman.
* **Show Certificate Expiry**: menampilkan sisa masa berlaku sertifikat SSL pada monitor HTTPS, tampil sebagai badge `Cert Exp.` di samping nama monitor.
* **Show Only Last Heartbeat**: menampilkan hanya hasil pengecekan terakhir, tanpa grafik heartbeat.
* **Domain Names**: mendaftarkan domain yang digunakan untuk mengakses status page, dibahas pada langkah 8.
* **Analytics Type**: integrasi layanan analytics, default `None`.
* **RSS Title**: judul feed RSS status page, dikosongkan untuk mengikuti judul status page.
* **Custom CSS**: CSS tambahan untuk menyesuaikan tampilan status page.

---

## 5. Menambahkan Grup dan Monitor

Monitor pada status page harus berada di dalam sebuah grup, sehingga grup perlu dibuat terlebih dahulu.

1. Klik **Add Group**, lalu ubah nama grup sesuai kebutuhan, contoh `Website`.
2. Pilih monitor yang ingin ditampilkan melalui dropdown **Add a monitor**.
3. Ulangi langkah 1 dan 2 untuk grup lain, contoh grup `Port` untuk monitor TCP Port dan grup `Server` untuk monitor Ping.
4. Klik **Save** pada panel kiri untuk menyimpan status page.

---

## 6. Mengakses Status Page

Setelah disimpan, status page dapat diakses melalui URL berikut tanpa perlu login:

```
http://IP_VPS:3001/status/SLUG
```

Ganti `IP_VPS` dengan IP Address publik VPS Uptime Kuma dan `SLUG` dengan slug yang dibuat pada langkah 2.

<p align="center">
<img alt="Tampilan Public Status Page" src="Images/5_halaman_jadi.png" />
  <br>
  <em>Gambar 5: Tampilan Public Status Page</em>
</p>

Informasi yang ditampilkan pada status page:

* **Banner status keseluruhan**, contoh `All Systems Operational` ketika seluruh monitor berstatus up.
* **Persentase uptime** pada masing-masing monitor.
* **Grafik heartbeat**, berupa bar hijau (up) atau merah (down) pada setiap siklus pengecekan.
* **Last Updated** dan **Refresh in**, sebagai penanda waktu pembaruan data terakhir dan hitung mundur refresh berikutnya sesuai nilai Refresh Interval.

> **Catatan:** Status page bersifat publik dan dapat diakses siapa pun yang mengetahui URL-nya, sehingga sebaiknya gunakan Friendly Name monitor yang tidak memuat informasi sensitif seperti nama host internal.

---

## 7. Membuat Incident pada Status Page

Fitur **Incident** digunakan untuk menyampaikan pengumuman gangguan atau jadwal pemeliharaan kepada pengunjung status page.

Pada halaman editor status page, klik **Create Incident**, lalu isi:

* **Title**: judul pengumuman, contoh `Maintenance`.
* **Content**: isi pengumuman, mendukung format Markdown.
* **Style**: warna banner pengumuman, tersedia pilihan `info`, `warning`, `danger`, `primary`, `light`, dan `dark`.

<p align="center">
<img alt="Membuat Incident pada Status Page" src="Images/6_create_incident.png" />
  <br>
  <em>Gambar 6: Membuat Incident pada Status Page</em>
</p>

Klik **Post** untuk menayangkan pengumuman, atau **Cancel** untuk membatalkannya. Setelah ditayangkan, banner pengumuman akan tampil di bagian paling atas status page beserta keterangan **Date Created**.

<p align="center">
<img alt="Tampilan Incident pada Status Page" src="Images/7_result_incident.png" />
  <br>
  <em>Gambar 7: Tampilan Incident pada Status Page</em>
</p>

---

## 8. (Opsional) Menampilkan Status Page pada Domain Sendiri

Secara default status page diakses melalui `IP_VPS:3001`. Agar dapat diakses melalui domain sendiri beserta HTTPS, diperlukan reverse proxy pada VPS Uptime Kuma, kemudian domain tersebut didaftarkan pada pengaturan **Domain Names** status page.

Prasyarat:

* Domain atau subdomain dengan A record yang sudah diarahkan (pointing) ke IP Address publik VPS Uptime Kuma.
* Port `80/tcp` dan `443/tcp` dapat diakses dari internet.

### 8.1 Install Nginx

```bash
apt update
apt install nginx -y
```

### 8.2 Membuat Konfigurasi Reverse Proxy

Buat file konfigurasi virtual host:

```bash
nano /etc/nginx/conf.d/uptime-kuma.conf
```

Isi dengan konfigurasi berikut, ganti `uptime-kuma.domainkamu.com` dengan domain yang digunakan:

```nginx
server {
    listen 80;
    server_name uptime-kuma.domainkamu.com;

    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Dukungan WebSocket
        proxy_set_header Sec-WebSocket-Key $http_sec_websocket_key;
        proxy_set_header Sec-WebSocket-Version $http_sec_websocket_version;
        proxy_set_header Sec-WebSocket-Extensions $http_sec_websocket_extensions;

        # Meningkatkan performa reverse proxy
        proxy_buffering off;
    }
}
```

Uji konfigurasi dan reload Nginx:

```bash
nginx -t
systemctl reload nginx
```

> **Catatan:** Header `Upgrade` dan `Connection` wajib disertakan karena Uptime Kuma menggunakan WebSocket. Tanpa kedua header tersebut, halaman tetap terbuka namun status monitor tidak diperbarui secara realtime.

### 8.3 Memasang SSL Let's Encrypt

Install Certbot beserta plugin Nginx:

```bash
apt install certbot python3-certbot-nginx -y
```

Terbitkan sertifikat SSL untuk domain yang digunakan:

```bash
certbot --nginx -d uptime-kuma.domainkamu.com
```

Certbot akan menambahkan konfigurasi HTTPS pada file virtual host secara otomatis, termasuk pengalihan dari HTTP ke HTTPS apabila opsi tersebut dipilih.

### 8.4 Mendaftarkan Domain pada Status Page

Kembali ke halaman editor status page, lalu pada panel kiri bagian **Domain Names**, klik ikon tambah dan masukkan domain yang sudah dikonfigurasi, contoh `uptime-kuma.domainkamu.com`.

<p align="center">
<img alt="Menambahkan Domain pada Status Page" src="Images/8_domain.png" />
  <br>
  <em>Gambar 8: Menambahkan Domain pada Status Page</em>
</p>

Klik **Save** untuk menyimpan perubahan.

### 8.5 Uji Coba Akses melalui Domain

Akses domain tersebut melalui browser. Status page akan langsung tampil pada domain yang didaftarkan, tanpa perlu menyertakan path `/status/SLUG`.

<p align="center">
<img alt="Status Page Diakses melalui Domain Sendiri" src="Images/9_domain_hasil.png" />
  <br>
  <em>Gambar 9: Status Page Diakses melalui Domain Sendiri</em>
</p>

---

## Troubleshooting

### Status Page Kosong Meski Monitor Sudah Ada

Pastikan monitor sudah ditambahkan ke dalam grup melalui dropdown **Add a monitor**, kemudian klik **Save**. Perubahan pada editor tidak tersimpan sebelum tombol Save ditekan.

### Monitor Tidak Muncul pada Dropdown Add a monitor

Dropdown hanya menampilkan monitor yang sudah dibuat pada Uptime Kuma. Tambahkan monitor terlebih dahulu sesuai artikel [Cara Menambahkan dan Mengonfigurasi Monitor di Uptime Kuma](https://kb.cloudkilat.id/uptime-kuma/cara-menambahkan-dan-mengonfigurasi-monitor-di-uptime-kuma).

### Slug Ditolak saat Dibuat

Pastikan slug hanya menggunakan karakter `a-z`, `0-9`, dan tanda hubung tunggal, serta belum digunakan oleh status page lain.

### Badge Cert Exp. Tidak Muncul pada Monitor

Aktifkan opsi **Show Certificate Expiry** pada panel kiri editor. Badge ini hanya tampil pada monitor HTTP(s) yang mengakses URL dengan HTTPS.

### Domain Menampilkan Halaman Login atau Error 404

Periksa hal berikut:

1. Pastikan domain sudah ditambahkan pada bagian **Domain Names** di editor status page, kemudian disimpan dengan tombol **Save**.
2. Pastikan A record domain sudah mengarah ke IP Address publik VPS Uptime Kuma dan propagasi DNS sudah selesai.
3. Pastikan `server_name` pada konfigurasi Nginx sama persis dengan domain yang diakses.

### Halaman Terbuka, Namun Status Tidak Diperbarui Secara Realtime

Kondisi ini umumnya terjadi apabila header WebSocket belum diteruskan oleh reverse proxy. Pastikan seluruh baris `proxy_set_header` pada bagian 8.2 sudah ditambahkan, lalu reload Nginx.

## Kesimpulan

Dengan Public Status Page, status layanan pada Uptime Kuma dapat ditampilkan kepada pengguna maupun klien tanpa memberikan akses ke dashboard admin. Monitor dapat dikelompokkan ke dalam beberapa grup sesuai jenis layanan, dilengkapi logo, deskripsi, footer khusus, serta pengumuman incident saat terjadi gangguan atau pemeliharaan. Melalui reverse proxy Nginx, SSL Let's Encrypt, dan pengaturan Domain Names, status page juga dapat diakses langsung pada domain sendiri sehingga terlihat lebih profesional.

CloudKilat menyediakan layanan Kilat VM beserta domain yang dapat digunakan untuk menjalankan Uptime Kuma sekaligus menampilkan Public Status Page pada domain sendiri. Layanan tersebut juga didukung oleh tim support CloudKilat dengan pelayanan selama 7x24 jam apabila mengalami kendala pada konfigurasi.

Terima kasih, sekian dan semoga bermanfaat.

## Referensi

- [Uptime Kuma Official Repository](https://github.com/louislam/uptime-kuma/)
- [Uptime Kuma - Reverse Proxy](https://github.com/louislam/uptime-kuma/wiki/Reverse-Proxy)