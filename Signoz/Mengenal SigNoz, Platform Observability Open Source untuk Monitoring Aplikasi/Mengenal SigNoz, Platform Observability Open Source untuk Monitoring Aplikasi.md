# Apa Itu SigNoz? Mengenal Platform Observability Berbasis OpenTelemetry

Halo, Kawan Belajar!

Ketika aplikasi mulai berjalan di banyak service atau server, memantau performanya secara manual menjadi semakin sulit. Diperlukan sebuah **observability platform** untuk mengetahui kondisi aplikasi secara menyeluruh, mulai dari kecepatan respons, error yang terjadi, hingga penggunaan resource server. Salah satu platform yang dapat digunakan untuk kebutuhan tersebut adalah **SigNoz**.

Pada artikel ini, kita akan membahas apa itu SigNoz, komponen yang membangunnya, cara kerjanya secara umum, serta kelebihan dan kekurangannya sebelum digunakan pada infrastruktur.

<p align="center"> 
  <img width="850" alt="Logo SigNoz" src="Images/signoz.svg" style="border-radius: 10px;" /> 
  <br> Gambar 1: Logo SigNoz 
</p>

## Apa Itu SigNoz?

SigNoz merupakan platform **observability open source** yang digunakan untuk memantau performa aplikasi dan infrastruktur secara menyeluruh. SigNoz menyatukan tiga jenis data observability dalam satu tampilan, yaitu:

* **Metrics** — data numerik seperti penggunaan CPU, memory, dan latency.
* **Traces** — jejak perjalanan satu request ketika melewati berbagai service pada aplikasi.
* **Logs** — catatan aktivitas yang dihasilkan oleh aplikasi maupun server.

Dengan menyatukan ketiga jenis data tersebut, proses analisis ketika terjadi kendala pada aplikasi dapat dilakukan dalam satu tempat, tanpa perlu berpindah antar tools yang berbeda untuk masing-masing jenis data.

## SigNoz dan OpenTelemetry

SigNoz dibangun di atas standar **OpenTelemetry**, yaitu sebuah proyek open source yang menyediakan spesifikasi umum untuk instrumentasi, pengumpulan, dan pengiriman data observability. Karena mengikuti standar ini, aplikasi yang sudah diinstrumentasi menggunakan OpenTelemetry dapat mengirimkan data ke SigNoz tanpa perlu library atau agent khusus yang terikat pada satu vendor tertentu (*vendor-neutral*).

## Komponen Utama SigNoz

Secara umum, SigNoz terdiri dari beberapa komponen utama yang bekerja sama:

| Komponen | Fungsi |
| --- | --- |
| **SigNoz Frontend & Query Service** | Menyediakan tampilan antarmuka (dashboard) serta melayani query terhadap data yang tersimpan. |
| **OTel Collector** | Menerima data metrics, traces, dan logs yang dikirimkan melalui protokol OTLP dari aplikasi maupun agent, kemudian meneruskannya ke database. |
| **ClickHouse** | Database kolom (*columnar database*) yang digunakan untuk menyimpan seluruh data observability, dirancang untuk menangani data dalam jumlah besar dengan performa query yang cepat. |

## Cara Kerja SigNoz Secara Umum

Secara sederhana, alur kerja SigNoz dapat digambarkan sebagai berikut:

1. Aplikasi atau server diinstrumentasi menggunakan SDK OpenTelemetry, atau dipasangi **OpenTelemetry Collector** sebagai agent.
2. Data metrics, traces, dan logs dikirimkan melalui protokol **OTLP** (OpenTelemetry Protocol) ke endpoint SigNoz, umumnya melalui port `4317` (gRPC) atau `4318` (HTTP).
3. **OTel Collector** pada sisi SigNoz menerima data tersebut, kemudian meneruskannya untuk disimpan pada **ClickHouse**.
4. Pengguna dapat mengakses tampilan SigNoz untuk melihat dashboard, mencari trace tertentu, membaca log, hingga mengatur alert berdasarkan data yang telah tersimpan.

## Kelebihan SigNoz

Beberapa kelebihan SigNoz antara lain:

* **Open source**, sehingga dapat digunakan dan dikembangkan secara bebas.
* **Berbasis OpenTelemetry**, sehingga mengikuti standar instrumentasi yang umum digunakan dan tidak terikat pada satu vendor.
* **All-in-one**, karena metrics, traces, dan logs dapat diakses dalam satu aplikasi.
* **Dapat di-self-host**, sehingga data observability sepenuhnya berada di bawah kendali pengguna.
* **Tidak ada biaya lisensi per host**, karena dapat dijalankan secara mandiri pada server sendiri.

## Kekurangan SigNoz

SigNoz juga memiliki beberapa hal yang perlu diperhatikan:

* Membutuhkan resource yang cukup besar, terutama karena penggunaan ClickHouse sebagai database.
* Memerlukan pemahaman dasar mengenai OpenTelemetry untuk melakukan instrumentasi aplikasi.
* Pengelolaan dilakukan secara mandiri apabila di-self-host, termasuk update, backup, dan keamanan.
* Retention data perlu diatur secara berkala agar penggunaan disk tetap terkendali.

## Kapan Sebaiknya Menggunakan SigNoz?

SigNoz cocok digunakan ketika:

* Membutuhkan monitoring aplikasi dan infrastruktur dalam satu platform, tanpa menggabungkan beberapa tools terpisah.
* Ingin menyimpan data observability pada server sendiri, tanpa bergantung pada layanan pihak ketiga.
* Sudah atau berencana menggunakan OpenTelemetry sebagai standar instrumentasi aplikasi.
* Membutuhkan alternatif self-hosted dari platform observability berbayar, tanpa biaya lisensi per host.

## Kesimpulan

SigNoz merupakan platform observability open source yang menyatukan metrics, traces, dan logs dalam satu tampilan, dibangun di atas standar OpenTelemetry agar tidak terikat pada satu vendor tertentu. Dengan memahami komponen dan cara kerjanya, SigNoz dapat menjadi pilihan untuk memantau performa aplikasi maupun infrastruktur secara mandiri.

---

CloudKilat menyediakan layanan **Kilat VM, hosting, serta berbagai layanan pendukung lainnya** dengan performa yang andal. Layanan CloudKilat juga didukung oleh tim support yang siap membantu dengan respons cepat dan pelayanan selama **7x24 jam**.

Untuk informasi lebih lanjut mengenai layanan CloudKilat, silakan kunjungi [website resmi CloudKilat](https://cloudkilat.id/).

Terima kasih, semoga artikel ini bermanfaat.

## Referensi

- [SigNoz Official Website](https://signoz.io/)
- [SigNoz Documentation](https://signoz.io/docs/)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
