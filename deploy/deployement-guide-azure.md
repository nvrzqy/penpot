# Panduan Deployment Penpot Self-Hosted di Microsoft Azure

## 1. Gambaran Umum Proyek

Penpot merupakan platform desain dan prototipe berbasis web yang bersifat *open-source*. Aplikasi ini dapat digunakan untuk merancang antarmuka pengguna (UI), membuat prototipe interaktif, dan mengelola sistem desain.

Pada proyek ini, Penpot diimplementasikan secara *self-hosted* menggunakan Microsoft Azure sebagai infrastruktur cloud dan Docker Compose untuk menjalankan serta mengelola layanan aplikasi. Untuk panduan deployment umum, lihat [Deployment Guide](deployement-guide.md).

Dengan pendekatan ini, pengguna dapat mengakses Penpot melalui browser tanpa perlu memasang aplikasi secara lokal. Implementasi akhir menggunakan domain [penpot-kdjk3.linkpc.net](https://penpot-kdjk3.linkpc.net/) dengan Caddy sebagai reverse proxy dan HTTPS.

### Tujuan

- Membangun instance Penpot pada Microsoft Azure.
- Mengimplementasikan layanan Penpot menggunakan Docker Compose.
- Mengonfigurasi database dan layanan pendukung aplikasi.
- Membuat aplikasi dapat diakses melalui domain publik yang aman menggunakan HTTPS.
- Mengonfigurasi DNS dan Caddy sebagai reverse proxy.
- Melakukan pengujian dan mendokumentasikan prosedur pemeliharaan serta penghapusan layanan.

## 2. Arsitektur Sistem

Implementasi Penpot terdiri dari beberapa komponen utama berikut.

| Komponen | Fungsi |
|---|---|
| Microsoft Azure Virtual Machine | Menyediakan lingkungan server untuk menjalankan Penpot. |
| DNS | Mengarahkan `penpot-kdjk3.linkpc.net` ke IP publik Azure VM. |
| Caddy | Menerima request pada port 80/443, mengelola TLS, dan meneruskan request ke Penpot. |
| Docker Engine | Menjalankan setiap komponen aplikasi dalam container. |
| Docker Compose | Mengatur konfigurasi, jaringan, dependensi, dan siklus hidup container. |
| Penpot Frontend | Menyediakan antarmuka yang diakses pengguna melalui browser. |
| Penpot Backend | Memproses permintaan aplikasi dan logika bisnis. |
| PostgreSQL | Menyimpan data aplikasi dan informasi proyek desain. |
| Valkey | Mendukung mekanisme internal seperti cache dan koordinasi notifikasi. |
| Penpot Exporter | Mendukung proses ekspor desain. |
| Penpot MCP | Menyediakan integrasi Model Context Protocol apabila diaktifkan. |
| Penpot Admin Console | Menyediakan fungsi administratif apabila disertakan dalam konfigurasi. |

Pengguna mengakses domain melalui HTTPS. Caddy menerima request pada port `443`, kemudian meneruskannya ke frontend Penpot pada `127.0.0.1:9001`. Frontend berkomunikasi dengan backend, sedangkan backend terhubung dengan PostgreSQL dan Valkey melalui jaringan internal Docker. Dengan demikian, database tidak perlu diekspos secara langsung ke internet.

## 3. Prasyarat

### 3.1 Infrastruktur

Sebelum instalasi dilakukan, beberapa kebutuhan berikut harus tersedia:

- Akun Microsoft Azure dengan subscription aktif.
- Virtual Machine yang menggunakan sistem operasi Ubuntu Server.
- Alamat IP publik untuk mengakses aplikasi.
- Domain atau hostname publik yang diarahkan ke IP VM.
- Network Security Group (NSG) untuk mengatur akses jaringan.
- Kapasitas CPU, RAM, dan penyimpanan yang memadai.

### 3.2 Perangkat dan perangkat lunak

- Terminal dengan dukungan SSH, misalnya Windows Subsystem for Linux (WSL).
- SSH private key untuk mengakses VM.
- Docker Engine.
- Docker Compose v2.
- Koneksi internet untuk mengunduh paket dan image container.

### 3.3 Port jaringan

| Port | Protokol | Kegunaan |
|---|---|---|
| 22 | TCP | Akses administrasi melalui SSH. |
| 80 | TCP | HTTP, redirect ke HTTPS, dan proses validasi sertifikat. |
| 443 | TCP | Akses utama Penpot melalui HTTPS. |
| 9001 | TCP | Akses langsung untuk pengujian awal; tidak perlu dibuka publik setelah Caddy aktif. |

Akses SSH sebaiknya dibatasi pada alamat IP administrator. Port aplikasi perlu diizinkan melalui NSG sesuai kebutuhan deployment.

Pada implementasi proyek, port `80` dan `443` digunakan oleh Caddy. Port `9001` masih dapat dijangkau langsung pada saat dokumentasi diperbarui dan sebaiknya ditutup dari internet setelah akses HTTPS dipastikan stabil.

## 4. Tahapan Instalasi

### 4.1 Membuat Virtual Machine di Microsoft Azure

1. Buka [Microsoft Azure Portal](https://portal.azure.com/).
2. Pilih menu **Virtual Machines**, kemudian pilih **Create → Azure virtual machine**.
3. Tentukan subscription dan region yang tersedia.
4. Pilih Ubuntu Server sebagai sistem operasi.
5. Pilih ukuran VM sesuai kebutuhan sumber daya aplikasi.
6. Tentukan username administrator dan buat SSH key.
7. Unduh private key dan simpan di lokasi yang aman.
8. Konfigurasikan inbound rule untuk SSH pada port TCP 22.
9. Pilih konfigurasi disk yang sesuai, kemudian lakukan validasi dan buat VM.
10. Setelah proses deployment selesai, catat alamat IP publik VM.

**Catatan keamanan:** Private key digunakan untuk autentikasi ke server dan tidak boleh diunggah ke repository publik.

### 4.2 Mengakses Virtual Machine melalui SSH

Siapkan private key yang digunakan saat membuat Virtual Machine (VM). Simpan file tersebut di lokasi yang aman dan sesuaikan perintah berikut dengan nama file, username, dan alamat IP VM masing-masing.

Salin private key ke direktori SSH:

```bash
mkdir -p ~/.ssh
cp /path/to/<NAMA_FILE_KEY>.pem ~/.ssh/<NAMA_FILE_KEY>.pem
chmod 600 ~/.ssh/<NAMA_FILE_KEY>.pem
```

Ganti `<NAMA_FILE_KEY>` dengan nama file private key yang dimiliki. Ganti `/path/to/` dengan lokasi file yang sebenarnya.

Hubungkan ke VM menggunakan perintah:

```bash
ssh -i ~/.ssh/<NAMA_FILE_KEY>.pem <USERNAME>@<IP_PUBLIK_VM>
```

Keterangan:

- `<NAMA_FILE_KEY>`: nama file private key yang digunakan untuk autentikasi SSH.
- `<USERNAME>`: username administrator yang ditentukan saat membuat VM.
- `<IP_PUBLIK_VM>`: alamat IP publik VM yang digunakan untuk mengakses server.

Jika koneksi berhasil, terminal akan menampilkan shell Linux dari VM. Seluruh perintah instalasi berikutnya dijalankan dari terminal server tersebut.

**Catatan keamanan:** Jangan membagikan private key atau mengunggahnya ke repository publik.

### 4.3 Memperbarui Sistem dan Memasang Docker

Perbarui indeks paket Ubuntu:

```bash
sudo apt update
```

Pasang Docker Engine, Docker Compose v2, dan curl:

```bash
sudo apt install -y docker.io docker-compose-v2 curl
```

Aktifkan layanan Docker agar berjalan otomatis saat sistem dinyalakan:

```bash
sudo systemctl enable --now docker
```

Periksa instalasi Docker dan Docker Compose:

```bash
sudo docker --version
sudo docker compose version
```

Kedua perintah tersebut harus menampilkan informasi versi tanpa kesalahan.

### 4.4 Mengunduh Konfigurasi Docker Compose Penpot

Buat direktori khusus untuk instalasi Penpot:

```bash
mkdir -p ~/penpot
cd ~/penpot
```

Unduh file Docker Compose dari repository resmi Penpot:

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

File tersebut mendefinisikan layanan yang akan dijalankan, image container, variabel lingkungan, jaringan internal, pemetaan port, dan volume penyimpanan.

### 4.5 Mengonfigurasi URL Publik Awal

Pada tahap awal, URL publik dikonfigurasi menggunakan alamat IP VM agar Penpot dapat diverifikasi sebelum DNS dan Caddy diterapkan.

Buka file konfigurasi:

```bash
nano docker-compose.yaml
```

Cari konfigurasi `PENPOT_PUBLIC_URI`, yang umumnya berada pada bagian konfigurasi URI bersama, seperti `x-uri`. Sesuaikan nilainya dengan alamat publik VM:

```yaml
PENPOT_PUBLIC_URI: http://<PUBLIC_IP>:9001
```

Ganti `<PUBLIC_IP>` dengan alamat IP publik Azure VM.

Simpan perubahan setelah selesai.

Konfigurasi HTTP ini ditujukan untuk pengujian sementara. Untuk penggunaan yang lebih aman, gunakan domain dan HTTPS serta konfigurasi cookie yang sesuai dengan dokumentasi Penpot. Penggunaan HTTP pada alamat publik dapat membatasi fitur browser tertentu.

### 4.6 Memeriksa Konfigurasi Database

PostgreSQL telah didefinisikan sebagai layanan dalam Docker Compose resmi Penpot. Oleh karena itu, PostgreSQL tidak perlu dipasang secara terpisah pada sistem operasi VM.

Konfigurasi bawaan umumnya menggunakan pengaturan berikut, tetapi nilai aktual harus diperiksa pada file Compose yang digunakan.

| Parameter | Contoh nilai bawaan |
|---|---|
| Layanan database | `penpot-postgres` |
| Nama database | `penpot` |
| Username database | `penpot` |
| Password database | `penpot` |
| URI koneksi | `postgresql://penpot-postgres/penpot` |

Backend menggunakan konfigurasi koneksi tersebut untuk berkomunikasi dengan PostgreSQL melalui jaringan internal Docker.

Untuk deployment yang digunakan secara serius, ganti kredensial bawaan dengan nilai yang kuat dan unik. Pastikan konfigurasi PostgreSQL dan backend menggunakan kredensial yang konsisten.

**Penting:** Jangan memublikasikan password, token, atau informasi rahasia lainnya di repository GitHub.

### 4.7 Menjalankan Penpot

Jalankan perintah berikut dari direktori yang berisi `docker-compose.yaml`:

```bash
sudo docker compose -p penpot -f docker-compose.yaml up -d
```

Docker akan mengunduh image yang diperlukan, membuat container, dan menjalankan layanan di latar belakang.

Opsi `-d` digunakan agar container tetap berjalan tanpa mempertahankan terminal dalam mode interaktif.

Pada proses pertama, pengunduhan image dan inisialisasi layanan dapat membutuhkan waktu beberapa menit.

### 4.8 Mengatur Akses Jaringan Azure untuk Pengujian Awal

Agar Penpot dapat diakses dari luar VM:

1. Buka Virtual Machine pada Azure Portal.
2. Masuk ke pengaturan jaringan.
3. Buka Network Security Group yang terhubung dengan VM.
4. Tambahkan inbound security rule untuk TCP port 9001.
5. Batasi sumber akses sesuai kebutuhan.
6. Simpan aturan tersebut.

Pastikan pemetaan port pada file Compose juga sesuai. Port database dan layanan internal lainnya tidak perlu dibuka ke internet.

### 4.9 Mengakses Penpot melalui IP pada Tahap Awal

Buka browser dan masukkan alamat berikut:

```text
http://<PUBLIC_IP>:9001
```

Ganti `<PUBLIC_IP>` dengan alamat IP publik VM.

Jika konfigurasi sudah benar, halaman Penpot akan muncul dan pengguna dapat melanjutkan ke dashboard aplikasi.

Ambil screenshot halaman Penpot yang berhasil dibuka sebagai bukti keberhasilan deployment.

### 4.10 Mengarahkan DNS ke Azure VM

Setelah akses awal melalui IP berhasil, domain diarahkan ke alamat IP publik Azure VM.

| Parameter | Nilai implementasi |
|---|---|
| Domain | `penpot-kdjk3.linkpc.net` |
| Tipe DNS record | `A` |
| Alamat IP tujuan | `20.211.42.204` |

Periksa hasil resolusi DNS:

```bash
nslookup penpot-kdjk3.linkpc.net
```

atau:

```bash
dig +short penpot-kdjk3.linkpc.net
```

Pada 22 Oktober 2026, domain tersebut berhasil di-resolve ke `20.211.42.204`.

### 4.11 Membuka Port HTTP dan HTTPS pada Azure

Tambahkan inbound rule pada Network Security Group:

| Port | Protokol | Aksi | Fungsi |
|---:|---|---|---|
| `80` | TCP | Allow | Redirect HTTP ke HTTPS dan validasi sertifikat |
| `443` | TCP | Allow | Akses utama Penpot melalui HTTPS |

Port database dan layanan internal lainnya tetap tidak perlu dibuka. Port `9001` digunakan sebagai upstream Caddy dan sebaiknya tidak diekspos publik setelah HTTPS stabil.

### 4.12 Menjalankan Caddy

Implementasi proyek menjalankan Caddy sebagai container terpisah menggunakan host networking. Repository ini menyediakan [template Caddyfile](Caddyfile) yang sama dengan konfigurasi pada Azure VM. Jika repository proyek tersedia pada VM, jalankan perintah berikut dari root repository:

```bash
sudo mkdir -p /opt/caddy
sudo cp deploy/Caddyfile /opt/caddy/Caddyfile
```

Jika template tidak tersedia pada VM, buat file secara manual:

```bash
sudo mkdir -p /opt/caddy
sudo nano /opt/caddy/Caddyfile
```

Isi file:

```caddyfile
penpot-kdjk3.linkpc.net {
    reverse_proxy 127.0.0.1:9001
}
```

Jalankan container Caddy:

```bash
sudo docker run -d \
  --name caddy \
  --restart unless-stopped \
  --network host \
  -v /opt/caddy/Caddyfile:/etc/caddy/Caddyfile:ro \
  -v caddy_data:/data \
  -v caddy_config:/config \
  caddy:2
```

Caddy menggunakan volume `caddy_data` untuk mempertahankan sertifikat dan data TLS, serta `caddy_config` untuk konfigurasi internal. Dengan DNS dan port jaringan yang benar, Caddy mengelola penerbitan, redirect, dan pembaruan sertifikat HTTPS secara otomatis.

Periksa container dan log:

```bash
sudo docker ps --filter name=caddy
sudo docker logs --tail=100 caddy
```

### 4.13 Memperbarui Konfigurasi Penpot untuk HTTPS

Ubah nilai URI publik pada `docker-compose.yaml`:

```yaml
PENPOT_PUBLIC_URI: https://penpot-kdjk3.linkpc.net
```

Hapus `disable-secure-session-cookies` dari `PENPOT_FLAGS` karena aplikasi sekarang diakses melalui HTTPS. Konfigurasi hasil perubahan:

```yaml
PENPOT_FLAGS: disable-email-verification enable-smtp enable-prepl-server enable-mcp enable-admin-console
```

Validasi file Compose dan terapkan perubahan tanpa menghapus volume:

```bash
sudo docker compose -p penpot -f docker-compose.yaml config -q
sudo docker compose -p penpot -f docker-compose.yaml up -d
```

### 4.14 Mengakses Domain HTTPS

Buka alamat berikut:

```text
https://penpot-kdjk3.linkpc.net/
```

Pengguna tidak perlu menambahkan port `9001`. Request HTTPS diterima Caddy pada port `443`, lalu diteruskan ke Penpot pada `127.0.0.1:9001`.

### 4.15 Membatasi Port 9001

Hasil verifikasi eksternal pada 22 Oktober 2026 menunjukkan port `9001` masih dapat diakses langsung. Untuk menghilangkan jalur HTTP yang melewati Caddy, ubah mapping frontend menjadi loopback-only:

```yaml
ports:
  - "127.0.0.1:9001:8080"
```

Terapkan perubahan, pastikan domain HTTPS tetap berfungsi, lalu hapus inbound rule `9001` dari Azure NSG. Caddy tetap dapat mengakses upstream karena berjalan dengan host networking.

## 5. Verifikasi Instalasi

### 5.1 Memeriksa Status Container

Jalankan:

```bash
sudo docker compose -p penpot -f docker-compose.yaml ps
```

Periksa apakah layanan utama berada dalam status berjalan. Status setiap container dapat berbeda sesuai proses inisialisasinya.

### 5.2 Memeriksa Log Aplikasi

Untuk melihat log semua layanan:

```bash
sudo docker compose -p penpot -f docker-compose.yaml logs --tail=100
```

Untuk mengikuti log backend:

```bash
sudo docker compose -p penpot -f docker-compose.yaml logs -f penpot-backend
```

Tekan `Ctrl+C` untuk menghentikan pemantauan log.

### 5.3 Menguji Fungsi Aplikasi

Lakukan pengujian berikut:

1. Pastikan halaman Penpot dapat dibuka melalui alamat publik.
2. Pastikan antarmuka aplikasi dimuat tanpa error yang menetap.
3. Pastikan pengguna dapat masuk ke dashboard sesuai konfigurasi autentikasi.
4. Buat satu file desain uji.
5. Simpan file tersebut dan pastikan data dapat diakses kembali.

Pengujian ini membantu memastikan bahwa frontend, backend, dan penyimpanan data bekerja bersama.

### 5.4 Memverifikasi DNS, Caddy, dan HTTPS

Jalankan pemeriksaan berikut dari perangkat di luar VM:

```bash
nslookup penpot-kdjk3.linkpc.net
curl -I http://penpot-kdjk3.linkpc.net/
curl -I https://penpot-kdjk3.linkpc.net/
```

Hasil verifikasi pada 22 Oktober 2026:

| Pemeriksaan | Hasil |
|---|---|
| Resolusi DNS | `penpot-kdjk3.linkpc.net` → `20.211.42.204` |
| Akses HTTP | `308 Permanent Redirect` menuju HTTPS |
| Server redirect | `Caddy` |
| Akses HTTPS | `200 OK` |
| Jalur proxy | Header `Via: 1.1 Caddy` tersedia pada respons HTTPS |
| Akses langsung port 9001 | Masih menghasilkan `200 OK`; perlu ditutup sebagai hardening |

Hasil tersebut membuktikan bahwa DNS, redirect, reverse proxy, sertifikat TLS, dan upstream Penpot bekerja. Pengujian fitur aplikasi secara menyeluruh tetap dilakukan pada tahap terpisah.

## 6. Penyimpanan dan Persistensi Data

Docker Compose menggunakan volume untuk menyimpan data agar tidak hilang ketika container dihentikan atau dibuat ulang.

Data persisten umumnya mencakup:

- File database PostgreSQL.
- Aset desain yang diunggah.

Periksa volume yang tersedia menggunakan:

```bash
sudo docker volume ls
```

Nama volume yang sebenarnya mengikuti konfigurasi Compose yang digunakan.

Menghentikan atau membuat ulang container tidak otomatis menghapus named volume. Namun, penghapusan volume dapat menyebabkan hilangnya data secara permanen.

Untuk proyek yang perlu dipertahankan, buat cadangan database dan aset sebelum melakukan pembaruan atau menghapus infrastruktur.

## 7. Troubleshooting

| Permasalahan | Kemungkinan penyebab | Solusi |
|---|---|---|
| SSH gagal terhubung | Username, IP, izin key, atau aturan jaringan tidak sesuai | Periksa perintah SSH, izin private key, status VM, dan port 22. |
| Browser tidak dapat membuka Penpot | Caddy atau frontend tidak berjalan | Periksa container Caddy, status Penpot, dan log kedua layanan. |
| Container terus melakukan restart | Kesalahan konfigurasi atau sumber daya VM tidak memadai | Periksa log Docker dan penggunaan CPU/RAM. |
| Backend tidak dapat terhubung ke database | PostgreSQL belum siap atau konfigurasi koneksi tidak sesuai | Periksa log PostgreSQL dan backend serta variabel konfigurasi database. |
| Domain tidak mengarah ke VM | DNS record salah atau belum terpropagasi | Periksa record `A`, `nslookup`, dan IP publik VM. |
| Sertifikat HTTPS gagal diterbitkan | Port 80/443 tertutup atau DNS salah | Periksa NSG, DNS, dan log Caddy. |
| Domain menampilkan error proxy | Caddy tidak dapat menjangkau `127.0.0.1:9001` | Periksa `Caddyfile`, mapping frontend, dan status Penpot. |
| Login/session bermasalah setelah HTTPS | Public URI atau secure cookie belum sesuai | Gunakan URI HTTPS dan hapus `disable-secure-session-cookies`. |
| Data hilang setelah container dibuat ulang | Data tidak disimpan pada volume persisten atau volume telah dihapus | Periksa konfigurasi volume dan siapkan backup. |

Perintah diagnostik tambahan:

```bash
sudo docker ps -a
sudo docker compose -p penpot -f docker-compose.yaml logs --tail=100
sudo docker logs --tail=100 caddy
df -h
free -h
```

## 8. Pemeliharaan Aplikasi

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

Unduh image terbaru yang sesuai dengan konfigurasi:

```bash
cd ~/penpot
sudo docker compose -p penpot -f docker-compose.yaml pull
```

Setelah meninjau pembaruan dan memastikan backup tersedia, buat ulang layanan:

```bash
sudo docker compose -p penpot -f docker-compose.yaml up -d
```

Sebelum memperbarui instance yang menyimpan data penting, periksa catatan rilis dan petunjuk migrasi resmi Penpot.

### 8.4 Mengelola Caddy

```bash
sudo docker ps --filter name=caddy
sudo docker logs --tail=100 caddy
sudo docker restart caddy
```

Jika `Caddyfile` diubah, restart container agar konfigurasi dimuat ulang. Jangan menghapus volume `caddy_data` tanpa alasan karena volume tersebut menyimpan material sertifikat dan state TLS.

## 9. Menghapus Deployment

Setelah proyek selesai, deployment dapat dihapus untuk mengurangi biaya penggunaan Azure.

### 9.1 Menghentikan dan Menghapus Container

```bash
cd ~/penpot
sudo docker compose -p penpot -f docker-compose.yaml down
```

Perintah ini menghapus container dan jaringan yang dikelola Compose, tetapi umumnya mempertahankan named volume.

### 9.2 Menghapus Volume Data

**Peringatan:** Langkah ini akan menghapus volume yang dikelola oleh proyek Compose dan dapat menghilangkan database serta aset desain secara permanen.

Pastikan semua data yang masih diperlukan telah dicadangkan sebelum menjalankan:

```bash
sudo docker compose -p penpot -f docker-compose.yaml down -v
```

### 9.3 Menghapus Resource Azure

1. Buka resource group tempat deployment dibuat.
2. Periksa VM, disk, public IP, network interface, dan resource lain yang terkait.
3. Cadangkan data yang masih diperlukan.
4. Hapus VM dan resource yang tidak lagi digunakan.
5. Pastikan resource yang dapat menimbulkan biaya berulang telah dihapus atau dihentikan sesuai kebutuhan.

Menghapus container Docker saja tidak otomatis menghentikan seluruh biaya Azure. Disk terkelola dan resource jaringan tertentu dapat tetap menimbulkan biaya selama masih tersedia.

Jika deployment akan dihapus sepenuhnya, container Caddy dapat dihapus setelah aplikasi tidak lagi digunakan:

```bash
sudo docker rm -f caddy
```

Volume Caddy dapat dipertahankan apabila sertifikat atau konfigurasi masih diperlukan. Menghapusnya berarti state TLS lokal tidak dapat dipakai kembali.

## 10. Kesimpulan

Implementasi ini menunjukkan bahwa Penpot dapat dijalankan secara mandiri pada Microsoft Azure dengan menggunakan Docker Compose. Pendekatan berbasis container mempermudah pengelolaan frontend, backend, database, dan layanan pendukung dalam satu konfigurasi deployment.

Aplikasi yang telah berjalan dapat diakses melalui [https://penpot-kdjk3.linkpc.net/](https://penpot-kdjk3.linkpc.net/). DNS telah mengarah ke Azure VM, Caddy berhasil melakukan redirect HTTP ke HTTPS dan meneruskan request ke Penpot, serta respons HTTPS berhasil diverifikasi.

HTTPS sudah diterapkan. Pengamanan lanjutan yang masih perlu dilakukan adalah menutup akses publik langsung ke port `9001`, menggunakan kredensial yang kuat, melakukan backup berkala, dan membatasi akses administrasi.

## Referensi

1. Penpot — Panduan instalasi Docker: https://help.penpot.dev/technical-guide/getting-started/docker/
2. Penpot — Panduan konfigurasi: https://help.penpot.app/technical-guide/configuration/
3. Konfigurasi Docker Compose resmi Penpot: https://github.com/penpot/penpot/blob/main/docker/images/docker-compose.yaml
4. Microsoft Azure Portal: https://portal.azure.com/
5. Caddy Documentation: https://caddyserver.com/docs/
6. Catatan implementasi DNS dan Caddy: https://chatgpt.com/share/6ad9025f-a608-83ec-bcc4-994fe8d48b9a
7. Catatan troubleshooting deployment: https://chatgpt.com/share/6ad90284-65f8-83ec-913a-71fb1e251b8e

## Daftar Screenshot Dokumentasi
<img width="1303" height="348" alt="image" src="https://github.com/user-attachments/assets/8f9363a0-2de3-414c-b0ec-dcecd65c158a" />
<img width="1337" height="557" alt="image" src="https://github.com/user-attachments/assets/06bb1614-8938-423c-b746-eaf040aa8479" />
<img width="1393" height="620" alt="image" src="https://github.com/user-attachments/assets/7daa5062-affa-476d-8590-3478dc8c363f" />

**Gambar 1** Ringkasan Virtual Machine pada Azure Portal.


<img width="1088" height="860" alt="image" src="https://github.com/user-attachments/assets/2f9299e5-4651-4edf-9746-48578f41ced7" />

**Gambar 2.** Terminal yang berhasil terhubung ke VM melalui SSH.


<img width="1911" height="695" alt="image" src="https://github.com/user-attachments/assets/751cc074-3957-409f-ac19-a4aa41aa93ea" />
<img width="1891" height="321" alt="image" src="https://github.com/user-attachments/assets/7297db82-f679-4652-be95-f207562ad1e2" />

**Gambar 3.** Verifikasi versi Docker dan Docker Compose.


<img width="1917" height="482" alt="image" src="https://github.com/user-attachments/assets/fb169de1-ca0f-4146-b66d-998550b3048a" />

**Gambar 4.** Status container Penpot yang berjalan.


<img width="1917" height="1151" alt="image" src="https://github.com/user-attachments/assets/55b8359b-b188-4e01-adae-10c0ddb85922" />
<img width="1917" height="1146" alt="image" src="https://github.com/user-attachments/assets/82617e26-396e-4ed6-9c94-f9f23f39d402" />

**Gambar 5.** Halaman Penpot yang berhasil diakses melalui IP publik sebelum domain dan HTTPS dikonfigurasi.


<img width="1917" height="1147" alt="image" src="https://github.com/user-attachments/assets/ae2f1d4b-9035-444a-b014-a4f570512726" />

**Gambar 6.** File desain uji yang berhasil dibuat dan disimpan.


<img width="1441" height="140" alt="image" src="https://github.com/user-attachments/assets/75b95b50-e004-416c-acf7-249aa68f6492" />

**Gambar 7.** Konfigurasi public URI Penpot setelah domain dan HTTPS diterapkan.


<img width="642" height="667" alt="image" src="https://github.com/user-attachments/assets/5be0405f-e385-46cf-9b0f-dea7c129e364" />

**Gambar 8.** Record DNS yang mengarahkan domain Penpot ke IP publik Azure VM.


<img width="1415" height="280" alt="image" src="https://github.com/user-attachments/assets/2b63e9be-d33e-4266-8227-a049669288b5" />

**Gambar 9.** Aturan jaringan Azure yang mengizinkan akses HTTP dan HTTPS.


<img width="716" height="137" alt="image" src="https://github.com/user-attachments/assets/5ec60efc-cb0f-460c-b72f-01262f38a5f9" />

**Gambar 10.** Konfigurasi Caddy sebagai reverse proxy menuju frontend Penpot.


<img width="1307" height="112" alt="image" src="https://github.com/user-attachments/assets/afc6f169-a2de-4435-a9f8-ad6882a45b7d" />

**Gambar 11.** Container Caddy berjalan pada Azure VM.


<img width="1917" height="1127" alt="image" src="https://github.com/user-attachments/assets/7b21b9fb-8b28-4d00-9909-e1ebff0117fe" />
<img width="1917" height="1110" alt="image" src="https://github.com/user-attachments/assets/50e80247-2f8b-42fd-9d1e-69be0acb4217" />

**Gambar 12.** Log Caddy setelah reverse proxy dan HTTPS berhasil diaktifkan.


<img width="630" height="195" alt="image" src="https://github.com/user-attachments/assets/7b2351a4-1520-47eb-a2c2-048ac2e1550c" />

**Gambar 13.** Verifikasi resolusi domain menuju IP publik Azure VM.


<img width="727" height="187" alt="image" src="https://github.com/user-attachments/assets/5eb91a2b-d502-4ba4-ae8b-5e35b015e869" />

**Gambar 14.** Verifikasi redirect HTTP ke HTTPS dan respons aplikasi melalui Caddy.


<img width="1867" height="1155" alt="image" src="https://github.com/user-attachments/assets/1ef2ae3e-0ac4-4098-ada7-ff4110abbb28" />

**Gambar 15.** Penpot berhasil diakses melalui domain publik menggunakan HTTPS.
