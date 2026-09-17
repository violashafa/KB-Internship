# Instalasi dan Konfigurasi SigNoz pada Ubuntu 24.04

Halo, Kawan Belajar! **SigNoz** merupakan salah satu observability platform open source yang cukup populer, karena mampu menyatukan metrics, traces, dan logs dalam satu tampilan tanpa perlu berlangganan tools terpisah.

Pada panduan ini, kita akan membahas cara instalasi dan konfigurasi SigNoz di atas Ubuntu 24.04 LTS pada Kilat VM, sehingga dapat langsung digunakan untuk memantau performa aplikasi maupun server.

## Apa Itu SigNoz?

SigNoz merupakan platform **observability open source** yang digunakan untuk memantau performa aplikasi dan infrastruktur. SigNoz menyatukan tiga jenis data observability, yaitu **metrics**, **traces**, dan **logs**, dalam satu tampilan. Dibanding menyusun sendiri kombinasi **Prometheus (metrics)**, **Loki (logs)**, dan **Tempo/Jaeger (traces)** yang masing-masing perlu di-setup dan dihubungkan ke **Grafana**, SigNoz sudah menggabungkan ketiganya dalam satu aplikasi dengan satu **database (ClickHouse)**, sehingga proses instalasi dan pengelolaannya lebih ringkas.

SigNoz dibangun di atas standar **OpenTelemetry**, sehingga pengiriman data dari aplikasi dapat dilakukan menggunakan library OpenTelemetry tanpa terikat pada satu vendor tertentu. Data yang dikirimkan disimpan pada database **ClickHouse** yang dirancang untuk menangani data dalam jumlah besar.

SigNoz cocok digunakan pada **Kilat VM** ketika pengguna membutuhkan tools monitoring yang dikelola secara mandiri tanpa biaya lisensi per host.

### Kelebihan SigNoz

Beberapa kelebihan SigNoz antara lain:

* **Open source**, sehingga dapat digunakan dan dikembangkan secara bebas.
* **Berbasis OpenTelemetry**, sehingga mengikuti standar instrumentasi yang umum digunakan.
* **All-in-one**, karena metrics, traces, dan logs dapat diakses dalam satu aplikasi.
* **Data tersimpan pada server sendiri**, sehingga kontrol data berada pada pengguna.
* **Tidak ada biaya per host**, karena dijalankan secara self-hosted.

### Kekurangan SigNoz

SigNoz juga memiliki beberapa hal yang perlu diperhatikan:

* Membutuhkan resource yang cukup besar, terutama karena penggunaan ClickHouse.
* Memerlukan pemahaman dasar mengenai OpenTelemetry untuk melakukan instrumentasi aplikasi.
* Pengelolaan dilakukan secara mandiri, termasuk update, backup, dan keamanan.
* Retention data perlu diatur agar penggunaan disk tetap terkendali.

## Kompatibilitas Sistem Operasi

SigNoz dapat dijalankan pada berbagai sistem operasi, tergantung metode instalasi yang dipilih. Seluruh metode instalasi dikelola menggunakan **Foundry** (`foundryctl`), CLI resmi SigNoz yang di-install melalui script `curl | bash` sehingga tidak bergantung pada package manager distribusi tertentu.

| Metode Instalasi       | Sistem Operasi yang Didukung                                              | Catatan                                                                 |
| ----------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| Docker Compose           | Linux, macOS, dan Windows (melalui WSL 2)                                   | Digunakan pada panduan ini. Cara paling cepat untuk mencoba SigNoz.       |
| Binary / Native (systemd) | Linux dengan systemd (Ubuntu, Debian, CentOS, Rocky Linux, AlmaLinux, dsb.) | SigNoz berjalan langsung sebagai service systemd tanpa container.         |
| Kubernetes (Helm)        | Kubernetes versi 1.22 ke atas, arsitektur x86-64/amd64 atau arm64            | Cocok untuk deployment pada cluster produksi atau multi-node.            |

> **Catatan:** Perbedaan antar distribusi Linux (Ubuntu/Debian vs CentOS/Rocky Linux/AlmaLinux) hanya berpengaruh pada perintah instalasi **Docker Engine** itu sendiri, karena Ubuntu/Debian menggunakan `apt` sedangkan CentOS/Rocky Linux/AlmaLinux menggunakan `dnf` atau `yum`. Setelah Docker Engine ter-install, seluruh langkah instalasi SigNoz menggunakan `foundryctl` tetap sama di semua distribusi.

