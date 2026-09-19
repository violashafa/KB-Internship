# Apa Itu CoreDNS? Mengenal DNS Server Berbasis Plugin

Halo, Kawan Belajar! Pernahkah kamu membutuhkan DNS server sendiri untuk mengelola domain dan DNS record pada server? Atau ingin memiliki DNS server yang dapat dikonfigurasi sesuai kebutuhan tanpa bergantung pada layanan DNS pihak ketiga?

Salah satu software yang dapat digunakan untuk kebutuhan tersebut adalah **CoreDNS**. CoreDNS merupakan DNS server open source yang menggunakan sistem berbasis plugin sehingga dapat dikonfigurasi sesuai dengan kebutuhan pengguna.

<p align="center">
  <img width="271" height="286" alt="logo-coredns" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/logo-coredns.png" style="border-radius: 10px;" />
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

## Kelebihan CoreDNS

- **Berbasis plugin**, sehingga fitur dapat disesuaikan dengan kebutuhan.
- **Konfigurasi sederhana** menggunakan Corefile.
- **Open source** dan dapat digunakan pada berbagai lingkungan, seperti server Linux dan container.

## Kekurangan CoreDNS

- **Tidak menyediakan GUI bawaan**, sehingga konfigurasi umumnya dilakukan melalui file dan terminal.
- **Membutuhkan pemahaman dasar DNS** untuk melakukan konfigurasi.
- **Pengelolaan zone dan DNS record dapat dilakukan secara manual** jika menggunakan konfigurasi berbasis file.
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

### Plugin `file`

Plugin `file` merupakan plugin CoreDNS yang digunakan untuk membaca dan melayani data DNS dari **zone file**. Penggunaan plugin ini dikonfigurasi melalui **Corefile** dengan menentukan lokasi zone file yang akan digunakan.

Pada contoh konfigurasi sebelumnya, CoreDNS menggunakan plugin `file` untuk membaca zone file `db.domainkamu.com` dan melayani DNS record untuk domain `domainkamu.com`.

### Zone File

**Zone file**  merupakan file yang berisi informasi DNS (DNS Record) suatu domain dan digunakan oleh CoreDNS sebagai sumber data DNS yang akan dilayani kepada client.

### DNS Record

DNS record merupakan informasi yang terdapat di dalam zone file dan digunakan untuk menentukan alamat atau fungsi suatu domain. Beberapa record yang umum digunakan antara lain:

| Record | Fungsi |
| ------ | ------ |
| `A` | Mengarahkan domain ke IPv4 Address |
| `AAAA` | Mengarahkan domain ke IPv6 Address |
| `CNAME` | Membuat alias dari suatu domain |
| `MX` | Menentukan mail server |
| `NS` | Menentukan nameserver |
| `TXT` | Menyimpan informasi berupa teks |

## Cara Kerja CoreDNS

CoreDNS bekerja dengan menerima DNS query dari client melalui port `53`, kemudian memproses permintaan tersebut berdasarkan konfigurasi yang terdapat pada **Corefile**.

Pada konfigurasi menggunakan plugin `file`, CoreDNS akan membaca zone file yang telah ditentukan pada Corefile. Zone file tersebut berisi berbagai DNS record yang digunakan untuk menentukan informasi DNS suatu domain. Setelah menemukan informasi yang sesuai dengan permintaan, CoreDNS mengirimkan DNS response kembali kepada client.

Berikut merupakan gambaran alur kerja CoreDNS:

<p align="center">
  <img width="1536" height="1024" alt="carakerja-coredns" src="https://raw.githubusercontent.com/violashafa/KB-Internship/main/CoreDNS/Images/carakerja-coredns.png" style="border-radius: 10px;" />
</p>

Secara berurutan, proses kerja CoreDNS pada gambar tersebut adalah:

1. **DNS Query**  
   Client mengirimkan permintaan DNS untuk mendapatkan informasi suatu domain.

2. **CoreDNS Server**  
   CoreDNS menerima DNS query melalui port `53` dan memproses permintaan tersebut.

3. **Corefile**  
   CoreDNS membaca konfigurasi pada Corefile untuk menentukan zone dan plugin yang digunakan.

