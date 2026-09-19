# Apa Itu CoreDNS? Mengenal DNS Server Berbasis Plugin

Halo, Kawan Belajar! Pernahkah kamu membutuhkan DNS server sendiri untuk mengelola domain dan DNS record pada server? Atau ingin memiliki DNS server yang dapat dikonfigurasi sesuai kebutuhan tanpa bergantung pada layanan DNS pihak ketiga?

Salah satu software yang dapat digunakan untuk kebutuhan tersebut adalah **CoreDNS**. CoreDNS merupakan DNS server open source yang menggunakan sistem berbasis plugin sehingga dapat dikonfigurasi sesuai dengan kebutuhan pengguna.

<p align="center">
  <img width="271" height="286" alt="logo-coredns" src="https://github.com/user-attachments/assets/89bdd4bb-3963-4c46-8ec6-64aac97ac2c9" style="border-radius: 10px;" />
  <br>
  Logo CoreDNS
</p>

## Mengenal CoreDNS

CoreDNS adalah **DNS server open source** yang digunakan untuk menangani permintaan DNS (*Domain Name System*). CoreDNS menggunakan sistem berbasis plugin sehingga fungsionalitasnya dapat disesuaikan melalui konfigurasi **Corefile**.

CoreDNS dapat digunakan sebagai **authoritative DNS server** untuk melayani DNS record suatu domain maupun sebagai DNS server yang meneruskan permintaan DNS ke server lain sesuai konfigurasi yang digunakan.

## Fungsi CoreDNS

Beberapa fungsi CoreDNS antara lain:

* **Authoritative DNS Server**, untuk menyediakan informasi DNS secara resmi untuk suatu domain.
* **Mengelola DNS Zone**, dengan menentukan zone yang akan dilayani oleh CoreDNS.
* **Melayani DNS Record**, seperti `A`, `AAAA`, `CNAME`, `MX`, `NS`, dan `TXT`.
* **DNS Forwarding**, untuk meneruskan permintaan DNS ke DNS server lain melalui plugin yang sesuai.
* **Plugin-based**, sehingga fitur CoreDNS dapat disesuaikan dengan kebutuhan melalui plugin.

## Komponen Utama CoreDNS

Dalam penggunaannya, terdapat beberapa komponen yang perlu dipahami:

### Corefile

**Corefile** merupakan file konfigurasi utama CoreDNS yang digunakan untuk menentukan zone dan plugin yang digunakan.

Contoh sederhana:

```text
domainkamu.com:53 {
    file /etc/coredns/zones/db.domainkamu.com
}
```

### Zone File

**Zone file** merupakan file yang berisi DNS record untuk suatu domain. Zone file digunakan oleh CoreDNS untuk menyimpan informasi DNS yang akan dilayani kepada client.

Contohnya:

```text
domainkamu.com.    IN    A    IP-VPS
www                IN    A    IP-VPS
```

### DNS Record

DNS record merupakan informasi yang digunakan untuk menentukan alamat atau fungsi suatu domain. Beberapa record yang umum digunakan antara lain:

| Record | Fungsi |
| ------ | ------ |
| `A` | Mengarahkan domain ke IPv4 Address |
| `AAAA` | Mengarahkan domain ke IPv6 Address |
| `CNAME` | Membuat alias dari suatu domain |
| `MX` | Menentukan mail server |
| `NS` | Menentukan nameserver |
| `TXT` | Menyimpan informasi berupa teks |

### Plugin `file`

Plugin `file` digunakan untuk membuat CoreDNS membaca dan melayani data DNS yang disimpan dalam **zone file**. Zone file berisi berbagai **DNS record**, seperti `A`, `NS`, dan `CNAME`, yang digunakan CoreDNS untuk memberikan jawaban atas permintaan DNS.
Konfigurasi plugin `file` ditentukan melalui **Corefile**, yang menunjukkan lokasi zone file yang akan digunakan.

Alur sederhananya:

```text
Corefile
   ↓
file plugin
   ↓
Zone File
   ↓
DNS Records
   ↓
DNS Response
```

