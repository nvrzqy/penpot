# Penpot Self-Hosting Project

> **Proyek Komunikasi Data dan Jaringan — Self-Hosted Web Application**

Repository ini berisi dokumentasi dan konfigurasi proyek **self-hosting Penpot**, sebuah platform desain dan prototyping berbasis web yang bersifat open-source.

---

## Deskripsi Tentang Proyek

Proyek ini bertujuan untuk mempelajari proses **deployment dan self-hosting aplikasi web** menggunakan Penpot.

Penpot dipilih sebagai aplikasi yang akan di-deploy karena menyediakan berbagai fitur untuk kebutuhan desain UI/UX, seperti pembuatan desain, prototyping, dan kolaborasi. Sebagai aplikasi open-source, Penpot dapat dijalankan menggunakan infrastruktur yang dikelola sendiri.

Dalam proyek ini, Penpot akan terlebih dahulu diuji dan dikonfigurasi secara lokal menggunakan **WSL2 dan Docker**. Setelah deployment lokal berhasil, aplikasi akan dipindahkan ke **VPS/hosting** sehingga dapat diakses melalui internet.

### Deployment Plan

```text
Development / Testing
        │
        ▼
┌───────────────────┐
│      Windows      │
│       WSL2        │
│      Ubuntu       │
│       Docker      │
│      Penpot       │
└─────────┬─────────┘
          │
          │ Testing
          ▼
┌───────────────────┐
│        VPS        │
│      Docker       │
│      Penpot       │
│  Domain + HTTPS   │
└─────────┬─────────┘
          │
          ▼
       Internet
          │
          ▼
      End Users
```

---

## Tujuan

Tujuan dari proyek ini adalah:

* Mempelajari konsep **self-hosting aplikasi web**.
* Melakukan instalasi dan konfigurasi Penpot.
* Memahami penggunaan Docker dalam deployment aplikasi web.
* Melakukan deployment aplikasi pada VPS/hosting.
* Membuat aplikasi dapat diakses melalui internet.
* Menguji fitur utama Penpot.
* Membandingkan Penpot dengan aplikasi sejenis.
* Mendokumentasikan proses instalasi dan deployment.

---

## Tentang Penpot

**Penpot** adalah platform desain dan prototyping berbasis web yang ditujukan untuk pembuatan desain UI/UX dan kolaborasi.

Beberapa kemampuan Penpot yang akan dipelajari dalam proyek ini meliputi:

* UI/UX design
* Prototyping
* Design components
* Project management
* Collaboration
* Design systems
* Web-based design workflow

Penpot bersifat **open-source** dan dapat di-deploy pada infrastructure sendiri.

### Repository Resmi

[Penpot GitHub Repository](https://github.com/penpot/penpot)

---

## 👥 Anggota Kelompok

| No. | Nama                 | NIM   |
| --: | -------------------- | ----- |
|   1 | **Nadya Shafwah Rizalti** | M0403241007 |
|   2 | **Nazwa Nadya Rahma** | M0403241060 |
|   3 | **Zivanka Aurellia Astadewi Maheswari** | M0403241111 |
|   4 | **Azalia Noverizqy Aqila Pramono** | M0403241123 |
|   4 | **Rafi Subhan Jaya Kusuma** | M0403241138 |

> **Catatan:** Pembagian peran dapat disesuaikan dengan kontribusi masing-masing anggota.

---

## Technology Stack

Teknologi yang direncanakan digunakan dalam proyek:

* **Penpot** — Web application
* **Docker** — Containerization
* **Docker Compose** — Container orchestration
* **WSL2** — Local development environment
* **Ubuntu** — Linux environment
* **VPS/Cloud Hosting** — Production deployment
* **Domain** — Public access
* **HTTPS/SSL** — Secure connection

---

## Deployment

### Local Development

Pada tahap awal, Penpot akan dijalankan pada environment lokal untuk melakukan instalasi dan pengujian.

```text
Windows
└── WSL2
    └── Ubuntu
        └── Docker
            └── Penpot
```

Local deployment digunakan untuk:

* Menguji instalasi.
* Mempelajari konfigurasi Penpot.
* Menguji fitur.
* Menemukan dan menyelesaikan masalah deployment sebelum menggunakan VPS.

### Production Deployment

Setelah pengujian lokal selesai, Penpot akan di-deploy pada VPS.

Target deployment:

```text
Internet
   │
   ▼
Domain
   │
   ▼
VPS
   │
   ├── Reverse Proxy
   │
   ├── HTTPS / SSL
   │
   └── Docker
        └── Penpot
```

Target akhirnya adalah aplikasi dapat diakses melalui internet menggunakan domain yang telah dikonfigurasi.


## 📖 Documentation

Dokumentasi proyek akan mencakup:

1. [Progress Report](progress-report.md)
2. Installation Guide *(akan ditambahkan)*
3. Deployment Guide *(akan ditambahkan)*
4. Application Usage *(akan ditambahkan)*
5. Penpot vs Figma *(akan ditambahkan)*

---

## 📸 Demo

Screenshot dan dokumentasi hasil deployment akan ditambahkan setelah Penpot berhasil dijalankan.

### Local Deployment

> *Screenshot Penpot pada local environment akan ditambahkan.*

### VPS Deployment

> *Screenshot Penpot yang telah dapat diakses melalui internet akan ditambahkan.*

### Application Demo

> *Screenshot fitur Penpot akan ditambahkan.*

---

## 📚 References

* [Penpot GitHub Repository](https://github.com/penpot/penpot)
* [Awesome Selfhosted](https://github.com/Kickball/awesome-selfhosted)
* [Figma](https://www.figma.com/)

---

## 📄 Project Requirements

Proyek ini dibuat sebagai bagian dari tugas **Komunikasi Data dan Jaringan** dengan ketentuan utama:

* Melakukan instalasi aplikasi web.
* Membuat dokumentasi instalasi.
* Mengisi konten aplikasi.
* Melakukan deployment pada hosting/VPS.
* Memastikan aplikasi dapat diakses melalui internet.
* Membuat presentasi.
* Membandingkan aplikasi dengan aplikasi sejenis.

---

## 👩‍💻 Project Status

**Current Status: Initial Setup & Research**

Saat ini proyek berada pada tahap pemilihan aplikasi, research, dan persiapan environment.