4. **Plugin `file`**  
   Plugin `file` membaca data DNS dari zone file yang telah dikonfigurasi.

5. **Zone File**  
   Zone file berisi data DNS untuk domain yang dilayani oleh CoreDNS.

6. **DNS Record**  
   CoreDNS mencocokkan permintaan dengan DNS record yang tersedia, seperti `A`, `AAAA`, `CNAME`, `MX`, `NS`, dan `TXT`.

7. **DNS Response**  
   Setelah mendapatkan informasi yang sesuai, CoreDNS mengirimkan hasil DNS query sebagai DNS response.

8. **Client**  
   Client menerima DNS response dan memperoleh informasi DNS yang diminta.

## Perbedaan BIND9, CoreDNS, dan PowerDNS

Ketiganya merupakan software yang dapat digunakan sebagai DNS server, tetapi memiliki pendekatan konfigurasi yang berbeda.

| DNS Server | Karakteristik |
| ---------- | ------------- |
| **BIND9** | DNS server klasik/standar yang banyak digunakan dan menggunakan konfigurasi BIND serta zone file. |
| **PowerDNS** | DNS server yang dapat digunakan untuk mengelola banyak zone dan dapat menggunakan database seperti MariaDB. PowerDNS juga dapat diintegrasikan dengan GUI/panel pihak ketiga. |
| **CoreDNS** | DNS server modern dengan konfigurasi berbasis plugin dan **Corefile**. |

## CoreDNS pada Kilat VM

CoreDNS dapat dijalankan pada **Kilat VM** dan dikonfigurasi sesuai kebutuhan. Kamu bisa mengatur zone, DNS record, serta konfigurasi DNS lainnya secara mandiri melalui server.

Untuk menggunakan CoreDNS sebagai authoritative DNS server, tentu kamu juga membutuhkan **domain**. CloudKilat menyediakan layanan **domain dan Kilat VM** yang bisa digunakan bersama untuk menjalankan berbagai layanan berbasis internet.

Dengan domain dan Kilat VM, kamu bisa mengelola domain, server, serta konfigurasi DNS sesuai kebutuhan.

## Kapan Menggunakan CoreDNS?

CoreDNS dapat digunakan ketika membutuhkan DNS server yang dapat dikonfigurasi secara mandiri, baik sebagai **authoritative DNS server** maupun untuk kebutuhan DNS lainnya melalui sistem berbasis plugin.

> **Catatan:** Authoritative DNS server adalah DNS server yang menyimpan dan memberikan informasi DNS record secara resmi untuk suatu domain.

## Siap Mencoba CoreDNS?

Setelah memahami konsep dasar CoreDNS, kamu dapat melanjutkan ke panduan instalasi dan konfigurasi CoreDNS pada Kilat VM.

**Panduan Instalasi dan Konfigurasi:**

[Cara Instalasi dan Konfigurasi CoreDNS di Ubuntu 24.04](https://github.com/violashafa/KB-Internship/blob/main/CoreDNS/Cara%20Instalasi%20dan%20Konfigurasi%20CoreDNS%20di%20Ubuntu%2024.04.md)

## Kesimpulan

CoreDNS merupakan DNS server open source yang menggunakan arsitektur berbasis plugin dan **Corefile** sebagai konfigurasi utamanya. CoreDNS dapat digunakan sebagai authoritative DNS server untuk melayani DNS record suatu domain dan dapat dikonfigurasi sesuai kebutuhan pengguna.

Dengan memahami Corefile, zone file, DNS record, dan plugin `file`, pengguna dapat memahami dasar penggunaan CoreDNS sebelum melakukan instalasi dan konfigurasi pada server.

---

CloudKilat menyediakan layanan **Kilat VM, hosting, domain, dan layanan tambahan lainnya** untuk berbagai kebutuhan pengguna. Layanan tersebut juga didukung oleh tim support CloudKilat dengan pelayanan selama **7x24 jam**.

Terima kasih, sekian dan semoga bermanfaat.

## Referensi

- [CoreDNS Official Website](https://coredns.io/)
- [CoreDNS GitHub](https://github.com/coredns/coredns)
- [CoreDNS File Plugin](https://coredns.io/plugins/file/)