## Cara Kerja CoreDNS

Ketika client mengirimkan DNS query, CoreDNS menerima permintaan tersebut melalui port `53`.

CoreDNS kemudian menggunakan konfigurasi pada **Corefile** untuk menentukan bagaimana query tersebut diproses. Jika menggunakan plugin `file`, CoreDNS akan membaca informasi DNS dari zone file yang telah dikonfigurasi dan memberikan DNS response kepada client.

```text
DNS Query
    ↓
CoreDNS :53
    ↓
Corefile
    ↓
file plugin
    ↓
Zone File
    ↓
DNS Record
    ↓
DNS Response
```

## Perbedaan BIND9, CoreDNS, dan PowerDNS

Ketiganya merupakan software yang dapat digunakan sebagai DNS server, tetapi memiliki pendekatan konfigurasi yang berbeda.

| DNS Server | Karakteristik |
| ---------- | ------------- |
| **BIND9** | DNS server klasik/standar yang banyak digunakan dan menggunakan konfigurasi BIND serta zone file. |
| **PowerDNS** | DNS server yang dapat digunakan untuk mengelola banyak zone dan dapat menggunakan database seperti MariaDB. PowerDNS juga dapat diintegrasikan dengan GUI/panel pihak ketiga. |
| **CoreDNS** | DNS server modern dengan konfigurasi berbasis plugin dan **Corefile**. |

## CoreDNS pada Kilat VM

CoreDNS dapat dijalankan pada **Kilat VM** dan dikonfigurasi secara mandiri sesuai kebutuhan. Pengguna memiliki akses untuk mengatur konfigurasi DNS, zone, dan DNS record pada server.

Namun, karena konfigurasi dilakukan secara mandiri, pengguna perlu memahami dasar DNS serta memperhatikan konfigurasi keamanan dan ketersediaan server.

## Kapan Menggunakan CoreDNS?

CoreDNS dapat digunakan ketika membutuhkan:

* DNS server yang dapat dikonfigurasi secara mandiri.
* **Authoritative DNS server** untuk suatu domain.
* Konfigurasi DNS berbasis file melalui **Corefile** dan zone file.
* DNS server dengan arsitektur berbasis plugin.
* DNS server yang dapat disesuaikan dengan kebutuhan melalui plugin.

> **Catatan:** Authoritative DNS server adalah DNS server yang menyimpan dan memberikan informasi DNS record secara resmi untuk suatu domain.

## Siap Mencoba CoreDNS?

Setelah memahami konsep dasar CoreDNS, kamu dapat melanjutkan ke panduan instalasi dan konfigurasi CoreDNS pada Kilat VM.

**Panduan Instalasi dan Konfigurasi:**

[Cara Instalasi dan Konfigurasi CoreDNS di Ubuntu 24.04](https://github.com/violashafa/KB-Internship/blob/main/CoreDNS/Cara%20Instalasi%20dan%20Konfigurasi%20CoreDNS%20di%20Ubuntu%2024.04.md)

## Kesimpulan

CoreDNS merupakan DNS server open source yang menggunakan arsitektur berbasis plugin dan **Corefile** sebagai konfigurasi utamanya. CoreDNS dapat digunakan sebagai authoritative DNS server untuk melayani DNS record suatu domain dan dapat dikonfigurasi sesuai kebutuhan pengguna.

Dengan memahami Corefile, zone file, DNS record, dan plugin `file`, pengguna dapat memahami dasar penggunaan CoreDNS sebelum melakukan instalasi dan konfigurasi pada server.

---

CloudKilat menyediakan layanan **server, hosting, dan layanan tambahan lainnya** yang memiliki performa andal. Layanan tersebut juga didukung oleh tim support CloudKilat dengan respons cepat dan pelayanan terbaik selama **7x24 jam**.

Terima kasih, sekian dan semoga bermanfaat.

## Referensi

- [CoreDNS Official Website](https://coredns.io/)
- [CoreDNS GitHub](https://github.com/coredns/coredns)
- [CoreDNS File Plugin](https://coredns.io/plugins/file/)
