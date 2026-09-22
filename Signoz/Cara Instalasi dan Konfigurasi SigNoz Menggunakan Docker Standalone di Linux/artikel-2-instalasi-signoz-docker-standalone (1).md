# Cara Instalasi dan Konfigurasi SigNoz Menggunakan Docker Standalone di Ubuntu 24.04

Halo, Kawan Belajar!

Ingin memantau metrics, traces, dan logs dari aplikasi maupun server secara mandiri tanpa bergantung pada layanan pihak ketiga? Salah satu tools yang dapat digunakan untuk kebutuhan tersebut adalah **SigNoz**.

Pada panduan ini, kita akan membahas cara menginstal dan mengonfigurasi **SigNoz** menggunakan metode **Docker Standalone** pada Kilat VM dengan sistem operasi **Ubuntu 24.04 LTS**, hingga dapat menerima data observability dari server lain.

> **Catatan:** Untuk memahami SigNoz lebih lanjut, mulai dari pengertian, komponen, hingga cara kerjanya, silakan baca [Apa Itu SigNoz? Mengenal Platform Observability Berbasis OpenTelemetry](#).

## Persiapan Awal

Sebelum memulai instalasi dan konfigurasi SigNoz, pastikan kamu sudah memiliki:

1. **Dua unit Kilat VM** dengan sistem operasi Ubuntu 24.04 LTS:
   * **VPS Utama**, digunakan untuk menjalankan SigNoz sebagai pusat penerima data (*centralized backend*).
   * **VPS Target**, digunakan sebagai simulasi server yang akan dipantau, minimal 1 GB RAM.
2. Akses **Root** atau user dengan hak akses `sudo` pada kedua VPS.
3. **IP Address publik** pada VPS Utama.
4. Spesifikasi minimal VPS Utama: **4 GB RAM**, disarankan **8 GB RAM dan 4 vCPU** untuk penggunaan yang lebih nyaman.
5. Ruang disk yang mencukupi pada VPS Utama untuk penyimpanan data monitoring.
6. Port `8080`, `4317`, dan `4318` pada VPS Utama dapat diakses dari internet.

Pada panduan ini, kita akan menggunakan contoh konfigurasi berikut:

| Konfigurasi        | Nilai                  |
| ------------------ | ---------------------- |
| Sistem Operasi     | Ubuntu 24.04 LTS       |
| IP Address VPS Utama | `IP-VPS-UTAMA`        |
| IP Address VPS Target | `IP-VPS-TARGET`      |
| Port UI SigNoz     | `8080`                 |
| Port OTLP gRPC     | `4317`                 |
| Port OTLP HTTP     | `4318`                 |

> **Catatan:** `IP-VPS-UTAMA` dan `IP-VPS-TARGET` hanya digunakan sebagai contoh. Silakan sesuaikan dengan IP Address publik yang digunakan.

## Rangkuman Versi yang Digunakan

Panduan ini menggunakan konfigurasi berikut:

| Komponen       | Versi            |
| -------------- | ---------------- |
| Sistem Operasi | Ubuntu 24.04 LTS |
| Docker Engine  | 29.6.2           |
| Docker Compose | v2 (plugin)      |
| SigNoz         | v0.141.1         |

> **Catatan:** Versi SigNoz dan Docker dapat berubah seiring adanya release terbaru. Jalankan `docker --version` dan periksa versi SigNoz pada menu Settings setelah instalasi untuk memastikan versi yang benar-benar terpasang di server kamu.

---

## 1. Update Sistem

Sebelum melakukan instalasi, lakukan update package pada **VPS Utama** dengan menjalankan perintah berikut:

```bash
apt update && apt upgrade -y
```

Kemudian install package pendukung yang diperlukan:

```bash
apt install -y curl ca-certificates gnupg
```

> **Catatan:** Perintah pada panduan ini diasumsikan dijalankan menggunakan user `root`. Jika menggunakan user biasa, tambahkan `sudo` pada perintah yang membutuhkan hak akses administratif.

## 2. Instalasi Docker

SigNoz dijalankan menggunakan beberapa container, sehingga Docker Engine beserta plugin Docker Compose perlu di-install terlebih dahulu pada **VPS Utama**.

Tambahkan GPG key resmi Docker:

```bash
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
```

Tambahkan repository Docker:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install Docker Engine dan plugin Docker Compose:

```bash
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Pastikan Docker telah berhasil di-install:

```bash
docker --version
docker compose version
```

<p align="center">
  <img width="850" alt="Verifikasi versi Docker dan Docker Compose" src="images/verifikasi-docker.png" style="border-radius: 10px;" />
  <br>
  Gambar 1: Verifikasi versi Docker dan Docker Compose
</p>

Pastikan service Docker berjalan dan aktif ketika server melakukan reboot:

```bash
systemctl enable --now docker
systemctl status docker
```

## 3. Instalasi SigNoz (Docker Standalone)

SigNoz versi terbaru di-install menggunakan tools bernama **foundryctl**, yaitu CLI resmi SigNoz untuk mengatur proses deployment.

Install `foundryctl`:

```bash
curl -fsSL https://signoz.io/foundry.sh | bash
```

Buat direktori kerja, kemudian buat file konfigurasi `casting.yaml`:

```bash
mkdir -p /opt/signoz && cd /opt/signoz
nano casting.yaml
```

Isi dengan konfigurasi berikut, yang menentukan target deployment berupa Docker Compose pada satu mesin (*Docker Standalone*):

```yaml
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

```bash
foundryctl cast -f casting.yaml
```

Perintah `cast` akan memvalidasi Docker pada server, membuat file Docker Compose pada direktori `pours/deployment/`, kemudian menjalankan seluruh container yang dibutuhkan SigNoz (aplikasi SigNoz, ClickHouse, dan OTel Collector).

<p align="center">
  <img width="850" alt="Proses instalasi SigNoz menggunakan foundryctl cast" src="images/proses-instalasi-signoz.png" style="border-radius: 10px;" />
  <br>
  Gambar 2: Proses instalasi SigNoz
</p>

> **Catatan:** Metode instalasi SigNoz dapat berubah seiring adanya release terbaru. Pastikan mengikuti [dokumentasi resmi SigNoz](https://signoz.io/docs/) pada saat instalasi dilakukan.

## 4. Verifikasi Container SigNoz

Pastikan seluruh container SigNoz telah berjalan:

```bash
docker ps
```

Pastikan container berada pada status `Up` dan `healthy`.

<p align="center">
  <img width="900" alt="Verifikasi container SigNoz" src="images/verifikasi-container-signoz.png" style="border-radius: 10px;" />
  <br>
  Gambar 3: Verifikasi container SigNoz
</p>

| Container        | Fungsi                                              |
| ----------------- | ---------------------------------------------------- |
| `signoz`          | Menjalankan aplikasi dan tampilan antarmuka SigNoz  |
| `clickhouse`      | Menyimpan data metrics, traces, dan logs             |
| `otel-collector`  | Menerima data observability melalui protokol OTLP    |

> **Catatan:** Nama dan jumlah container dapat berbeda tergantung versi SigNoz yang digunakan.

Untuk memeriksa log salah satu container, gunakan:

```bash
docker logs -f NAMA-CONTAINER
```

## 5. Penyesuaian SELinux (Khusus Distro Berbasis RHEL)

Ubuntu menggunakan **AppArmor**, bukan SELinux, sehingga langkah pada bagian ini **tidak diperlukan** apabila mengikuti panduan ini pada Ubuntu 24.04. Bagian ini disediakan sebagai referensi tambahan apabila kamu menjalankan Docker Standalone pada distro berbasis RHEL (CentOS, Rocky Linux, atau AlmaLinux) dengan SELinux dalam mode `enforcing`.

Pada distro tersebut, SELinux dapat memblokir akses container ke volume atau port tertentu meskipun konfigurasi Docker sudah benar.

Periksa status SELinux:

```bash
sestatus
```

<p align="center">
  <img width="600" alt="Verifikasi status SELinux" src="images/verifikasi-selinux-status.png" style="border-radius: 10px;" />
  <br>
  Gambar 4: Verifikasi status SELinux
</p>

Jika terdapat proses yang diblokir SELinux, periksa log audit untuk mengetahui penyebabnya:

```bash
ausearch -m avc -ts recent
```

<p align="center">
  <img width="850" alt="Log audit SELinux yang memblokir proses" src="images/log-audit-selinux.png" style="border-radius: 10px;" />
  <br>
  Gambar 5: Log audit SELinux
</p>

Untuk mengizinkan container Docker mengakses volume bind-mount, tambahkan label SELinux `:z` atau `:Z` pada definisi volume di file compose, misalnya:

```yaml
volumes:
  - ./data:/var/lib/clickhouse:Z
```

Label `:Z` memberikan akses privat untuk container tersebut, sedangkan `:z` (huruf kecil) memberikan akses yang dapat digunakan bersama oleh beberapa container.

Apabila SELinux tetap memblokir proses tertentu setelah penyesuaian label volume, policy khusus dapat dibuat menggunakan `audit2allow`:

```bash
ausearch -m avc -ts recent | audit2allow -M signoz-policy
semodule -i signoz-policy.pp
```

> **Catatan:** Penggunaan `audit2allow` sebaiknya dilakukan dengan hati-hati karena dapat memberikan izin yang lebih luas dari yang seharusnya. Pastikan meninjau isi policy yang dihasilkan sebelum menerapkannya, dan lakukan pengujian langsung pada distro terkait karena kebutuhan penyesuaian dapat berbeda tergantung versi SELinux dan Docker yang digunakan.

## 6. Konfigurasi Firewall

SigNoz menggunakan beberapa port yang perlu diizinkan pada firewall **VPS Utama**.

| Port   | Protokol | Fungsi                          |
| ------ | -------- | -------------------------------- |
| `8080` | TCP      | Akses tampilan antarmuka SigNoz |
| `4317` | TCP      | Pengiriman data OTLP gRPC        |
| `4318` | TCP      | Pengiriman data OTLP HTTP        |

Jika menggunakan UFW, jalankan:

```bash
ufw allow 8080/tcp
ufw allow 4317/tcp
ufw allow 4318/tcp
ufw status
```

<p align="center">
  <img width="600" alt="Verifikasi status UFW" src="images/verifikasi-ufw-status.png" style="border-radius: 10px;" />
  <br>
  Gambar 6: Verifikasi status UFW
</p>

> **Catatan:** Karena pada panduan ini VPS Target akan mengirim data melalui **IP publik** (bukan IP privat), port `4317` dan `4318` perlu dapat diakses dari internet. Untuk keamanan, SigNoz self-hosted secara bawaan **tidak memiliki autentikasi** pada endpoint OTLP tersebut. Batasi firewall agar hanya menerima koneksi dari IP Address VPS Target, bukan dari seluruh internet, misalnya:
>
> ```bash
> ufw allow from IP-VPS-TARGET to any port 4317 proto tcp
> ufw allow from IP-VPS-TARGET to any port 4318 proto tcp
> ```

## 7. Mengakses Tampilan SigNoz

Akses SigNoz melalui browser menggunakan alamat berikut:

```text
http://IP-VPS-UTAMA:8080
```

Pada akses pertama, SigNoz akan meminta pembuatan akun administrator. Masukkan nama, email, dan password yang akan digunakan.

<p align="center">
  <img width="850" alt="Halaman pembuatan akun administrator SigNoz" src="images/setup-akun-admin-signoz.png" style="border-radius: 10px;" />
  <br>
  Gambar 7: Pembuatan akun administrator
</p>

Setelah akun berhasil dibuat, tampilan utama SigNoz akan ditampilkan dengan kondisi Quick Stats kosong karena belum ada data yang masuk.

<p align="center">
  <img width="900" alt="Tampilan utama SigNoz dengan Quick Stats kosong" src="images/tampilan-utama-signoz.png" style="border-radius: 10px;" />
  <br>
  Gambar 8: Tampilan utama SigNoz
</p>

> **Catatan:** Gunakan password yang kuat karena tampilan SigNoz dapat diakses melalui internet.

## 8. Instalasi Agent pada VPS Target

Agar SigNoz dapat menampilkan data, **VPS Target** perlu dipasangi **OpenTelemetry Collector** sebagai agent, kemudian diarahkan untuk mengirim data ke **VPS Utama** melalui IP publik.

Login ke **VPS Target**, kemudian update sistem terlebih dahulu:

```bash
apt update && apt upgrade -y
apt install -y curl tar
```

### 8.1 Download OTel Collector Binary

Download OpenTelemetry Collector Contrib untuk Linux AMD64:

```bash
cd /tmp
curl -LO https://github.com/open-telemetry/opentelemetry-collector-releases/releases/latest/download/otelcol-contrib_linux_amd64.tar.gz
```

Ekstrak dan pindahkan binary:

```bash
tar -xzf otelcol-contrib_linux_amd64.tar.gz
mv otelcol-contrib /usr/local/bin/otelcol-contrib
chmod +x /usr/local/bin/otelcol-contrib
```

### 8.2 Membuat Konfigurasi Collector

Buat direktori konfigurasi:

```bash
mkdir -p /etc/otelcol-contrib
nano /etc/otelcol-contrib/config.yaml
```

Isi dengan konfigurasi berikut untuk mengumpulkan host metrics dari VPS Target dan mengirimkannya ke VPS Utama melalui IP publik:

```yaml
receivers:
  hostmetrics:
    collection_interval: 60s
    scrapers:
      cpu: {}
      disk: {}
      load: {}
      filesystem: {}
      memory: {}
      network: {}
      paging: {}
      process:
        mute_process_name_error: true
        mute_process_exe_error: true

processors:
  batch:
    send_batch_size: 1000
    timeout: 10s
  resourcedetection:
    detectors: [env, system]

exporters:
  otlp:
    endpoint: "IP-VPS-UTAMA:4317"
    tls:
      insecure: true

service:
  pipelines:
    metrics:
      receivers: [hostmetrics]
      processors: [batch, resourcedetection]
      exporters: [otlp]
```

> **Catatan:** Ganti `IP-VPS-UTAMA` dengan IP Address publik VPS Utama tempat SigNoz berjalan. Karena komunikasi dilakukan melalui IP publik, opsi `tls.insecure: true` digunakan sebagai contoh sederhana; pada lingkungan produksi, pertimbangkan menambahkan TLS atau membatasi akses melalui firewall seperti pada langkah 6.

### 8.3 Menjalankan Collector sebagai Service

Buat file service systemd agar collector berjalan otomatis:

```bash
nano /etc/systemd/system/otelcol-contrib.service
```

Isi dengan konfigurasi berikut:

```ini
[Unit]
Description=OpenTelemetry Collector Contrib
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/otelcol-contrib --config /etc/otelcol-contrib/config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Reload systemd, kemudian jalankan service:

```bash
systemctl daemon-reload
systemctl enable --now otelcol-contrib
systemctl status otelcol-contrib
```

<p align="center">
  <img width="850" alt="Status service OTel Collector pada VPS Target" src="images/status-service-otelcol.png" style="border-radius: 10px;" />
  <br>
  Gambar 9: Status service OTel Collector
</p>

Pastikan status menunjukkan `active (running)`. Jika terdapat error, periksa log:

```bash
journalctl -u otelcol-contrib -n 50 --no-pager
```

## 9. Verifikasi Data pada SigNoz

Kembali ke tampilan SigNoz pada **VPS Utama**, kemudian masuk ke menu **Infrastructure Monitoring > Hosts**.

Setelah beberapa saat (mengikuti nilai `collection_interval`), VPS Target akan muncul pada daftar host beserta metrik CPU, memory, disk, dan network.

<p align="center">
  <img width="900" alt="VPS Target muncul pada Infrastructure Monitoring SigNoz" src="images/verifikasi-host-infrastructure-monitoring.png" style="border-radius: 10px;" />
  <br>
  Gambar 10: Data VPS Target pada Infrastructure Monitoring
</p>

> **Catatan:** Apabila host tidak muncul, periksa kembali:
> 1. Apakah service `otelcol-contrib` pada VPS Target dalam kondisi `active (running)`.
> 2. Apakah port `4317` pada VPS Utama dapat diakses dari IP Address VPS Target (uji dengan `curl -v telnet://IP-VPS-UTAMA:4317` dari VPS Target).
> 3. Apakah firewall pada VPS Utama sudah mengizinkan koneksi dari IP VPS Target sesuai langkah 6.

## Troubleshooting

### Container SigNoz Tidak Dapat Berjalan

Periksa status dan log container:

```bash
docker ps -a
docker logs NAMA-CONTAINER
```

Pastikan resource server mencukupi, terutama memory, karena SigNoz membutuhkan minimal 4 GB RAM.

### Tampilan SigNoz Tidak Dapat Diakses

Periksa beberapa hal berikut:

1. Pastikan seluruh container berada pada status `Up`.
2. Pastikan port `8080` telah diizinkan pada firewall.
3. Periksa penggunaan port dengan `ss -lntup | grep ':8080'`.

### Data dari VPS Target Tidak Muncul

Periksa beberapa hal berikut:

1. Pastikan endpoint pada `config.yaml` VPS Target sudah sesuai dengan IP publik VPS Utama.
2. Pastikan port `4317`/`4318` pada VPS Utama dapat diakses dari VPS Target, baik dari sisi firewall Ubuntu (UFW) maupun security group/firewall pada layanan Kilat VM.
3. Periksa log service `otelcol-contrib` pada VPS Target melalui `journalctl -u otelcol-contrib`.

## Kesimpulan

SigNoz dapat diinstal menggunakan metode Docker Standalone pada Kilat VM dengan sistem operasi Ubuntu 24.04 LTS. Dengan menjalankan SigNoz melalui `foundryctl` dan menghubungkannya dengan OpenTelemetry Collector pada server lain melalui IP publik, satu VPS Utama dapat digunakan sebagai pusat monitoring untuk memantau kondisi VPS Target.

Dengan mengikuti panduan ini, SigNoz telah berhasil di-install dan terbukti dapat menerima data dari server lain. Agar penggunaan lebih optimal, pastikan akses endpoint OTLP dibatasi hanya untuk IP yang dipercaya, serta lanjutkan eksplorasi fitur SigNoz lainnya seperti logs, traces, dan alert pada artikel terpisah.

---

CloudKilat menyediakan layanan **Kilat VM, hosting, serta berbagai layanan pendukung lainnya** dengan performa yang andal. Layanan CloudKilat juga didukung oleh tim support yang siap membantu dengan respons cepat dan pelayanan selama **7x24 jam**.

Untuk informasi lebih lanjut mengenai layanan CloudKilat, silakan kunjungi [website resmi CloudKilat](https://cloudkilat.id/).

Terima kasih, semoga panduan ini bermanfaat.

## Referensi

- [SigNoz Official Website](https://signoz.io/)
- [SigNoz Documentation](https://signoz.io/docs/)
- [SigNoz GitHub - Foundry](https://github.com/SigNoz/foundry)
- [OpenTelemetry Collector Releases](https://github.com/open-telemetry/opentelemetry-collector-releases)
