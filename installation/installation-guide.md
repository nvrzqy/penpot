# Panduan Instalasi Penpot Self-Hosted Menggunakan Docker

## 1. Gambaran Umum

Penpot merupakan platform desain UI/UX berbasis web yang bersifat *open-source*. Aplikasi ini memungkinkan pengguna membuat desain antarmuka, prototipe interaktif, dan sistem desain melalui browser.

Panduan ini menjelaskan cara memasang Penpot pada Virtual Private Server (VPS) menggunakan Docker dan Docker Compose. Metode ini dapat digunakan pada berbagai penyedia VPS, seperti layanan cloud publik, server virtual pribadi, maupun server Linux yang dikelola sendiri.

### Tujuan

- Menjalankan Penpot pada server yang dapat diakses melalui jaringan.
- Menggunakan Docker Compose untuk mempermudah instalasi dan pengelolaan layanan.
- Mengonfigurasi database serta layanan pendukung Penpot.
- Memverifikasi bahwa aplikasi dapat diakses melalui browser.
- Menyediakan petunjuk pemeliharaan dan penghapusan instalasi.

## 2. Arsitektur Sistem

Penpot terdiri dari beberapa layanan yang bekerja bersama dalam satu lingkungan deployment.

| Komponen | Fungsi |
|---|---|
| VPS | Menyediakan sumber daya komputasi untuk menjalankan aplikasi. |
| Docker Engine | Menjalankan layanan dalam container. |
| Docker Compose | Mengelola konfigurasi dan siklus hidup container. |
| Penpot Frontend | Menyediakan antarmuka web untuk pengguna. |
| Penpot Backend | Menangani pemrosesan permintaan dan logika aplikasi. |
| PostgreSQL | Menyimpan data aplikasi dan proyek desain. |
| Valkey | Mendukung layanan internal seperti cache dan koordinasi notifikasi. |
| Penpot Exporter | Mendukung proses ekspor desain. |
| Layanan pendukung lainnya | Menyediakan fitur tambahan sesuai konfigurasi dan versi Penpot. |

Frontend berkomunikasi dengan backend, sedangkan backend terhubung ke PostgreSQL dan Valkey melalui jaringan internal Docker. Karena itu, layanan database dan internal lainnya tidak perlu dibuka langsung ke internet.

## 3. Prasyarat

### 3.1 Kebutuhan Server

Siapkan VPS dengan persyaratan berikut:

- Sistem operasi Linux, misalnya Ubuntu Server LTS.
- Hak akses administrator melalui `sudo`.
- Koneksi internet untuk mengunduh paket dan image Docker.
- Alamat IP yang dapat dijangkau pengguna, atau domain yang diarahkan ke server.
- RAM, CPU, dan kapasitas penyimpanan yang mencukupi untuk seluruh layanan Penpot.

Kebutuhan sumber daya bergantung pada jumlah pengguna, ukuran proyek desain, dan beban kerja. Untuk menentukan spesifikasi VPS, periksa dokumentasi resmi Penpot dan sesuaikan dengan kebutuhan penggunaan.

### 3.2 Perangkat yang Dibutuhkan

- Terminal pada komputer administrator.
- SSH client untuk mengakses server.
- Docker Engine.
- Docker Compose v2.
- Browser modern untuk mengakses Penpot.

### 3.3 Port Jaringan

Port yang dibutuhkan bergantung pada konfigurasi deployment.

| Port | Fungsi |
|---|---|
| 22/TCP | Akses SSH untuk administrasi server. |
| 9001/TCP | Akses HTTP Penpot jika menggunakan pemetaan port langsung pada contoh konfigurasi ini. |
| 80/TCP | HTTP untuk konfigurasi web server atau reverse proxy. |
| 443/TCP | HTTPS untuk akses web yang aman. |

