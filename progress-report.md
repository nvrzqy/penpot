# Progress Report

## Self-Hosting Penpot

### 1. Informasi Proyek

| Keterangan          | Detail                                         |
| ------------------- | ---------------------------------------------- |
| Aplikasi            | Penpot                                         |
| Kategori            | Collaborative Design / UI Design & Prototyping |
| Repository          | https://github.com/penpot/penpot               |
| Aplikasi Pembanding | Figma                                          |
| Deployment Awal     | WSL2 / Ubuntu                                  |
| Deployment Final    | Hosting/VPS                                    |

---

## 2. Pemilihan Aplikasi

Aplikasi yang dipilih untuk proyek self-hosting adalah **Penpot**, sebuah platform desain dan prototyping berbasis web yang bersifat open-source dan dapat di-host secara mandiri.

Penpot dipilih karena memiliki fungsi yang relevan dengan kebutuhan desain UI/UX dan menyediakan fitur seperti pembuatan desain, prototyping, kolaborasi, serta pengelolaan proyek desain. Selain itu, Penpot dapat dijalankan menggunakan infrastruktur sendiri sehingga sesuai dengan tujuan proyek untuk mempelajari proses self-hosting aplikasi web.

Repository resmi Penpot:

> https://github.com/penpot/penpot

### Alasan Pemilihan Penpot

Beberapa alasan pemilihan Penpot adalah:

1. Penpot merupakan aplikasi web open-source yang dapat di-self-host.
2. Penpot memiliki fitur yang cukup lengkap untuk digunakan sebagai aplikasi web yang didemonstrasikan.
3. Penpot memiliki aplikasi pembanding yang jelas, yaitu **Figma**.
4. Penpot memiliki penggunaan yang relevan dengan kebutuhan desain UI/UX.
5. Deployment Penpot dapat digunakan sebagai sarana untuk mempelajari Docker dan arsitektur aplikasi web.
6. Aplikasi dapat dikembangkan lebih lanjut dengan deployment pada VPS agar dapat diakses melalui internet.

---

## 3. Aplikasi Pembanding

Aplikasi yang digunakan sebagai pembanding adalah **Figma**.

Figma merupakan platform desain berbasis web yang banyak digunakan untuk membuat desain UI, prototyping, dan melakukan kolaborasi secara online.

Perbandingan Penpot dan Figma nantinya akan dilakukan berdasarkan beberapa aspek, antara lain:

| Aspek              | Penpot                             | Figma                              |
| ------------------ | ---------------------------------- | ---------------------------------- |
| Jenis aplikasi     | Collaborative design & prototyping | Collaborative design & prototyping |
| Berbasis web       | Ya                                 | Ya                                 |
| Open-source        | Ya                                 | Tidak                              |
| Self-hosting       | Ya                                 | Tidak                              |
| Kolaborasi         | Ya                                 | Ya                                 |
| Prototyping        | Ya                                 | Ya                                 |
| Deployment sendiri | Ya                                 | Tidak                              |
| Model layanan      | Self-hosted / cloud                | Cloud/SaaS                         |

Perbandingan fitur dan pengalaman penggunaan akan dilakukan lebih lanjut setelah Penpot berhasil dijalankan dan digunakan.

---

## 4. Analisis Kebutuhan

Sebelum melakukan deployment, dilakukan analisis awal terhadap kebutuhan Penpot.

### 4.1 Environment

Untuk tahap awal, Penpot akan dijalankan secara lokal menggunakan:

* Windows
* WSL2
* Ubuntu
* Docker
* Docker Compose

Deployment lokal digunakan untuk mempelajari proses instalasi, konfigurasi, dan menjalankan Penpot sebelum aplikasi dipindahkan ke VPS.

### 4.2 Docker

Penpot menggunakan beberapa komponen aplikasi yang dapat dijalankan menggunakan container. Oleh karena itu, Docker digunakan untuk mempermudah proses deployment dan pengelolaan service.

Secara umum, deployment dapat digambarkan sebagai:

```text
┌──────────────────────┐
│    Windows Host      │
│                      │
│  ┌────────────────┐  │
│  │      WSL2      │  │
│  │    Ubuntu      │  │
│  │                │  │
│  │ ┌────────────┐ │  │
│  │ │   Docker   │ │  │
│  │ │            │ │  │
│  │ │  ┌──────┐  │ │  │
│  │ │  │Penpot│  │ │  │
│  │ │  └──────┘  │ │  │
│  │ └────────────┘ │  │
│  └────────────────┘  │
└──────────────────────┘
           │
           ▼
      Web Browser
```

### 4.3 Kebutuhan Resource

Kebutuhan resource akan disesuaikan dengan dokumentasi resmi Penpot dan hasil pengujian saat deployment.

Resource yang akan diperhatikan meliputi:

* CPU
* RAM
* Storage
* Docker
* Database
* Network

Pengujian resource akan dilakukan setelah Penpot berhasil dijalankan pada environment lokal.

---

## 5. Rencana Instalasi Lokal