## Memilih Metode Instalasi

Berdasarkan dokumentasi resmi, SigNoz dapat diinstal menggunakan salah satu dari tiga metode berikut:

* **Docker Compose**, menggunakan Docker Engine dan Docker Compose plugin. Seluruh komponen SigNoz (aplikasi, ClickHouse, PostgreSQL, OTel Collector) berjalan sebagai container pada satu mesin. Cocok untuk mencoba SigNoz dengan cepat atau digunakan pada satu VPS/VM tunggal.
* **Binary / Native (systemd)**, menjalankan SigNoz beserta ClickHouse dan PostgreSQL langsung sebagai service systemd tanpa Docker. Cocok apabila VPS tidak menjalankan Docker atau tim ingin mengelola setiap service secara langsung pada sistem operasi.
* **Kubernetes (Helm)**, menggunakan Helm chart resmi SigNoz untuk deployment pada cluster Kubernetes. Cocok untuk lingkungan produksi berskala besar atau yang sudah menjalankan workload pada Kubernetes.

Pada panduan ini, SigNoz akan di-install menggunakan **metode Docker Compose** pada Kilat VM dengan sistem operasi Ubuntu 24.04 LTS, karena metode ini paling ringkas dan cukup untuk kebutuhan monitoring pada satu server.

## Persiapan Awal

Sebelum memulai instalasi dan konfigurasi SigNoz, pastikan kamu sudah memiliki:

1. **Kilat VM** dengan sistem operasi Ubuntu 24.04 LTS.
2. Akses **Root** atau user dengan hak akses `sudo`.
3. **IP Address publik** pada Kilat VM.
4. Spesifikasi minimal **4 GB RAM**.
5. Ruang disk yang mencukupi untuk penyimpanan data monitoring.
6. Port `8080`, `4317`, dan `4318` dapat diakses sesuai kebutuhan.
7. Domain, apabila SigNoz akan diakses menggunakan reverse proxy dan SSL (opsional).

Pada panduan ini, kita akan menggunakan contoh konfigurasi berikut:

| Konfigurasi        | Nilai                  |
| ------------------ | ---------------------- |
| Sistem Operasi     | Ubuntu 24.04 LTS       |
| IP Address         | `IP-VPS`               |
| Domain             | `signoz.domainkamu.com`|
| Port UI SigNoz     | `8080`                 |
| Port OTLP gRPC     | `4317`                 |
| Port OTLP HTTP     | `4318`                 |

> **Catatan:** `domainkamu.com` dan `IP-VPS` hanya digunakan sebagai contoh. Silakan sesuaikan dengan domain dan IP Address yang digunakan.

## Rangkuman Versi yang Digunakan

Panduan ini menggunakan konfigurasi berikut:

| Komponen       | Versi            |
| -------------- | ---------------- |
| Sistem Operasi | Ubuntu 24.04 LTS |
| Docker Engine  | 29.8.1           |
| Docker Compose | v5.5.1      |
| SigNoz         | v0.142.1         |
| ClickHouse     | Mengikuti bawaan SigNoz v0.142.1 |

> **Catatan:** Versi di atas adalah versi terbaru pada saat panduan ini ditulis (September 2026). Karena baik Docker maupun SigNoz cukup aktif merilis versi baru, jalankan `docker --version` dan cek versi SigNoz yang ter-install setelah instalasi untuk memastikan versi yang benar-benar terpasang di server kamu.

## 1. Update Sistem

Sebelum melakukan instalasi, lakukan update package pada Ubuntu dengan menjalankan perintah berikut:

```
apt update && apt upgrade -y
```

Kemudian install package pendukung yang diperlukan:

```
apt install -y curl ca-certificates gnupg
```

<p align="center">
  <img width="850" alt="Update sistem Ubuntu" src="/images/signoz-01-update-sistem.png" style="border-radius: 10px;" />
  <br>
  Gambar 1: Update sistem
</p>