Port 9001 hanya diperlukan jika konfigurasi Compose mengekspos port tersebut. Jika Penpot ditempatkan di belakang reverse proxy, pengguna umumnya mengakses aplikasi melalui port 80 atau 443.

Batasi akses SSH ke alamat IP tepercaya jika memungkinkan. Jangan membuka port PostgreSQL atau layanan internal lainnya ke internet.

## 4. Tahapan Instalasi

### 4.1 Mengakses VPS melalui SSH

Pastikan VPS sudah aktif dan informasi berikut tersedia:

- Alamat IP server.
- Username administrator.
- Password atau private key untuk autentikasi SSH.

Hubungkan ke server menggunakan:

```bash
ssh <USERNAME>@<IP_SERVER>
```

Jika menggunakan private key:

```bash
ssh -i /path/to/private_key <USERNAME>@<IP_SERVER>
```

Ganti placeholder sesuai konfigurasi VPS. Apabila menggunakan private key pada Linux atau WSL, atur izin file agar hanya dapat dibaca oleh pemiliknya:

```bash
chmod 600 /path/to/private_key
```

Setelah berhasil masuk, terminal akan menampilkan shell dari server Linux.

### 4.2 Memperbarui Sistem

Perbarui daftar paket dan paket sistem:

```bash
sudo apt update
sudo apt upgrade -y
```

Perintah ini ditujukan untuk distribusi berbasis Debian atau Ubuntu. Jika menggunakan distribusi Linux lain, gunakan pengelola paket yang sesuai.

### 4.3 Memasang Docker dan Docker Compose

Pada Ubuntu, Docker dapat dipasang melalui paket berikut:

```bash
sudo apt install -y docker.io docker-compose-v2 curl
```

Aktifkan layanan Docker:

```bash
sudo systemctl enable --now docker
```

Periksa apakah instalasi berhasil:

```bash
sudo docker --version
sudo docker compose version
```

Kedua perintah harus menampilkan informasi versi.

Untuk deployment produksi, pertimbangkan menggunakan repositori resmi Docker agar memperoleh versi Engine dan Compose yang sesuai dengan kebutuhan sistem.

### 4.4 Mengunduh Konfigurasi Penpot

Buat direktori khusus untuk Penpot:

```bash
mkdir -p ~/penpot
cd ~/penpot
```

Unduh file Docker Compose dari repository resmi:

```bash
curl -fsSL \
  https://raw.githubusercontent.com/penpot/penpot/main/docker/images/docker-compose.yaml \
  -o docker-compose.yaml
```
atau

```bash
wget https://raw.githubusercontent.com/penpot/penpot/main/docker/images/docker-compose.yaml
```

Pastikan file berhasil diunduh:

```bash
ls -lh docker-compose.yaml
```

File tersebut mendefinisikan layanan, image container, variabel lingkungan, jaringan, port, dan volume penyimpanan yang diperlukan oleh Penpot.

### 4.5 Mengonfigurasi URL Publik

Penpot perlu mengetahui alamat publik yang digunakan pengguna untuk mengakses aplikasi.

Buka file konfigurasi:

```bash
nano docker-compose.yaml
```

Cari pengaturan `PENPOT_PUBLIC_URI`, yang dapat berada di bagian konfigurasi bersama, seperti `x-uri`, bergantung pada versi file Compose.

Sesuaikan nilainya dengan alamat deployment.

Jika menggunakan IP publik dan port 9001:

```yaml
PENPOT_PUBLIC_URI: http://<IP_SERVER>:9001
```

Jika menggunakan domain dan HTTPS:

```yaml
PENPOT_PUBLIC_URI: https://<DOMAIN_ANDA>
```

Ganti placeholder dengan alamat yang benar. Pastikan port publik sesuai dengan pemetaan port dan konfigurasi reverse proxy.

Untuk penggunaan melalui internet, HTTPS sangat disarankan. Penggunaan HTTP dapat membatasi fitur browser tertentu dan tidak sesuai untuk menangani sesi pengguna secara aman.