Instalasi lokal belum dilakukan pada tahap progress ini.

Rencana instalasi yang akan dilakukan adalah:

```text
WSL2
  ↓
Ubuntu
  ↓
Install / konfigurasi Docker
  ↓
Download repository / deployment configuration Penpot
  ↓
Konfigurasi environment
  ↓
Menjalankan container Penpot
  ↓
Membuka Penpot melalui browser
  ↓
Pengujian fitur
```

### Target Instalasi

Target dari instalasi lokal adalah agar Penpot dapat diakses melalui browser pada environment lokal, misalnya melalui alamat:

```text
http://localhost:<port>
```

Port aktual akan mengikuti konfigurasi deployment Penpot yang digunakan.

### Status

**Status: Belum dilakukan**

Tahap selanjutnya adalah melakukan instalasi dan konfigurasi Penpot pada WSL2.

---

## 6. Rencana Pengujian

Setelah Penpot berhasil dijalankan, beberapa fitur akan diuji untuk memastikan aplikasi dapat digunakan dengan baik.

Pengujian yang direncanakan:

* Membuat akun/user
* Login
* Membuat project
* Membuat file/design
* Membuat objek desain
* Menggunakan fitur prototyping
* Menguji penyimpanan project
* Menguji fitur kolaborasi
* Menguji akses dari browser

Hasil pengujian akan dicatat dan digunakan sebagai bahan dokumentasi serta presentasi.

---

## 7. Rencana Deployment ke VPS

Setelah deployment lokal berhasil, aplikasi akan dipindahkan ke server/hosting agar dapat diakses melalui internet.

Arsitektur yang direncanakan:

```text
                    Internet
                       │
                       ▼
                 Domain / URL
                       │
                       ▼
              ┌────────────────┐
              │      VPS       │
              │                │
              │ Docker         │
              │   └─ Penpot    │
              │                │
              └────────────────┘
                       │
                       ▼
                 Penpot Web App
```

Tahapan deployment VPS yang direncanakan:

1. Menyiapkan VPS.
2. Menginstall Docker dan dependency yang diperlukan.
3. Melakukan deployment Penpot.
4. Melakukan konfigurasi network.
5. Menghubungkan domain/subdomain.
6. Mengatur HTTPS/SSL.
7. Melakukan pengujian dari jaringan eksternal.
8. Memastikan aplikasi dapat diakses dengan lancar melalui internet.

---

## 8. Rencana Pengisian Konten

Setelah aplikasi berhasil di-deploy, Penpot akan diisi dengan konten untuk kebutuhan demonstrasi.

Contoh konten:

* Project desain website
* Beberapa halaman UI
* Komponen UI
* Prototype sederhana
* Contoh desain landing page
* Contoh desain aplikasi mobile

Konten tersebut akan digunakan saat demo untuk menunjukkan fitur utama Penpot.

---

## 9. Rencana Dokumentasi

Dokumentasi akhir akan mencakup:

1. Pengenalan Penpot.
2. Alasan pemilihan aplikasi.
3. Kebutuhan sistem.
4. Persiapan environment.
5. Instalasi Penpot.
6. Konfigurasi aplikasi.
7. Deployment pada VPS.
8. Konfigurasi domain dan HTTPS.
9. Penggunaan fitur utama.
10. Pengujian aplikasi.
11. Perbandingan Penpot dengan Figma.
12. Kesimpulan.

---

## 10. Status Progress

| Tahap                          | Status        |
| ------------------------------ | ------------- |
| Menentukan aplikasi            | ✅ Selesai     |
| Menentukan aplikasi pembanding | ✅ Selesai     |
| Mempelajari repository Penpot  | ✅ Selesai     |
| Analisis awal kebutuhan        | 🔄 Berjalan   |
| Menyiapkan WSL2                | 🔄 Berjalan   |
| Instalasi Docker               | ⬜ Belum       |
| Instalasi Penpot lokal         | ⬜ Belum       |
| Pengujian Penpot               | ⬜ Belum       |
| Pengisian konten               | ⬜ Belum       |
| Deployment VPS                 | ⬜ Belum       |
| Domain & HTTPS                 | ⬜ Belum       |
| Perbandingan dengan Figma      | 🔄 Riset awal |
| Dokumentasi final              | ⬜ Belum       |
| PPT presentasi                 | ⬜ Belum       |

---

## 11. Next Step

Prioritas pengerjaan selanjutnya:

1. Menyiapkan WSL2 Ubuntu.
2. Memastikan Docker dapat berjalan pada WSL2.
3. Melakukan instalasi Penpot secara lokal.
4. Memastikan Penpot dapat diakses melalui browser.
5. Menguji fitur-fitur utama.
6. Mendokumentasikan proses instalasi menggunakan screenshot.
7. Menyiapkan deployment Penpot pada VPS.
8. Melakukan konfigurasi agar aplikasi dapat diakses melalui internet.
9. Melengkapi perbandingan Penpot dengan Figma.
10. Menyusun dokumentasi dan PPT final.
