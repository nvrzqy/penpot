<img width="1230" height="305" alt="Penpot" src="https://github.com/user-attachments/assets/841799ec-52df-4f8b-960b-cdc78f3a100d" />

# Penpot Self-Hosting Project

> **Proyek Komunikasi Data dan Jaringan — Self-Hosted Web Application**

Repository ini berisi konfigurasi dan dokumentasi proyek self-hosting **Penpot**, sebuah aplikasi desain UI/UX dan prototyping berbasis web yang bersifat open-source.

## Anggota Kelompok

| No. | Nama | NIM |
|---:|---|---|
| 1 | **Nadya Shafwah Rizalti** | M0403241007 |
| 2 | **Nazwa Nadya Rahma** | M0403241060 |
| 3 | **Zivanka Aurellia Astadewi Maheswari** | M0403241111 |
| 4 | **Azalia Noverizqy Aqila Pramono** | M0403241123 |
| 5 | **Rafi Subhan Jaya Kusuma** | M0403241138 |

## Tentang Penpot

Penpot adalah platform desain digital yang dapat digunakan untuk membuat antarmuka pengguna, prototype interaktif, komponen desain, dan design system. Aplikasi ini berjalan melalui browser dan mendukung proses kerja kolaboratif.

Berbeda dari banyak aplikasi desain berbasis cloud, Penpot menyediakan opsi self-hosting. Organisasi atau pengguna dapat menjalankan Penpot pada infrastruktur yang dikelola sendiri sehingga memiliki kontrol lebih besar terhadap konfigurasi, layanan, dan penyimpanan data.

Kemampuan utama Penpot meliputi:

- Desain UI/UX berbasis web.
- Pembuatan prototype interaktif.
- Components dan reusable assets.
- Design system.
- Pengelolaan project dan file desain.
- Kolaborasi antar pengguna.
- Ekspor hasil desain.
- Self-hosting.

Repository resmi Penpot tersedia di [GitHub Penpot](https://github.com/penpot/penpot).

## Tentang Proyek

Proyek ini dibuat untuk mempelajari penerapan aplikasi web yang di-host secara mandiri. Penpot digunakan sebagai studi kasus karena memiliki beberapa layanan yang saling terhubung, seperti frontend, backend, database, cache, dan layanan ekspor.

Docker dan Docker Compose digunakan untuk menjalankan serta mengelola layanan-layanan tersebut dalam container. Implementasi proyek mencakup penggunaan pada komputer lokal dan server berbasis cloud.

Figma dipilih sebagai aplikasi pembanding karena memiliki fungsi utama yang serupa dalam bidang desain antarmuka, prototyping, dan kolaborasi.

## Tujuan

Tujuan proyek ini adalah:

- Memahami konsep self-hosting aplikasi web.
- Mempelajari penggunaan Docker dan Docker Compose.
- Melakukan instalasi dan konfigurasi Penpot.
- Menerapkan aplikasi web pada infrastruktur server.
- Memahami hubungan antara aplikasi, database, penyimpanan, dan jaringan.
- Mendokumentasikan proses instalasi dan implementasi.
- Mempelajari fitur utama Penpot.
- Melakukan pengujian terhadap fungsi aplikasi.
- Membandingkan Penpot dengan Figma.

## Teknologi yang Digunakan

| Teknologi | Fungsi |
|---|---|
| Penpot | Aplikasi desain dan prototyping |
| Docker | Menjalankan layanan dalam container |
| Docker Compose | Mengelola kumpulan layanan Penpot |
| PostgreSQL | Menyimpan data aplikasi |
| Valkey | Mendukung cache dan koordinasi layanan |
| Ubuntu Server | Sistem operasi server |
| Microsoft Azure | Infrastruktur cloud untuk implementasi server |

## Dokumentasi

1. [Laporan Proyek](docs/report.md)
2. [Panduan Instalasi](installation/installation-guide.md)
3. [Panduan Deployment](deploy/deployement-guide.md)
4. [Docker Compose](deploy/docker-compose.yaml)
5. Application Usage
6. Penpot vs Figma

## Struktur Repository

```text
penpot/
├── deploy/
│   ├── deployement-guide.md
│   ├── deployement-guide-azure.md
│   └── docker-compose.yaml
├── docs/
│   └── report.md
├── installation/
│   └── installation-guide.md
└── README.md
```


## Referensi

- [Penpot](https://penpot.app/)
- [Penpot GitHub Repository](https://github.com/penpot/penpot)
- [Penpot Self-Hosting Guide](https://help.penpot.app/technical-guide/getting-started/)
- [Docker Documentation](https://docs.docker.com/)
- [Figma](https://www.figma.com/)