### 4.6 Memeriksa Konfigurasi Database

Docker Compose resmi Penpot sudah mendefinisikan layanan PostgreSQL dan pengaturan koneksinya. Dengan demikian, PostgreSQL tidak perlu dipasang secara terpisah pada sistem operasi VPS.

Sebelum menjalankan aplikasi, periksa konfigurasi berikut pada file Compose:

- Nama layanan PostgreSQL.
- Nama database.
- Username dan password database.
- URI koneksi yang digunakan backend.
- Volume persisten untuk penyimpanan data.

Nilai tersebut harus konsisten antara konfigurasi database dan backend. Jangan mengasumsikan bahwa nama layanan atau kredensial selalu sama pada setiap versi Compose.

Untuk penggunaan yang lebih aman, ganti kredensial bawaan dengan password yang kuat dan unik. Jangan menyimpan informasi rahasia dalam repository publik.

### 4.7 Menjalankan Penpot

Jalankan perintah berikut dari direktori yang berisi file Compose:

```bash
sudo docker compose -p penpot -f docker-compose.yaml up -d
```

Docker akan mengunduh image yang diperlukan, membuat container dan jaringan, serta menjalankan layanan di latar belakang.

Pada proses pertama, pengunduhan image dan inisialisasi database dapat membutuhkan waktu beberapa menit.

### 4.8 Mengatur Akses Jaringan

Pastikan firewall pada VPS, firewall sistem operasi, atau aturan jaringan dari penyedia VPS mengizinkan port yang diperlukan.

Untuk akses langsung menggunakan port 9001, pastikan:

1. File Compose memetakan port host 9001 ke port frontend container yang sesuai.
2. Firewall mengizinkan koneksi TCP pada port 9001.
3. URL publik Penpot menggunakan alamat dan port yang sama.

Jika menggunakan reverse proxy, arahkan domain ke server dan teruskan permintaan web ke frontend Penpot. Konfigurasikan sertifikat TLS untuk menyediakan akses HTTPS.

## 5. Verifikasi Instalasi

### 5.1 Memeriksa Status Container

Jalankan:

```bash
sudo docker compose -p penpot -f docker-compose.yaml ps
```

Periksa apakah layanan utama berjalan dengan benar. Status masing-masing container dapat berbeda sesuai fungsi dan proses inisialisasinya.

### 5.2 Memeriksa Log

Untuk melihat log semua layanan:

```bash
sudo docker compose -p penpot -f docker-compose.yaml logs --tail=100
```

Untuk memantau log backend:

```bash
sudo docker compose -p penpot -f docker-compose.yaml logs -f penpot-backend
```

Tekan `Ctrl+C` untuk berhenti mengikuti log.

### 5.3 Mengakses Aplikasi

Jika menggunakan port langsung:

```text
http://<IP_SERVER>:9001
```

Jika menggunakan domain dan HTTPS:

```text
https://<DOMAIN_ANDA>
```

Jika konfigurasi sudah benar, halaman Penpot akan muncul pada browser.

Lakukan pengujian dengan membuat file desain sederhana, menyimpannya, dan memastikan file tersebut dapat dibuka kembali.

## 6. Penyimpanan Data

Penpot menggunakan volume Docker untuk mempertahankan data di luar siklus hidup container.

Data persisten dapat mencakup:

- Data PostgreSQL.
- Aset desain yang diunggah.
- Data lain yang ditentukan dalam konfigurasi volume.

Untuk melihat daftar volume:

```bash
sudo docker volume ls
```

Menghentikan atau membuat ulang container tidak otomatis menghapus named volume. Namun, menghapus volume dapat menghilangkan data secara permanen.

Lakukan pencadangan database dan aset secara berkala jika data perlu dipertahankan. Pastikan prosedur pemulihan juga diuji.

## 7. Troubleshooting

