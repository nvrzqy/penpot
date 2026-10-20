# Panduan Instalasi Penpot di Komputer Lokal

## 1. Gambaran Umum

Panduan ini menjelaskan cara menjalankan Penpot secara lokal menggunakan Docker dan Docker Compose. Instalasi ini ditujukan untuk pembelajaran, pengembangan, dan pengujian pada satu komputer. Penpot tidak dihosting pada VPS dan tidak memerlukan domain, alamat IP publik, reverse proxy, atau HTTPS.

Setelah instalasi selesai, Penpot dapat diakses melalui:

```text
http://localhost:9001
```

Konfigurasi yang digunakan adalah [`docker-compose.yaml`](../deploy/docker-compose.yaml) yang tersedia di repository ini.

## 2. Arsitektur Instalasi Lokal

```text
Komputer lokal
├── Docker
│   ├── Penpot Frontend
│   ├── Penpot Backend
│   ├── PostgreSQL
│   ├── Valkey
│   └── Layanan pendukung Penpot
└── Browser → http://localhost:9001
```

Seluruh layanan berjalan sebagai container pada komputer yang sama. PostgreSQL, Valkey, dan layanan internal lainnya berkomunikasi melalui jaringan internal Docker dan tidak perlu dibuka ke internet.

## 3. Prasyarat

Siapkan perangkat berikut:

- Komputer Windows, macOS, atau Linux 64-bit.
- RAM dan ruang penyimpanan yang cukup untuk menjalankan seluruh container Penpot.
- Koneksi internet saat pertama kali mengunduh image Docker.
- Git untuk mengambil repository, atau salinan repository yang sudah tersedia.
- Browser modern.
- Docker Desktop pada Windows/macOS, atau Docker Engine dan Docker Compose v2 pada Linux.

Pada Windows, pastikan Docker Desktop sudah berjalan dan menggunakan backend WSL 2. Perintah pada panduan ini dapat dijalankan melalui PowerShell, Windows Terminal, atau terminal WSL dari direktori repository.

## 4. Menyiapkan Docker

### 4.1 Windows dan macOS

1. Instal Docker Desktop.
2. Jalankan Docker Desktop hingga status Docker Engine menunjukkan bahwa layanan siap digunakan.
3. Buka terminal dan periksa instalasi:

```bash
docker --version
docker compose version
```

### 4.2 Linux

Instal Docker Engine dan plugin Docker Compose menggunakan petunjuk yang sesuai dengan distribusi Linux. Setelah itu, periksa instalasi:

```bash
docker --version
docker compose version
```

Jika akun pengguna belum memiliki akses ke Docker, jalankan perintah Docker menggunakan `sudo` atau konfigurasikan grup `docker` sesuai kebijakan sistem.

## 5. Menyiapkan Repository

Jika repository belum tersedia, clone repository lalu masuk ke direktorinya:

```bash
git clone <URL_REPOSITORY>
cd penpot
```

Jika repository sudah tersedia, cukup buka terminal pada folder `penpot`.

Pastikan file Compose tersedia:

```bash
ls deploy/docker-compose.yaml
```

Pada PowerShell, pemeriksaan yang setara dapat dilakukan dengan:

```powershell
Get-Item deploy\docker-compose.yaml
```

## 6. Memeriksa Konfigurasi Lokal

File Compose pada repository ini sudah menggunakan URL lokal:

```yaml
PENPOT_PUBLIC_URI: http://localhost:9001
```

Frontend Penpot dipublikasikan pada port `9001` komputer:

```yaml
ports:
  - 9001:8080
```

Tidak perlu mengganti nilai tersebut untuk penggunaan lokal biasa.

Jika Penpot harus benar-benar hanya menerima koneksi dari komputer yang sama, ubah pemetaan port frontend menjadi:

```yaml
ports:
  - "127.0.0.1:9001:8080"
```

Binding `127.0.0.1` mencegah layanan menerima koneksi melalui alamat jaringan lokal komputer. Jangan membuka port `9001` pada router atau melakukan port forwarding untuk instalasi lokal.

> **Catatan keamanan:** Konfigurasi lokal menggunakan pengaturan yang memudahkan pengujian, termasuk cookie non-secure dan email verifikasi yang dinonaktifkan. Jangan gunakan konfigurasi ini sebagai deployment produksi yang dapat diakses dari internet.

## 7. Menjalankan Penpot

Dari root repository `penpot`, jalankan:

```bash
docker compose -p penpot -f deploy/docker-compose.yaml up -d
```

Docker akan mengunduh image yang diperlukan, membuat jaringan dan volume, lalu menjalankan seluruh layanan di latar belakang. Proses pertama dapat membutuhkan waktu beberapa menit.

Jika menggunakan Linux dan Docker memerlukan hak administrator:

```bash
sudo docker compose -p penpot -f deploy/docker-compose.yaml up -d
```

## 8. Memverifikasi Instalasi

Periksa status container:

```bash
docker compose -p penpot -f deploy/docker-compose.yaml ps
```

Tunggu hingga layanan utama berjalan. Jika ada container yang berhenti atau terus melakukan restart, periksa log:

```bash
docker compose -p penpot -f deploy/docker-compose.yaml logs --tail=100
```

Untuk mengikuti log backend secara langsung:

```bash
docker compose -p penpot -f deploy/docker-compose.yaml logs -f penpot-backend
```

Tekan `Ctrl+C` untuk berhenti mengikuti log tanpa menghentikan container.

## 9. Mengakses Penpot

Buka browser pada komputer yang sama, lalu kunjungi:

```text
http://localhost:9001
```

Lakukan pengujian sederhana:

1. Pastikan halaman Penpot dapat dimuat.
2. Buat akun lokal jika halaman registrasi tersedia.
3. Buat satu project dan file desain.
4. Tambahkan objek sederhana lalu simpan.
5. Muat ulang halaman dan pastikan data masih tersedia.

## 10. Mengelola Layanan

### 10.1 Melihat Status

```bash
docker compose -p penpot -f deploy/docker-compose.yaml ps
```

### 10.2 Menghentikan Sementara

```bash
docker compose -p penpot -f deploy/docker-compose.yaml stop
```

Perintah ini menghentikan container tanpa menghapus data.

### 10.3 Menjalankan Kembali

```bash
docker compose -p penpot -f deploy/docker-compose.yaml start
```

### 10.4 Menghentikan dan Menghapus Container

```bash
docker compose -p penpot -f deploy/docker-compose.yaml down
```

Perintah ini menghapus container dan jaringan Compose, tetapi mempertahankan named volume yang menyimpan data.

## 11. Penyimpanan Data

Data Penpot disimpan dalam named volume Docker, termasuk database PostgreSQL dan aset desain. Lihat volume yang tersedia dengan:

```bash
docker volume ls
```

Membuat ulang container dengan `up -d` tidak otomatis menghapus named volume, sehingga data biasanya tetap tersedia.

Untuk menghapus instalasi sekaligus seluruh data lokalnya, gunakan:

```bash
docker compose -p penpot -f deploy/docker-compose.yaml down -v
```

> **Peringatan:** Opsi `-v` menghapus database dan aset Penpot secara permanen. Jalankan hanya jika seluruh data lokal memang tidak diperlukan lagi.

## 12. Troubleshooting

| Permasalahan | Kemungkinan penyebab | Solusi |
|---|---|---|
| Perintah `docker` tidak dikenali | Docker belum terinstal atau terminal belum dimulai ulang | Instal/jalankan Docker Desktop, lalu buka ulang terminal. |
| Docker daemon tidak dapat dihubungi | Docker Desktop atau layanan Docker belum berjalan | Jalankan Docker dan tunggu hingga engine siap. |
| Port `9001` sudah digunakan | Aplikasi lain memakai port yang sama | Hentikan aplikasi tersebut atau ganti port host, misalnya `9002:8080`, lalu sesuaikan `PENPOT_PUBLIC_URI` menjadi `http://localhost:9002`. |
| Halaman belum dapat dibuka | Container masih melakukan inisialisasi | Tunggu beberapa menit, periksa status, lalu lihat log container. |
| Container terus restart | Image, konfigurasi, atau resource komputer bermasalah | Periksa log dan pastikan RAM serta ruang penyimpanan mencukupi. |
| Data tidak ditemukan | Project Compose atau volume yang digunakan berbeda | Selalu gunakan nama project `-p penpot` dan periksa daftar volume. |

Perintah diagnostik tambahan:

```bash
docker ps -a
docker compose -p penpot -f deploy/docker-compose.yaml config
docker compose -p penpot -f deploy/docker-compose.yaml logs --tail=100
```

## 13. Perbedaan Instalasi Lokal dan Deployment

| Aspek | Instalasi lokal | Deployment VPS/Azure |
|---|---|---|
| Lokasi aplikasi | Komputer sendiri | Server cloud/VPS |
| Alamat akses | `http://localhost:9001` | IP publik atau domain |
| Akses internet publik | Tidak diperlukan | Diperlukan |
| Domain dan HTTPS | Tidak diperlukan | Direkomendasikan |
| Firewall/NSG cloud | Tidak diperlukan | Perlu dikonfigurasi |
| Tujuan | Belajar dan pengujian | Akses pengguna melalui jaringan/internet |

Untuk menjalankan Penpot pada server, lihat [Panduan Deployment Umum](../deploy/deployement-guide.md) atau [Panduan Deployment Azure](../deploy/deployement-guide-azure.md).

## Referensi

1. [Dokumentasi Docker Penpot](https://help.penpot.app/technical-guide/getting-started/docker/)
2. [Dokumentasi konfigurasi Penpot](https://help.penpot.app/technical-guide/configuration/)
3. [Konfigurasi Docker Compose pada repository ini](../deploy/docker-compose.yaml)