> **Catatan:** Perintah pada panduan ini diasumsikan dijalankan menggunakan user `root`. Jika menggunakan user biasa, tambahkan `sudo` pada perintah yang membutuhkan hak akses administratif.

## 2. Instalasi Docker

SigNoz dijalankan menggunakan beberapa container, sehingga Docker Engine beserta plugin Docker Compose perlu di-install terlebih dahulu.

> **Catatan:** Perintah pada bagian ini menggunakan `apt` karena mengikuti [dokumentasi resmi Docker untuk Ubuntu](https://docs.docker.com/engine/install/ubuntu/). Apabila digunakan pada CentOS, Rocky Linux, atau AlmaLinux, ganti dengan perintah `dnf`/`yum` sesuai [dokumentasi resmi Docker untuk CentOS](https://docs.docker.com/engine/install/centos/) atau distribusi terkait. Setelah Docker Engine ter-install, seluruh langkah instalasi SigNoz pada bagian selanjutnya tetap sama di semua distribusi.

Tambahkan GPG key resmi Docker:

```
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
```

Tambahkan repository Docker:

```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Kemudian install Docker Engine dan plugin Docker Compose:

```
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Pastikan Docker telah berhasil di-install:

```
docker --version
docker compose version
```

<p align="center">
  <img width="850" alt="Verifikasi versi Docker" src="/images/signoz-02-verifikasi-docker.png" style="border-radius: 10px;" />
  <br>
  Gambar 2: Verifikasi instalasi Docker
</p>

Pastikan service Docker berjalan dan aktif ketika server melakukan reboot:

```
systemctl enable --now docker
systemctl status docker
```

## 3. Instalasi SigNoz

SigNoz versi terbaru di-install menggunakan tools bernama **foundryctl**, yaitu CLI resmi SigNoz untuk mengatur proses deployment.

Install `foundryctl`:

```
curl -fsSL https://signoz.io/foundry.sh | bash
```

Mengaktifkan dan Mendaftarkan PATH `foundryctl`

```
echo 'export PATH="/root/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Buat file konfigurasi `casting.yaml` yang menentukan target deployment (Docker Compose pada satu mesin):

```
nano casting.yaml
```

Isi dengan konfigurasi berikut:

```
apiVersion: v1alpha1
kind: Installation
metadata:
  name: signoz
spec:
  deployment:
    flavor: compose
    mode: docker
```

Simpan file tersebut, kemudian jalankan proses deploy:

```
foundryctl cast -f casting.yaml
```

Perintah `cast` akan memvalidasi Docker pada server, membuat file Docker Compose pada direktori `pours/deployment/`, kemudian menjalankan seluruh container yang dibutuhkan SigNoz (aplikasi SigNoz, ClickHouse, dan OTel Collector).

> **Catatan:** Metode instalasi SigNoz dapat berubah seiring adanya release terbaru. Pastikan mengikuti dokumentasi resmi SigNoz pada saat instalasi dilakukan, terutama karena versi CLI `foundryctl` maupun struktur `casting.yaml` bisa saja diperbarui setelah panduan ini ditulis.

Proses instalasi akan men-download beberapa image container, sehingga membutuhkan waktu tergantung koneksi internet dan spesifikasi server.

<p align="center">
  <img width="850" alt="Proses instalasi SigNoz" src="/images/signoz-03-proses-instalasi.png" style="border-radius: 10px;" />
  <br>
  Gambar 3: Proses instalasi SigNoz
</p>

## 4. Verifikasi Container SigNoz

Setelah proses instalasi selesai, pastikan seluruh container SigNoz telah berjalan:

```
docker ps
```

<p align="center">
  <img width="900" alt="Verifikasi container SigNoz" src="/images/signoz-04-verifikasi-container.png" style="border-radius: 10px;" />
  <br>
  Gambar 4: Verifikasi container SigNoz
</p>

Pastikan container berada pada status `Up` dan `healthy`.

Beberapa container utama yang dijalankan oleh SigNoz:

| Container                                | Fungsi                                                      |
| ---------------------------------------- | ----------------------------------------------------------- |
| `signoz-signoz-0`                        | Menjalankan aplikasi dan tampilan antarmuka SigNoz           |
| `signoz-ingester-1`                      | Menerima data observability melalui protokol OTLP            |
| `signoz-telemetrystore-clickhouse-0-0`   | Menyimpan data metrics, traces, dan logs                     |
| `signoz-telemetrykeeper-clickhousekeeper-0` | Mengelola koordinasi cluster ClickHouse                   |
| `signoz-metastore-postgres-0`            | Menyimpan dashboard, alert rule, saved view, dan data pengguna |

> **Catatan:** Nama dan jumlah container dapat berbeda tergantung versi SigNoz yang digunakan.

Untuk memeriksa log salah satu container, gunakan perintah berikut:

```
docker logs -f NAMA-CONTAINER
```

## 5. Konfigurasi Firewall

SigNoz menggunakan beberapa port yang perlu diizinkan pada firewall.

| Port   | Protokol | Fungsi                              |
| ------ | -------- | ----------------------------------- |
| `8080` | TCP      | Akses tampilan antarmuka SigNoz     |
| `4317` | TCP      | Pengiriman data OTLP gRPC           |
| `4318` | TCP      | Pengiriman data OTLP HTTP           |

Jika menggunakan UFW, jalankan:

```
ufw allow 8080/tcp
ufw allow 4317/tcp
ufw allow 4318/tcp
```

Kemudian periksa status firewall:

```
ufw status
```

<p align="center">
  <img width="600" alt="Verifikasi UFW" src="/images/signoz-05-verifikasi-ufw.png" style="border-radius: 10px;" />
  <br>
  Gambar 5: Verifikasi UFW
</p>

> **Catatan:** Port `4317` dan `4318` hanya perlu dibuka ke internet apabila data observability dikirim dari server lain. Apabila data hanya dikirim dari server yang sama, port tersebut sebaiknya dibatasi aksesnya.

## 6. Mengakses Tampilan SigNoz

Setelah seluruh container berjalan, SigNoz dapat diakses melalui browser menggunakan alamat berikut:

```
http://IP-VPS:8080
```

Pada akses pertama, SigNoz akan meminta pembuatan akun administrator. Masukkan nama, email, dan password yang akan digunakan.

<p align="center">
  <img width="850" alt="Halaman pembuatan akun SigNoz" src="/images/signoz-06-pembuatan-akun.png" style="border-radius: 10px;" />
  <br>
  Gambar 6: Pembuatan akun administrator
</p>

Setelah akun berhasil dibuat, tampilan utama SigNoz akan ditampilkan.

<p align="center">
  <img width="900" alt="Tampilan utama SigNoz" src="/images/signoz-07-tampilan-utama.png" style="border-radius: 10px;" />
  <br>
  Gambar 7: Tampilan utama SigNoz
</p>

> **Catatan:** Gunakan password yang kuat karena tampilan SigNoz dapat diakses melalui internet.

## 7. Mengirim Data ke SigNoz

Setelah SigNoz berhasil dijalankan, tampilan SigNoz masih belum menampilkan data karena belum terdapat data yang dikirimkan.

Seluruh data observability masuk ke SigNoz melalui **OTLP (OpenTelemetry Protocol)**, yaitu protokol standar OpenTelemetry. Endpoint OTLP sudah aktif secara otomatis sebagai bagian dari container collector bawaan SigNoz (`signoz-ingester-1`) dan mendengarkan pada dua port berikut:

| Endpoint             | Protokol    | Digunakan Untuk                                            |
| -------------------- | ----------- | ---------------------------------------------------------- |
| `IP-VPS:4317`        | OTLP gRPC   | Pengiriman data dalam jumlah besar, performa lebih efisien   |
| `http://IP-VPS:4318` | OTLP HTTP   | Pengiriman data dari environment yang terbatas pada HTTP     |

Pada OTLP HTTP, setiap jenis data memiliki path masing-masing:

```
http://IP-VPS:4318/v1/traces
http://IP-VPS:4318/v1/metrics
http://IP-VPS:4318/v1/logs
```

Data dapat dikirimkan ke endpoint tersebut melalui dua cara:

* **OpenTelemetry Collector sebagai agent**, untuk mengambil metrik server (CPU, memory, disk, network) maupun membaca file log pada server. Cara ini tidak memerlukan perubahan pada kode aplikasi.
* **Instrumentasi aplikasi menggunakan library OpenTelemetry**, untuk mengirimkan traces dan metrik aplikasi. Library instrumentasi tersedia untuk berbagai bahasa seperti Python, Node.js, Java, Go, PHP, dan .NET.

Untuk menguji apakah endpoint sudah dapat menerima data, kirimkan satu buah log menggunakan `curl`:

```
curl -i -X POST http://IP-VPS:4318/v1/logs \
  -H "Content-Type: application/json" \
  -d '{
    "resourceLogs": [{
      "resource": {
        "attributes": [
          { "key": "service.name", "value": { "stringValue": "uji-coba-otlp" } }
        ]
      },
      "scopeLogs": [{
        "logRecords": [{
          "timeUnixNano": "'"$(date +%s)000000000"'",
          "severityText": "INFO",
          "body": { "stringValue": "Halo dari uji coba OTLP" }
        }]
      }]
    }]
  }'
```

Apabila endpoint berjalan normal, akan muncul respons berikut:

```
HTTP/1.1 200 OK
Content-Type: application/json
{"partialSuccess":{}}
```

Buka tampilan SigNoz, kemudian masuk ke menu **Logs**. Log dengan isi `Halo dari uji coba OTLP` akan muncul pada daftar log dengan service name `uji-coba-otlp`.

<p align="center">
  <img width="900" alt="Tampilan SigNoz setelah menerima data" src="/images/signoz-08-data-masuk.png" style="border-radius: 10px;" />
  <br>
  Gambar 8: Contoh tampilan SigNoz setelah menerima data
</p>

> **Catatan:** Endpoint OTLP bawaan SigNoz tidak dilengkapi autentikasi. Apabila port `4317` dan `4318` terbuka ke internet, siapa pun yang mengetahui IP Address server dapat mengirimkan data ke SigNoz kamu. Batasi akses kedua port tersebut hanya untuk IP Address server pengirim data, misalnya menggunakan UFW:
>
> ```
> ufw allow from IP-SERVER-PENGIRIM to any port 4317 proto tcp
> ufw allow from IP-SERVER-PENGIRIM to any port 4318 proto tcp
> ```

## 8. Konfigurasi Reverse Proxy dan SSL (Opsional)

Secara bawaan, SigNoz diakses menggunakan IP Address dan port `8080` tanpa enkripsi. Untuk penggunaan pada lingkungan produksi, disarankan menggunakan reverse proxy beserta SSL agar akses dilakukan melalui domain dan protokol HTTPS.

Install Nginx dan Certbot:

```
apt install -y nginx certbot python3-certbot-nginx
```

Buat file konfigurasi Nginx:

```
nano /etc/nginx/sites-available/signoz
```

Tambahkan konfigurasi berikut:

```
server {
    listen 80;
    server_name signoz.domainkamu.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

> **Catatan:** SigNoz belum secara resmi mendukung reverse proxy dengan path prefix (misalnya `domainkamu.com/signoz`), sehingga sebaiknya gunakan subdomain terpisah seperti `signoz.domainkamu.com` seperti pada contoh di atas. Setelah konfigurasi diterapkan, uji fitur **Live Tail** pada menu Logs untuk memastikan update log tetap muncul secara realtime melalui domain, bukan hanya saat diakses langsung lewat `IP-VPS:8080`.

Aktifkan konfigurasi tersebut:

```
ln -s /etc/nginx/sites-available/signoz /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

Kemudian install sertifikat SSL:

```
certbot --nginx -d signoz.domainkamu.com
```

Setelah sertifikat berhasil dipasang, SigNoz dapat diakses melalui:

```
https://signoz.domainkamu.com
```

> **Catatan:** Pastikan record `A` pada domain telah diarahkan ke IP Address Kilat VM sebelum menjalankan Certbot. Setelah reverse proxy digunakan, port `8080` dapat dibatasi agar tidak diakses langsung dari internet.

## 9. Konfigurasi Retention Data (Opsional)

Data monitoring akan terus bertambah seiring waktu, sehingga retention data perlu diatur agar penggunaan disk tetap terkendali.

Secara bawaan, SigNoz menyimpan data logs dan traces selama **7 hari**, serta data metrics selama **30 hari**.

Pengaturan retention dapat diubah melalui tampilan SigNoz pada tab **General** di menu **Settings**. Pada menu tersebut, durasi penyimpanan dapat diatur secara terpisah untuk metrics, traces, dan logs.

<p align="center">
  <img width="900" alt="Konfigurasi retention SigNoz" src="/images/signoz-9-retention-data.png" style="border-radius: 10px;" />
  <br>
  Gambar 10: Konfigurasi retention data
</p>

Untuk memeriksa penggunaan disk pada server, gunakan perintah berikut:

```
df -h
docker system df
```

> **Catatan:** Semakin lama durasi retention, semakin besar kebutuhan disk yang diperlukan. Sesuaikan durasi retention dengan kapasitas disk Kilat VM yang digunakan.

## Pengelolaan Service SigNoz

Beberapa perintah yang sering digunakan untuk mengelola SigNoz:

Masuk terlebih dahulu ke direktori hasil `forge`, yaitu tempat file Docker Compose SigNoz dibuat:

```
cd ~/pours/deployment
```

Menghentikan SigNoz:

```
docker compose -f compose.yaml down
```

Menjalankan kembali SigNoz:

```
docker compose -f compose.yaml up -d
```

Melihat status seluruh service:

```
docker compose -f compose.yaml ps
```

Melihat log container:

```
docker compose -f compose.yaml logs -f
```

> **Catatan:** Sesuaikan lokasi direktori `pours/` dengan tempat file `casting.yaml` dijalankan. Hindari mengubah isi file pada direktori tersebut secara manual, karena akan ditimpa ketika `foundryctl forge` atau `foundryctl cast` dijalankan kembali.

## Troubleshooting

### Container SigNoz Tidak Dapat Berjalan

Periksa status dan log container:

```
docker ps -a
docker logs NAMA-CONTAINER
```

Pastikan resource server mencukupi, terutama memory, karena SigNoz membutuhkan minimal 4 GB RAM.

### ClickHouse Gagal Berjalan

Container ClickHouse dapat gagal berjalan pada server dengan CPU yang tidak mendukung instruksi AVX2.

Periksa dukungan AVX2 pada CPU:

```
grep -o 'avx2' /proc/cpuinfo | head -1
```

Jika tidak terdapat output, CPU tidak mendukung AVX2 dan konfigurasi ClickHouse perlu disesuaikan.

### Tampilan SigNoz Tidak Dapat Diakses

Jika tampilan SigNoz tidak dapat diakses, periksa beberapa hal berikut:

1. Pastikan seluruh container berada pada status `Up`.
2. Pastikan port `8080` telah diizinkan pada firewall.
3. Pastikan tidak terdapat service lain yang menggunakan port `8080`.
4. Pastikan security rule pada jaringan tidak memblokir port tersebut.

Periksa penggunaan port menggunakan perintah berikut:

```
ss -lntup | grep ':8080'
```

### Data Tidak Muncul pada SigNoz

Jika data tidak muncul, periksa beberapa hal berikut:

1. Pastikan endpoint OTLP yang digunakan sudah sesuai.
2. Pastikan port `4317` atau `4318` dapat diakses dari sisi pengirim data.
3. Periksa log container OTel Collector untuk melihat apakah data diterima.
4. Pastikan konfigurasi pada sisi aplikasi atau agent sudah benar.

### Penggunaan Disk Terus Bertambah

Periksa penggunaan disk:

```
df -h
```

Jika penggunaan disk terus bertambah, sesuaikan konfigurasi retention data pada menu **Settings**.

## Kesimpulan

SigNoz dapat digunakan sebagai platform observability pada Kilat VM dengan sistem operasi Ubuntu 24.04 LTS. Dengan menjalankan SigNoz menggunakan Docker Compose, pengguna dapat memantau metrics, traces, dan logs dari aplikasi maupun server dalam satu tampilan.

Dengan mengikuti panduan ini, SigNoz telah berhasil di-install dan dapat menerima data melalui protokol OTLP. Agar penggunaan lebih optimal, pastikan akses SigNoz diamankan menggunakan reverse proxy dan SSL, serta retention data disesuaikan dengan kapasitas disk yang tersedia.

## Referensi

- [SigNoz Official Website](https://signoz.io/)
- [SigNoz Documentation](https://signoz.io/docs/)
- [SigNoz GitHub](https://github.com/SigNoz/signoz)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