| Permasalahan | Kemungkinan penyebab | Solusi |
|---|---|---|
| SSH tidak dapat terhubung | IP, username, private key, atau firewall tidak sesuai | Periksa informasi koneksi dan aturan akses port 22. |
| Halaman Penpot tidak dapat dibuka | Container frontend berhenti atau port diblokir | Periksa status container, pemetaan port, dan firewall. |
| Container terus melakukan restart | Kesalahan konfigurasi atau sumber daya tidak mencukupi | Periksa log layanan dan penggunaan CPU/RAM. |
| Backend tidak terhubung ke database | PostgreSQL belum siap atau pengaturan koneksi salah | Periksa log database dan backend serta variabel konfigurasi. |
| Data hilang setelah container dibuat ulang | Data tidak disimpan dalam volume persisten atau volume terhapus | Periksa konfigurasi volume dan backup. |
| Fitur browser tertentu tidak berfungsi | Aplikasi diakses menggunakan HTTP | Konfigurasikan domain dan HTTPS. |

Perintah diagnostik tambahan:

```bash
sudo docker ps -a
sudo docker compose -p penpot -f docker-compose.yaml logs --tail=100
df -h
free -h
```

## 8. Pemeliharaan

### 8.1 Menghentikan Layanan

```bash
cd ~/penpot
sudo docker compose -p penpot -f docker-compose.yaml stop
```

Perintah ini menghentikan container tanpa menghapus konfigurasi dan volume persisten.

### 8.2 Menjalankan Kembali Layanan

```bash
cd ~/penpot
sudo docker compose -p penpot -f docker-compose.yaml start
```

### 8.3 Memperbarui Image

Unduh image terbaru sesuai konfigurasi Compose:

```bash
cd ~/penpot
sudo docker compose -p penpot -f docker-compose.yaml pull
```

Setelah memeriksa kompatibilitas versi dan memastikan backup tersedia, terapkan pembaruan:

```bash
sudo docker compose -p penpot -f docker-compose.yaml up -d
```

Sebelum memperbarui deployment yang menyimpan data penting, periksa catatan rilis dan petunjuk migrasi resmi Penpot.

## 9. Penghapusan Instalasi

Jika Penpot tidak lagi diperlukan, layanan dapat dihentikan dan dihapus.

### 9.1 Menghentikan dan Menghapus Container

```bash
cd ~/penpot
sudo docker compose -p penpot -f docker-compose.yaml down
```

Perintah ini menghapus container dan jaringan yang dikelola Compose, tetapi biasanya mempertahankan named volume.

### 9.2 Menghapus Volume

**Peringatan:** Langkah ini dapat menghapus database, aset desain, dan data persisten lainnya secara permanen.

Pastikan data sudah dicadangkan jika masih diperlukan. Untuk menghapus volume yang dikelola Compose:

```bash
sudo docker compose -p penpot -f docker-compose.yaml down -v
```

### 9.3 Membersihkan Infrastruktur VPS

Jika menggunakan VPS dari penyedia cloud, hentikan atau hapus server melalui panel penyedia setelah memastikan data tidak lagi diperlukan.

Periksa juga disk, alamat IP, snapshot, backup, dan resource tambahan yang masih menimbulkan biaya. Jika menggunakan server pribadi, pastikan container dan resource terkait sudah dibersihkan sesuai kebutuhan. Jika ada hal lain yang perlu dipastikan, dapat dicek melalui website resmi penpot pada referensi di bawah.

## Referensi

1. Penpot — Panduan instalasi Docker: https://help.penpot.dev/technical-guide/getting-started/docker/
2. Penpot — Panduan konfigurasi: https://help.penpot.app/technical-guide/configuration/
3. Konfigurasi Docker Compose resmi Penpot: https://github.com/penpot/penpot/blob/main/docker/images/docker-compose.yaml
4. Docker Documentation: https://docs.docker.com/
