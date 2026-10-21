# Laporan Proyek Self-Hosting Penpot

> Dokumen ini merupakan laporan berkala yang diperbarui mengikuti perkembangan proyek.

**Terakhir diperbarui:** 22 Oktober 2026
**Status saat ini:** Deployment, DNS, reverse proxy Caddy, dan HTTPS telah selesai. Dokumentasi penggunaan fitur utama, pengujian aplikasi, dan perbandingan Penpot dengan Figma menjadi tahap berikutnya.

## 1. Informasi Proyek

| Keterangan | Detail |
|---|---|
| Aplikasi | Penpot |
| Kategori | Collaborative Design / UI Design & Prototyping |
| Repository aplikasi | https://github.com/penpot/penpot |
| Metode implementasi | Self-hosting dengan Docker Compose |
| Infrastruktur final | Microsoft Azure Virtual Machine |
| Sistem operasi server | Ubuntu Server |
| Domain publik | [penpot-kdjk3.linkpc.net](https://penpot-kdjk3.linkpc.net/) |
| Akses aplikasi saat ini | HTTPS melalui Caddy pada port `443` |
| Alamat IP Azure VM | `20.211.42.204` |
| Aplikasi pembanding | Figma |

## 2. Ringkasan Proyek

Proyek ini bertujuan mempelajari proses instalasi, konfigurasi, dan deployment aplikasi web secara mandiri. Aplikasi yang dipilih adalah **Penpot**, sebuah platform desain UI/UX dan prototyping berbasis web yang bersifat open-source dan mendukung self-hosting.

Penpot dijalankan menggunakan Docker Compose agar frontend, backend, database, cache, dan layanan pendukung dapat dikelola dalam satu konfigurasi. Setelah tahap persiapan dan penyusunan panduan instalasi lokal, implementasi final dilakukan pada Virtual Machine di Microsoft Azure.

Pada tahap laporan ini, deployment telah berhasil dilakukan. Container Penpot berjalan pada Azure VM, DNS mengarahkan `penpot-kdjk3.linkpc.net` ke IP publik server, dan Caddy menyediakan reverse proxy serta HTTPS. Aplikasi dapat diakses melalui [https://penpot-kdjk3.linkpc.net/](https://penpot-kdjk3.linkpc.net/). Bukti implementasi berupa screenshot Azure Portal, koneksi SSH, versi Docker, status container, halaman Penpot, dan file desain awal telah didokumentasikan.

Pekerjaan selanjutnya difokuskan pada penggunaan fitur utama, pengujian aplikasi secara terstruktur, serta perbandingan Penpot dengan Figma.

## 3. Pemilihan Aplikasi

Penpot dipilih karena memenuhi kebutuhan proyek self-hosting aplikasi web dan memiliki fungsi yang relevan untuk desain UI/UX.

Alasan pemilihan Penpot:

1. Bersifat open-source dan dapat dijalankan pada infrastruktur sendiri.
2. Menyediakan fitur desain antarmuka, prototyping, komponen, dan kolaborasi.
3. Memiliki arsitektur berbasis layanan yang sesuai untuk mempelajari Docker.
4. Dapat diakses melalui browser sehingga cocok untuk deployment pada server.
5. Memiliki aplikasi pembanding yang relevan dan populer, yaitu Figma.

## 4. Ruang Lingkup Implementasi

Ruang lingkup yang telah dikerjakan meliputi:

- Analisis awal aplikasi dan kebutuhan deployment.
- Penyediaan konfigurasi Docker Compose.
- Penyusunan panduan instalasi Penpot pada komputer lokal.
- Penyusunan panduan deployment umum dan Microsoft Azure.
- Pembuatan Azure Virtual Machine berbasis Ubuntu Server.
- Instalasi Docker Engine dan Docker Compose pada VM.
- Konfigurasi URL publik dan port aplikasi.
- Menjalankan seluruh layanan Penpot menggunakan Docker Compose.
- Konfigurasi akses jaringan melalui Azure Network Security Group.
- Konfigurasi DNS menuju IP publik Azure VM.
- Implementasi Caddy sebagai reverse proxy.
- Penerapan HTTPS dan secure session cookie.
- Verifikasi redirect HTTP ke HTTPS dan respons aplikasi.
- Verifikasi status container dan akses Penpot melalui browser.
- Dokumentasi tahapan deployment dan bukti screenshot.

Ruang lingkup yang belum selesai:

- Dokumentasi penggunaan fitur utama Penpot.
- Pengujian fungsional aplikasi secara terstruktur.
- Analisis dan perbandingan Penpot dengan Figma.

### 4.1 Rencana Deployment Awal

Pada awal proyek, implementasi direncanakan melalui dua tahap. Penpot terlebih dahulu disiapkan pada lingkungan lokal untuk mempelajari konfigurasi Docker, kemudian dipindahkan ke server agar dapat diakses melalui internet.

```text
Development / Testing
        │
        ▼
┌─────────────────────────┐
│      Local Machine      │
│                         │
│       VM / WSL          │
│        Ubuntu           │
│        Docker           │
│        Penpot           │
└────────────┬────────────┘
             │
             │ Konfigurasi dan validasi awal
             ▼
┌─────────────────────────┐
│      Cloud / VPS        │
│                         │
│      Ubuntu Server      │
│         Docker          │
│         Penpot          │
└────────────┬────────────┘
             │
             │ Akses jaringan
             ▼
          Internet
             │
             ▼
         End Users
```

Rencana tersebut direalisasikan menggunakan Microsoft Azure sebagai penyedia infrastruktur cloud. Implementasi akhir menggunakan Azure Virtual Machine, Ubuntu Server, dan Docker Compose.

### 4.2 Rencana Instalasi Lokal

Lingkungan lokal digunakan sebagai acuan untuk menjalankan Penpot tanpa hosting dan tanpa IP publik.

```text
┌──────────────────────────────────┐
│          Komputer Lokal          │
│                                  │
│  ┌────────────────────────────┐  │
│  │   Docker / Docker Desktop  │  │
│  │                            │  │
│  │  ┌──────────────────────┐  │  │
│  │  │    Penpot Stack      │  │  │
│  │  │                      │  │  │
│  │  │ Frontend + Backend   │  │  │
│  │  │ PostgreSQL + Valkey  │  │  │
│  │  └──────────┬───────────┘  │  │
│  └─────────────┼──────────────┘  │
│                │                 │
│                ▼                 │
│      http://localhost:9001       │
└──────────────────────────────────┘
```

Panduan untuk skenario ini tersedia pada [Installation Guide Lokal](../installation/installation-guide.md).

### 4.3 Realisasi Deployment Akhir

```text
┌────────────────────────────┐
│          End User          │
│        Web Browser         │
└─────────────┬──────────────┘
              │ HTTPS :443
              ▼
┌────────────────────────────┐
│ penpot-kdjk3.linkpc.net    │
│ DNS A → 20.211.42.204      │
└─────────────┬──────────────┘
              │
              ▼
┌────────────────────────────────────┐
│          Microsoft Azure           │
│                                    │
│  Network Security Group            │
│  ├── TCP 22  (SSH)                 │
│  ├── TCP 80  (HTTP redirect)       │
│  └── TCP 443 (HTTPS)               │
│             │                      │
│             ▼                      │
│  ┌──────────────────────────────┐  │
│  │ Azure VM / Ubuntu Server     │  │
│  │                              │  │
│  │ Caddy :80/:443               │  │
│  │        │ reverse proxy       │  │
│  │        ▼                     │  │
│  │ Penpot Frontend :9001        │  │
│  │        │                     │  │
│  │        ├── Backend           │  │
│  │        ├── PostgreSQL        │  │
│  │        ├── Valkey            │  │
│  │        └── Supporting Service│  │
│  └──────────────────────────────┘  │
└────────────────────────────────────┘
```

### 4.4 Perbandingan Rencana dan Realisasi

| Tahap | Rencana awal | Realisasi |
|---|---|---|
| Persiapan | VM/WSL dan Docker pada komputer lokal | Konfigurasi lokal dan panduan instalasi telah disiapkan |
| Server | VPS atau cloud hosting | Microsoft Azure Virtual Machine |
| Sistem operasi | Ubuntu | Ubuntu Server |
| Orkestrasi | Docker Compose | Docker Compose project `penpot` |
| Akses | Domain atau IP publik | Domain `penpot-kdjk3.linkpc.net` |
| Reverse proxy | Direncanakan untuk akses produksi | Caddy container dengan host networking |
| HTTPS | Direncanakan untuk penggunaan produksi | Sudah aktif dan terverifikasi |
| Hasil | Penpot dapat diakses melalui internet | Berhasil diakses melalui HTTPS tanpa port tambahan |

## 5. Arsitektur Deployment

### 5.1 Topologi Infrastruktur

Deployment menggunakan satu Azure Virtual Machine sebagai host seluruh container. Pengguna mengakses aplikasi melalui domain HTTPS, sedangkan administrator mengelola server melalui SSH.

```mermaid
flowchart TB
    user[Pengguna<br/>Web Browser]
    admin[Administrator<br/>Terminal SSH]
    internet((Internet))
    dns[DNS<br/>penpot-kdjk3.linkpc.net]

    subgraph azure[Microsoft Azure]
        publicIP[Public IP<br/>20.211.42.204]
        nsg[Network Security Group<br/>TCP 22, 80, dan 443]

        subgraph vm[Azure Virtual Machine]
            ubuntu[Ubuntu Server]
            caddy[Caddy<br/>Reverse Proxy + TLS]
            penpot[Penpot Stack]
        end
    end

    user -->|HTTPS :443| internet
    admin -->|SSH :22| internet
    internet --> dns
    dns --> publicIP
    publicIP --> nsg
    nsg --> ubuntu
    ubuntu --> caddy
    caddy -->|127.0.0.1:9001| penpot
```

Alur akses eksternal:

1. Browser me-resolve `penpot-kdjk3.linkpc.net` melalui DNS ke `20.211.42.204`.
2. Browser membuat koneksi HTTPS ke port `443` Azure VM.
3. Network Security Group memeriksa inbound rule.
4. Caddy menangani TLS dan meneruskan request ke `127.0.0.1:9001`.
5. Docker meneruskan port host `9001` ke port `8080` milik container frontend.
6. Frontend berkomunikasi dengan layanan internal melalui network Docker bernama `penpot`.

### 5.2 Arsitektur Layanan Docker

```mermaid
flowchart LR
    browser[Browser]
    dns[DNS<br/>penpot-kdjk3.linkpc.net]

    subgraph host[Azure VM / Ubuntu Server]
        caddy[Caddy Container<br/>Host Network]

        subgraph network[Docker Network: penpot]
            frontend[Penpot Frontend<br/>2.18]
            backend[Penpot Backend<br/>2.18]
            admin[Admin Console<br/>2.18]
            mcp[Penpot MCP<br/>2.18]
            exporter[Penpot Exporter<br/>2.18]
            postgres[(PostgreSQL 15)]
            valkey[(Valkey 8.1)]
            mail[Mailcatcher]
        end

        assets[(penpot_assets)]
        dbdata[(penpot_postgres_v15)]
    end

    browser -->|HTTPS :443| dns
    dns --> caddy
    caddy -->|Reverse proxy 127.0.0.1:9001| frontend
    frontend --> backend
    frontend --> admin
    frontend --> mcp
    frontend --> exporter
    backend --> postgres
    backend --> valkey
    backend --> mail
    admin --> postgres
    exporter --> valkey
    frontend --- assets
    backend --- assets
    postgres --- dbdata
```

Service Penpot berada pada jaringan internal yang sama. Caddy berjalan sebagai container terpisah dengan host networking agar dapat menerima traffic pada port `80`/`443` dan menjangkau Penpot pada loopback port `9001`. PostgreSQL dan Valkey tidak dipublikasikan ke internet. Dependensi kesehatan pada Compose memastikan backend menunggu PostgreSQL dan Valkey siap sebelum mulai bekerja.

### 5.3 Fungsi Setiap Komponen

| Komponen | Image/versi pada Compose | Fungsi | Akses |
|---|---|---|---|
| Caddy | `caddy:2` | Reverse proxy, redirect HTTP, dan pengelolaan TLS otomatis | Host `80` dan `443` |
| Penpot Frontend | `penpotapp/frontend:2.18` | Menyajikan antarmuka web dan menjadi pintu masuk pengguna | Host `9001` → container `8080` |
| Penpot Backend | `penpotapp/backend:2.18` | Menangani autentikasi, logika aplikasi, file, dan komunikasi data | Internal Docker |
| Penpot Admin Console | `penpotapp/admin-console:2.18` | Mendukung fungsi administrasi Penpot | Internal Docker |
| Penpot MCP | `penpotapp/mcp:2.18` | Menyediakan layanan integrasi Model Context Protocol | Internal Docker |
| Penpot Exporter | `penpotapp/exporter:2.18` | Memproses kebutuhan ekspor desain | Internal Docker |
| PostgreSQL | `postgres:15` | Menyimpan akun, project, file, dan metadata aplikasi | Internal Docker |
| Valkey | `valkey/valkey:8.1` | Cache serta koordinasi notifikasi dan WebSocket | Internal Docker |
| Mailcatcher | `sj26/mailcatcher:latest` | SMTP sementara untuk menangkap email pada tahap pengujian | SMTP internal `1025`; UI host `1080` |

Versi Penpot `2.18` merupakan nilai default pada file Compose dan dapat dioverride melalui variabel `PENPOT_VERSION`.

### 5.4 Alur Request dan Data

```mermaid
sequenceDiagram
    actor U as Pengguna
    participant D as DNS
    participant C as Caddy
    participant F as Penpot Frontend
    participant B as Penpot Backend
    participant V as Valkey
    participant P as PostgreSQL
    participant A as Volume Assets

    U->>D: Resolve penpot-kdjk3.linkpc.net
    D-->>U: 20.211.42.204
    U->>C: HTTPS request :443
    C->>F: Reverse proxy ke 127.0.0.1:9001
    F-->>C: Antarmuka web
    C-->>U: HTTPS response
    U->>C: Login / membuka file desain
    C->>F: Meneruskan request aplikasi
    F->>B: Request API
    B->>V: Membaca/menulis cache dan notifikasi
    B->>P: Membaca/menulis data aplikasi
    B->>A: Membaca/menulis aset desain
    P-->>B: Hasil query
    A-->>B: Data aset
    B-->>F: Response aplikasi
    F-->>C: Response aplikasi
    C-->>U: Menampilkan hasil melalui HTTPS
```

Diagram ini menunjukkan bahwa browser tidak berkomunikasi langsung dengan frontend atau database. Traffic publik masuk melalui Caddy, sedangkan semua operasi data diproses melalui backend.

### 5.5 Arsitektur Persistensi

```mermaid
flowchart LR
    frontend[Penpot Frontend] --> assets[(Volume penpot_assets)]
    backend[Penpot Backend] --> assets
    postgres[PostgreSQL 15] --> database[(Volume penpot_postgres_v15)]

    assets --> assetData[File dan aset desain]
    database --> appData[Akun, project, file, dan metadata]
```

Named volume memisahkan data persisten dari lifecycle container. Container dapat dihentikan atau dibuat ulang tanpa otomatis menghapus database dan aset. Perintah `down -v` tetap harus dihindari jika data masih diperlukan karena opsi tersebut menghapus volume.

### 5.6 Batas Jaringan

| Jalur | Port | Status implementasi | Keterangan |
|---|---:|---|---|
| Administrator → Azure VM | `22/TCP` | Dibuka melalui NSG | Digunakan untuk SSH |
| Pengguna → Caddy | `80/TCP` | Aktif | Redirect HTTP ke HTTPS dan validasi sertifikat |
| Pengguna → Caddy | `443/TCP` | Aktif | Akses utama Penpot melalui HTTPS |
| Pengguna → Penpot Frontend | `9001/TCP` | Tidak dibuka publik | Frontend dibatasi ke loopback dan hanya menjadi upstream Caddy |
| Pengguna → PostgreSQL | `5432/TCP` | Tidak dibuka | Hanya digunakan pada network Docker |
| Pengguna → Valkey | `6379/TCP` | Tidak dibuka | Hanya digunakan pada network Docker |
| Host → Mailcatcher UI | `1080/TCP` | Dipetakan oleh Compose, tidak diperlukan untuk pengguna | NSG sebaiknya tetap menutup port ini |

Untuk instalasi lokal, `PENPOT_PUBLIC_URI` tetap menggunakan `http://localhost:9001`. Salinan Compose pada Azure menggunakan `https://penpot-kdjk3.linkpc.net`.

## 6. Implementasi

### 6.1 Persiapan Konfigurasi

File [`docker-compose.yaml`](../deploy/docker-compose.yaml) digunakan untuk mendefinisikan seluruh layanan Penpot. Konfigurasi ini mencakup:

- Image container yang digunakan setiap layanan.
- Jaringan internal Docker.
- Port frontend `9001:8080`.
- Konfigurasi koneksi PostgreSQL dan Valkey.
- Volume persisten untuk database dan aset.
- Variabel lingkungan dan fitur Penpot.

Panduan instalasi lokal juga telah dibuat sebagai acuan menjalankan Penpot tanpa hosting. Panduan tersebut menggunakan alamat `http://localhost:9001` dan tidak memerlukan domain, IP publik, maupun konfigurasi cloud.

Parameter penting pada Compose:

| Parameter | Nilai implementasi | Fungsi |
|---|---|---|
| Project Compose | `penpot` | Memberikan prefix konsisten pada resource Docker |
| Versi default Penpot | `2.18` | Menentukan tag image frontend, backend, admin console, MCP, dan exporter |
| Public URI lokal | `http://localhost:9001` | URL akses ketika dijalankan pada komputer lokal |
| Public URI Azure | `https://penpot-kdjk3.linkpc.net` | URL publik final implementasi server |
| Port frontend | `9001:8080` | Memetakan port VM ke frontend Penpot |
| Penyimpanan aset | `penpot_assets` | Menyimpan file dan aset desain |
| Penyimpanan database | `penpot_postgres_v15` | Menyimpan data PostgreSQL |
| Network internal | `penpot` | Menghubungkan seluruh container |
| Restart policy | `always` | Menjalankan kembali container setelah berhenti/reboot |

### 6.2 Penyediaan Microsoft Azure Virtual Machine

Deployment final menggunakan Virtual Machine pada Microsoft Azure dengan Ubuntu Server. Tahapan yang dilakukan meliputi:

1. Membuat Azure Virtual Machine.
2. Menentukan ukuran VM dan penyimpanan.
3. Membuat akun administrator dan SSH key.
4. Mengaktifkan akses administrasi melalui port `22`.
5. Menyimpan alamat IP publik VM.
6. Menghubungkan terminal lokal ke VM menggunakan SSH.

Private key disimpan secara lokal dan tidak dimasukkan ke repository.

Resource Azure yang terlibat dalam implementasi:

| Resource | Peran |
|---|---|
| Virtual Machine | Menjalankan Ubuntu, Docker, dan Penpot |
| OS disk | Menyimpan sistem operasi serta data Docker host |
| Network interface | Menghubungkan VM dengan virtual network |
| Public IP address | Menyediakan alamat akses dari internet |
| Virtual network/subnet | Menjadi jaringan VM pada Azure |
| Network Security Group | Mengatur traffic masuk ke port SSH dan aplikasi |

Koneksi administrator ke server dilakukan menggunakan private key:

```bash
mkdir -p ~/.ssh
cp /path/to/<NAMA_FILE_KEY>.pem ~/.ssh/<NAMA_FILE_KEY>.pem
chmod 600 ~/.ssh/<NAMA_FILE_KEY>.pem
ssh -i ~/.ssh/<NAMA_FILE_KEY>.pem <USERNAME>@<IP_PUBLIK_VM>
```

Izin `600` membatasi pembacaan private key hanya kepada pemilik file.

### 6.3 Instalasi Docker

Pada Azure VM dilakukan pembaruan indeks paket, kemudian dipasang Docker Engine, Docker Compose v2, dan utilitas yang diperlukan. Layanan Docker diaktifkan agar berjalan otomatis bersama sistem.

Perintah implementasi:

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-v2 curl
sudo systemctl enable --now docker
```

Instalasi diverifikasi menggunakan informasi versi Docker dan Docker Compose. Bukti verifikasi telah disertakan pada dokumentasi deployment Azure.

```bash
sudo docker --version
sudo docker compose version
sudo systemctl status docker
```

Output versi memastikan binary tersedia, sedangkan status service memastikan Docker daemon aktif.

### 6.4 Konfigurasi Penpot

Konfigurasi Docker Compose Penpot ditempatkan pada server. Setelah domain dan HTTPS diterapkan, nilai `PENPOT_PUBLIC_URI` disesuaikan menjadi:

```yaml
PENPOT_PUBLIC_URI: https://penpot-kdjk3.linkpc.net
```

Flag `disable-secure-session-cookies` dihapus karena session sekarang dilayani melalui HTTPS:

```yaml
PENPOT_FLAGS: disable-email-verification enable-smtp enable-prepl-server enable-mcp enable-admin-console
```

Konfigurasi PostgreSQL dan backend diperiksa agar menggunakan nama layanan, database, username, password, dan URI koneksi yang konsisten pada jaringan internal Docker.

Konfigurasi penting yang diperiksa:

| Konfigurasi | Nilai contoh pada Compose | Catatan |
|---|---|---|
| `PENPOT_PUBLIC_URI` | `https://penpot-kdjk3.linkpc.net` | Sesuai alamat publik yang digunakan pengguna |
| `PENPOT_DATABASE_URI` | `postgresql://penpot-postgres/penpot` | Menggunakan nama service pada network Docker |
| `PENPOT_DATABASE_USERNAME` | `penpot` | Harus konsisten dengan PostgreSQL |
| `PENPOT_DATABASE_PASSWORD` | Tidak ditampilkan pada laporan | Harus konsisten dan tidak dipublikasikan |
| `PENPOT_REDIS_URI` | `redis://penpot-valkey/0` | Menghubungkan backend/exporter ke Valkey |
| `PENPOT_OBJECTS_STORAGE_BACKEND` | `fs` | Menggunakan penyimpanan file pada volume |
| `PENPOT_OBJECTS_STORAGE_FS_DIRECTORY` | `/opt/data/assets` | Lokasi aset di dalam container |

Sebelum menjalankan deployment, konfigurasi dapat diperiksa secara statis dengan:

```bash
sudo docker compose -p penpot -f docker-compose.yaml config
```

Perintah tersebut membantu menemukan kesalahan struktur YAML atau hasil interpolasi variabel tanpa langsung menjalankan container.

### 6.5 Menjalankan Layanan

Penpot dijalankan sebagai project Compose bernama `penpot`:

```bash
sudo docker compose -p penpot -f docker-compose.yaml up -d
```

Docker mengunduh image yang diperlukan, membuat network dan volume, lalu menjalankan layanan di latar belakang. Status layanan kemudian diperiksa menggunakan:

```bash
sudo docker compose -p penpot -f docker-compose.yaml ps
```

Hasil dokumentasi menunjukkan container utama berhasil berjalan.

Status dan log diperiksa menggunakan:

```bash
sudo docker compose -p penpot -f docker-compose.yaml ps
sudo docker compose -p penpot -f docker-compose.yaml logs --tail=100
sudo docker compose -p penpot -f docker-compose.yaml logs -f penpot-backend
```

PostgreSQL menggunakan health check `pg_isready`, sedangkan Valkey menggunakan `valkey-cli ping`. Backend baru dijalankan setelah kedua layanan tersebut berstatus sehat sesuai dependensi pada Compose.

### 6.6 Konfigurasi Jaringan Azure

Network Security Group pada Azure dikonfigurasi untuk mengizinkan:

| Port | Protokol | Fungsi |
|---|---|---|
| `22` | TCP | Administrasi server melalui SSH |
| `80` | TCP | Redirect HTTP dan validasi sertifikat Caddy |
| `443` | TCP | Akses Penpot melalui HTTPS |
| `9001` | TCP | Upstream lokal Penpot untuk Caddy; tidak dibuka ke internet |

Database dan layanan internal tidak diekspos langsung ke internet. Akses utama aplikasi menggunakan domain HTTPS melalui Caddy. Akses eksternal langsung ke port `9001` telah ditutup setelah tahap pengujian awal selesai.

Urutan konfigurasi jaringan:

1. Membuka halaman VM pada Azure Portal.
2. Memilih pengaturan networking atau Network Security Group.
3. Memastikan inbound rule SSH `22/TCP` tersedia untuk administrasi.
4. Menambahkan inbound rule `80/TCP` dan `443/TCP` untuk Caddy.
5. Mempertahankan `9001/TCP` selama pengujian awal.
6. Membatasi source IP untuk SSH jika akses publik penuh tidak diperlukan.
7. Tidak membuka port PostgreSQL, Valkey, atau layanan internal lain.
8. Menguji akses HTTPS dari browser menggunakan jaringan di luar VM.

### 6.7 Implementasi DNS, Caddy, dan HTTPS

#### 6.7.1 Konfigurasi DNS

DNS record tipe `A` dibuat agar hostname publik mengarah ke Azure VM:

| Hostname | Tipe | Nilai |
|---|---|---|
| `penpot-kdjk3.linkpc.net` | `A` | `20.211.42.204` |

Resolusi diverifikasi dari luar server menggunakan:

```bash
nslookup penpot-kdjk3.linkpc.net
```

#### 6.7.2 Konfigurasi Caddy

Caddy dijalankan sebagai container terpisah menggunakan host networking. Konfigurasi yang digunakan juga disediakan sebagai [template Caddyfile](../deploy/Caddyfile) dalam repository agar dapat digunakan kembali. File tersebut disalin ke `/opt/caddy/Caddyfile` pada server dan berisi:

```caddyfile
penpot-kdjk3.linkpc.net {
    reverse_proxy 127.0.0.1:9001
}
```

Container dibuat dengan:

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

`--network host` memungkinkan Caddy menerima traffic langsung pada port `80`/`443` dan meneruskan request ke Penpot melalui `127.0.0.1:9001`. Volume `caddy_data` mempertahankan sertifikat serta state TLS, sedangkan `caddy_config` menyimpan konfigurasi internal Caddy.

#### 6.7.3 Penerapan Konfigurasi HTTPS pada Penpot

Setelah `PENPOT_PUBLIC_URI` dan `PENPOT_FLAGS` diperbarui, konfigurasi divalidasi dan container Penpot dibuat ulang tanpa menghapus volume:

```bash
sudo docker compose -p penpot -f docker-compose.yaml config -q
sudo docker compose -p penpot -f docker-compose.yaml up -d
```

#### 6.7.4 Verifikasi Eksternal

Verifikasi pada 22 Oktober 2026 menghasilkan:

```text
DNS    : penpot-kdjk3.linkpc.net → 20.211.42.204
HTTP   : 308 Permanent Redirect → HTTPS
HTTPS  : 200 OK
Proxy  : Server/redirect Caddy dan header Via: 1.1 Caddy
```

Perintah pemeriksaan:

```bash
curl -I http://penpot-kdjk3.linkpc.net/
curl -I https://penpot-kdjk3.linkpc.net/
```

Hasil tersebut membuktikan DNS, redirect, penerbitan sertifikat TLS, reverse proxy, dan upstream Penpot bekerja. Setelah frontend dibatasi ke loopback dan inbound rule `9001` dihapus, verifikasi eksternal pada 22 Oktober 2026 menunjukkan koneksi langsung ke port tersebut mengalami timeout. Pada pemeriksaan yang sama, domain HTTPS tetap menghasilkan respons `200 OK`.

### 6.8 Persistensi Data

Docker named volume digunakan untuk menyimpan:

- Database PostgreSQL.
- Aset desain yang diunggah ke Penpot.

Dengan mekanisme ini, data tidak otomatis hilang ketika container dihentikan atau dibuat ulang. Penghapusan volume tetap harus dilakukan secara hati-hati karena dapat menghapus data secara permanen.

| Volume | Dipasang pada | Data yang disimpan |
|---|---|---|
| `penpot_assets` | Frontend dan backend | Aset, gambar, dan file yang dikelola Penpot |
| `penpot_postgres_v15` | PostgreSQL | Database akun, project, file, dan metadata |

Volume dapat diperiksa menggunakan:

```bash
sudo docker volume ls
sudo docker compose -p penpot -f docker-compose.yaml config --volumes
```

Backup perlu mencakup kedua volume. Menyalin direktori volume secara langsung ketika database aktif tidak disarankan; backup database dan volume harus dilakukan dengan prosedur yang konsisten.

### 6.9 Urutan Deployment

```mermaid
flowchart TD
    A[Mulai] --> B[Buat Azure VM Ubuntu]
    B --> C[Konfigurasi SSH dan NSG port 22]
    C --> D[Hubungkan ke VM melalui SSH]
    D --> E[Update package Ubuntu]
    E --> F[Instal Docker dan Compose]
    F --> G[Siapkan docker-compose.yaml]
    G --> H[Atur konfigurasi awal Penpot]
    H --> I[Periksa konfigurasi Compose]
    I --> J[Jalankan docker compose up -d]
    J --> K[Periksa health dan log container]
    K --> L[Verifikasi akses awal port 9001]
    L --> M[Buat DNS A record]
    M --> N[Buka NSG port 80 dan 443]
    N --> O[Konfigurasi dan jalankan Caddy]
    O --> P[Ubah Public URI menjadi HTTPS]
    P --> Q[Aktifkan secure session cookie]
    Q --> R[Terapkan ulang Compose]
    R --> S[Verifikasi DNS redirect dan HTTPS]
    S --> T[Dokumentasikan hasil]
    T --> U[Deployment selesai]
```

Diagram tersebut memperlihatkan deployment dua fase: verifikasi awal melalui port `9001`, kemudian penerapan domain dan HTTPS melalui Caddy. Dokumentasi dilakukan setelah DNS, redirect, reverse proxy, serta respons HTTPS berhasil diverifikasi.

### 6.10 Operasional dan Pemeliharaan

Perintah yang digunakan untuk mengelola lifecycle aplikasi:

| Aktivitas | Perintah |
|---|---|
| Melihat status | `sudo docker compose -p penpot -f docker-compose.yaml ps` |
| Melihat log | `sudo docker compose -p penpot -f docker-compose.yaml logs --tail=100` |
| Menghentikan sementara | `sudo docker compose -p penpot -f docker-compose.yaml stop` |
| Menjalankan kembali | `sudo docker compose -p penpot -f docker-compose.yaml start` |
| Mengunduh image | `sudo docker compose -p penpot -f docker-compose.yaml pull` |
| Menerapkan konfigurasi/image | `sudo docker compose -p penpot -f docker-compose.yaml up -d` |
| Menghapus container/network | `sudo docker compose -p penpot -f docker-compose.yaml down` |
| Melihat status Caddy | `sudo docker ps --filter name=caddy` |
| Melihat log Caddy | `sudo docker logs --tail=100 caddy` |
| Memuat ulang Caddy | `sudo docker restart caddy` |

Perintah `down` tidak menghapus named volume secara default. Perintah `down -v` tidak dicantumkan sebagai operasi rutin karena dapat menghapus seluruh data persisten.

Prosedur pembaruan yang aman:

1. Periksa versi dan catatan rilis Penpot.
2. Buat backup database dan aset.
3. Unduh image yang dibutuhkan.
4. Jalankan ulang stack menggunakan `up -d`.
5. Periksa status, health check, dan log.
6. Verifikasi akses melalui browser dan integritas data.

### 6.11 Keamanan dan Batasan Implementasi

Implementasi saat ini sudah menggunakan domain dan HTTPS melalui Caddy. Jalur publik lama pada port `9001` telah ditutup sehingga pengguna hanya mengakses Penpot melalui Caddy pada port `80` dan `443`.

| Temuan | Kondisi saat ini | Rekomendasi lanjutan |
|---|---|---|
| Transport web utama | HTTPS melalui Caddy | Pertahankan pembaruan otomatis sertifikat dan pantau log Caddy |
| Port frontend langsung | Port `9001` dibatasi ke loopback dan tidak dapat diakses dari internet | Pertahankan binding lokal dan jangan membuka kembali rule NSG `9001` |
| Cookie | `disable-secure-session-cookies` sudah dihapus | Pertahankan konfigurasi secure cookie selama HTTPS digunakan |
| Secret key | Compose menyediakan placeholder | Ganti dengan secret acak yang kuat |
| Kredensial database | Nilai Compose perlu diamankan | Gunakan password unik dan jangan commit secret |
| SSH | Menggunakan private key | Batasi source IP port `22` dan nonaktifkan login yang tidak diperlukan |
| Mailcatcher | UI dipetakan ke host `1080` | Jangan membuka port `1080` melalui NSG; hapus mapping jika tidak dibutuhkan |
| Backup | Belum dijelaskan sebagai otomatis | Buat jadwal backup database dan aset |
| Monitoring | Menggunakan status dan log Docker | Tambahkan monitoring resource dan alert untuk penggunaan jangka panjang |

### 6.12 Troubleshooting Deployment

| Permasalahan | Pemeriksaan | Tindakan |
|---|---|---|
| SSH tidak dapat terhubung | Status VM, IP, username, key, dan NSG port `22` | Perbaiki parameter SSH atau inbound rule |
| Docker daemon tidak aktif | `sudo systemctl status docker` | Jalankan `sudo systemctl start docker` |
| Konfigurasi Compose invalid | `docker compose ... config` | Perbaiki YAML atau variabel environment |
| Container berhenti/restart | `docker compose ... ps` dan `logs` | Identifikasi error konfigurasi atau resource |
| Backend gagal terhubung database | Health PostgreSQL dan kredensial | Samakan konfigurasi database dan backend |
| Domain tidak dapat di-resolve | Record DNS salah atau belum terpropagasi | Periksa record `A` dan hasil `nslookup` |
| HTTP tidak redirect ke HTTPS | Caddy berhenti atau port `80` tertutup | Periksa container Caddy, NSG, dan log |
| HTTPS gagal atau sertifikat tidak valid | DNS/port 80/443 bermasalah | Periksa DNS, NSG, waktu server, dan log Caddy |
| Domain menampilkan error proxy | Caddy tidak dapat menjangkau port `9001` | Periksa `Caddyfile`, mapping frontend, dan status Penpot |
| Login/session gagal | Public URI atau secure cookie tidak sesuai | Pastikan URI memakai HTTPS dan flag insecure sudah dihapus |
| Storage server penuh | `df -h` dan penggunaan volume Docker | Bersihkan data yang aman atau tambah kapasitas disk |

Perintah diagnostik utama:

```bash
sudo docker ps -a
sudo docker compose -p penpot -f docker-compose.yaml ps
sudo docker compose -p penpot -f docker-compose.yaml logs --tail=100
sudo docker logs --tail=100 caddy
sudo docker volume ls
df -h
free -h
```

## 7. Hasil Deployment dan Verifikasi

| Verifikasi | Hasil | Keterangan |
|---|---|---|
| Azure VM berhasil dibuat | ✅ Berhasil | Informasi VM tersedia pada Azure Portal |
| Koneksi SSH ke VM | ✅ Berhasil | Terminal berhasil masuk ke Ubuntu Server |
| Docker Engine terpasang | ✅ Berhasil | Versi Docker berhasil ditampilkan |
| Docker Compose tersedia | ✅ Berhasil | Versi Docker Compose berhasil ditampilkan |
| Container Penpot berjalan | ✅ Berhasil | Status layanan diperiksa melalui Docker Compose |
| Akses awal melalui IP publik | ✅ Berhasil | Digunakan untuk verifikasi sebelum domain dikonfigurasi |
| Penyimpanan file desain awal | ✅ Berhasil | File desain awal berhasil dibuat dan ditampilkan kembali |
| DNS domain | ✅ Berhasil | Domain mengarah ke `20.211.42.204` |
| Caddy reverse proxy | ✅ Berhasil | Request diteruskan ke `127.0.0.1:9001` |
| Redirect HTTP ke HTTPS | ✅ Berhasil | Respons `308 Permanent Redirect` |
| Akses domain HTTPS | ✅ Berhasil | Respons `200 OK` pada `https://penpot-kdjk3.linkpc.net/` |
| Secure session cookie | ✅ Diterapkan | Flag `disable-secure-session-cookies` telah dihapus |
| Pembatasan port 9001 | ✅ Berhasil | Koneksi eksternal mengalami timeout dan domain HTTPS tetap aktif |
| Pengujian fitur terstruktur | ⏳ Belum dilakukan | Akan dilakukan pada tahap pengujian aplikasi |

Verifikasi di atas memastikan bahwa infrastruktur, container, DNS, reverse proxy, TLS, jaringan, dan akses dasar aplikasi telah berfungsi. Pembuatan file desain awal hanya digunakan sebagai pemeriksaan awal bahwa aplikasi dapat digunakan dan data dapat disimpan; kegiatan tersebut belum menggantikan pengujian fungsional yang terstruktur.

## 8. Dokumentasi yang Telah Dibuat

| Dokumen | Isi | Status |
|---|---|---|
| [Installation Guide Lokal](../installation/installation-guide.md) | Instalasi Penpot pada komputer lokal menggunakan Docker | ✅ Selesai |
| [Deployment Guide Umum](../deploy/deployement-guide.md) | Deployment Penpot pada VPS/server Linux | ✅ Selesai |
| [Deployment Guide Azure](../deploy/deployement-guide-azure.md) | Implementasi pada Azure VM, verifikasi, pemeliharaan, dan screenshot | ✅ Selesai |
| [Docker Compose](../deploy/docker-compose.yaml) | Definisi layanan dan konfigurasi container Penpot | ✅ Tersedia |
| [Template Caddyfile](../deploy/Caddyfile) | Konfigurasi domain dan reverse proxy Caddy menuju Penpot | ✅ Tersedia |
| Panduan penggunaan fitur utama | Cara menggunakan fitur-fitur utama Penpot | ⏳ Belum dibuat |
| Dokumen pengujian aplikasi | Skenario, hasil, dan bukti pengujian | ⏳ Belum dibuat |
| Perbandingan Penpot dan Figma | Perbandingan fitur dan pengalaman penggunaan | ⏳ Belum diselesaikan |

Bukti visual implementasi tersedia pada bagian [Daftar Screenshot Dokumentasi](../deploy/deployement-guide-azure.md#daftar-screenshot-dokumentasi) dalam panduan deployment Azure, meliputi:

1. Ringkasan Virtual Machine pada Azure Portal.
2. Terminal yang berhasil terhubung melalui SSH.
3. Verifikasi versi Docker dan Docker Compose.
4. Status container Penpot yang berjalan.
5. Halaman Penpot yang berhasil diakses melalui IP publik pada tahap awal.
6. File desain awal yang berhasil dibuat dan disimpan.

Bukti visual deployment akhir yang telah ditambahkan meliputi:

7. Konfigurasi `PENPOT_PUBLIC_URI` menggunakan domain HTTPS.
8. Record DNS yang mengarah ke IP publik Azure VM.
9. Inbound rules Azure NSG untuk port `80` dan `443`.
10. Isi `Caddyfile` yang meneruskan request ke `127.0.0.1:9001`.
11. Status container Caddy.
12. Log Caddy setelah HTTPS aktif.
13. Hasil resolusi DNS domain.
14. Hasil redirect HTTP dan respons HTTPS.
15. Halaman Penpot yang diakses melalui domain HTTPS.

Seluruh screenshot dan keterangannya juga tersedia pada bagian [Daftar Screenshot Dokumentasi](../deploy/deployement-guide-azure.md#daftar-screenshot-dokumentasi) dalam panduan deployment Azure.

### 8.1 Dokumentasi Implementasi

<img width="1303" height="348" alt="Informasi Azure Virtual Machine" src="https://github.com/user-attachments/assets/8f9363a0-2de3-414c-b0ec-dcecd65c158a" />
<img width="1337" height="557" alt="Konfigurasi Azure Virtual Machine" src="https://github.com/user-attachments/assets/06bb1614-8938-423c-b746-eaf040aa8479" />
<img width="1393" height="620" alt="Networking Azure Virtual Machine" src="https://github.com/user-attachments/assets/7daa5062-affa-476d-8590-3478dc8c363f" />

**Gambar 1.** Ringkasan resource Virtual Machine yang digunakan pada Microsoft Azure.

<img width="1088" height="860" alt="Koneksi SSH ke Azure VM" src="https://github.com/user-attachments/assets/2f9299e5-4651-4edf-9746-48578f41ced7" />

**Gambar 2.** Terminal berhasil terhubung ke Ubuntu Server pada Azure VM melalui SSH.

<img width="1911" height="695" alt="Versi Docker" src="https://github.com/user-attachments/assets/751cc074-3957-409f-ac19-a4aa41aa93ea" />
<img width="1891" height="321" alt="Versi Docker Compose" src="https://github.com/user-attachments/assets/7297db82-f679-4652-be95-f207562ad1e2" />

**Gambar 3.** Verifikasi bahwa Docker Engine dan Docker Compose telah terpasang pada server.

<img width="1917" height="482" alt="Status container Penpot" src="https://github.com/user-attachments/assets/fb169de1-ca0f-4146-b66d-998550b3048a" />

**Gambar 4.** Status container Penpot setelah stack dijalankan menggunakan Docker Compose.

<img width="1917" height="1151" alt="Halaman awal Penpot" src="https://github.com/user-attachments/assets/55b8359b-b188-4e01-adae-10c0ddb85922" />
<img width="1917" height="1146" alt="Dashboard Penpot" src="https://github.com/user-attachments/assets/82617e26-396e-4ed6-9c94-f9f23f39d402" />

**Gambar 5.** Penpot berhasil diakses menggunakan IP publik Azure VM pada tahap awal, sebelum DNS dan HTTPS diterapkan.

<img width="1917" height="1147" alt="File desain awal Penpot" src="https://github.com/user-attachments/assets/ae2f1d4b-9035-444a-b014-a4f570512726" />

**Gambar 6.** File desain awal berhasil dibuat dan disimpan sebagai verifikasi operasional dasar.


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

### 8.2 Keterlacakan Implementasi

| Aktivitas | Konfigurasi/perintah | Bukti |
|---|---|---|
| Menyediakan VM | Azure Portal | Gambar 1 |
| Mengakses server | `ssh -i <KEY> <USER>@<IP>` | Gambar 2 |
| Memasang runtime | `docker --version` dan `docker compose version` | Gambar 3 |
| Menjalankan stack | `docker compose -p penpot -f docker-compose.yaml up -d` | Gambar 4 |
| Memverifikasi akses awal | `http://<IP_PUBLIK_VM>:9001` | Gambar 5 |
| Memverifikasi penyimpanan awal | Membuat dan membuka file desain | Gambar 6 |
| Mengonfigurasi URI publik | `PENPOT_PUBLIC_URI: https://penpot-kdjk3.linkpc.net` | Gambar 7 |
| Mengarahkan domain ke VM | Record DNS `A` menuju `20.211.42.204` | Gambar 8 |
| Membuka akses web | Inbound rules TCP `80` dan `443` | Gambar 9 |
| Mengonfigurasi reverse proxy | `/opt/caddy/Caddyfile` | Gambar 10 |
| Menjalankan Caddy | `docker ps` dan `docker logs` | Gambar 11–12 |
| Memverifikasi DNS | `nslookup penpot-kdjk3.linkpc.net` | Gambar 13 |
| Memverifikasi redirect dan HTTPS | `curl -I` melalui HTTP dan HTTPS | Gambar 14 |
| Memverifikasi akses akhir | `https://penpot-kdjk3.linkpc.net/` melalui browser | Gambar 15 |

## 9. Pekerjaan yang Belum Selesai

### 9.1 Penggunaan Fitur Utama

Dokumentasi penggunaan akan membahas fitur-fitur berikut:

- Registrasi akun dan login.
- Pembuatan project dan file desain.
- Penggunaan shape, text, warna, dan layout.
- Pembuatan komponen dan penggunaan assets.
- Pembuatan prototype dan interaction.
- Pengelolaan halaman dan workspace.
- Kolaborasi serta berbagi project jika kondisi deployment memungkinkan.
- Ekspor hasil desain.

Setiap bagian akan dilengkapi langkah penggunaan dan screenshot.

### 9.2 Pengujian Aplikasi

Pengujian aplikasi akan dilakukan secara terstruktur dan dipisahkan dari verifikasi deployment. Rencana pengujiannya meliputi:

| Area pengujian | Skenario awal |
|---|---|
| Autentikasi | Registrasi, login, logout, dan penanganan kredensial salah |
| Project | Membuat, membuka, mengganti nama, dan menghapus project |
| File desain | Membuat, mengedit, menyimpan, dan membuka kembali file |
| Komponen desain | Shape, text, warna, layer, components, dan assets |
| Prototyping | Membuat interaction dan menjalankan preview |
| Persistensi | Memastikan data tetap tersedia setelah container restart |
| Kolaborasi | Undangan dan akses beberapa pengguna jika dapat diterapkan |
| Ekspor | Mengekspor desain ke format yang tersedia |
| Akses web | Menguji akses dan tampilan melalui browser yang digunakan |

Hasil pengujian nantinya akan mencatat langkah, hasil yang diharapkan, hasil aktual, status lulus/gagal, dan bukti screenshot.

### 9.3 Perbandingan Penpot dan Figma

Perbandingan akhir akan dilakukan setelah penggunaan dan pengujian Penpot selesai agar penilaian didasarkan pada pengalaman implementasi langsung. Aspek yang akan dibandingkan antara lain:

- Model lisensi dan keterbukaan kode sumber.
- Dukungan self-hosting.
- Kemudahan penggunaan antarmuka.
- Fitur desain dan prototyping.
- Components dan design system.
- Kolaborasi.
- Ekspor dan interoperabilitas.
- Kebutuhan infrastruktur dan pemeliharaan.
- Keamanan serta kontrol data.
- Biaya penggunaan.

## 10. Status Proyek

| Tahap | Status | Catatan |
|---|---|---|
| Pemilihan aplikasi | ✅ Selesai | Penpot dipilih sebagai aplikasi utama |
| Pemilihan aplikasi pembanding | ✅ Selesai | Figma dipilih sebagai pembanding |
| Analisis kebutuhan | ✅ Selesai | Kebutuhan dasar aplikasi dan deployment telah dianalisis |
| Konfigurasi Docker Compose | ✅ Selesai | Konfigurasi tersedia di repository |
| Panduan instalasi lokal | ✅ Selesai | Panduan penggunaan lokal telah dibuat |
| Penyediaan Azure VM | ✅ Selesai | VM Ubuntu telah tersedia |
| Instalasi Docker pada VM | ✅ Selesai | Docker dan Compose berhasil diverifikasi |
| Deployment Penpot | ✅ Selesai | Seluruh layanan dijalankan melalui Docker Compose |
| Konfigurasi DNS | ✅ Selesai | Domain mengarah ke IP publik Azure VM |
| Caddy reverse proxy | ✅ Selesai | Caddy meneruskan request ke Penpot |
| HTTPS/TLS | ✅ Selesai | Redirect dan respons HTTPS telah diverifikasi |
| Konfigurasi akses jaringan | ✅ Selesai | Port SSH, HTTP, dan HTTPS telah dikonfigurasi |
| Verifikasi deployment | ✅ Selesai | Container, DNS, redirect, proxy, dan akses HTTPS telah diverifikasi |
| Hardening port 9001 | ✅ Selesai | Port tidak dapat diakses dari internet; upstream lokal tetap digunakan Caddy |
| Dokumentasi deployment | ✅ Selesai | Panduan dan screenshot telah tersedia |
| Penggunaan fitur utama | ⏳ Belum | Menjadi tahap selanjutnya |
| Pengujian aplikasi | ⏳ Belum | Menunggu dokumentasi fitur utama |
| Perbandingan Penpot–Figma | ⏳ Belum | Dilakukan setelah pengujian Penpot |
| Dokumentasi/presentasi akhir | 🔄 Berjalan | Laporan ini akan diperbarui secara berkala |

## 11. Rencana Pembaruan Berikutnya

Urutan pekerjaan berikutnya:

1. Menggunakan dan mendokumentasikan fitur utama Penpot.
2. Menyiapkan skenario dan tabel pengujian aplikasi.
3. Menjalankan pengujian serta mengumpulkan bukti hasilnya.
4. Membandingkan Penpot dengan Figma berdasarkan hasil penggunaan.
5. Memperbarui kesimpulan dan status laporan.
6. Menyusun materi presentasi akhir.

## 12. Kesimpulan Sementara

Penpot telah berhasil diimplementasikan secara self-hosted pada Microsoft Azure menggunakan Docker Compose. DNS `penpot-kdjk3.linkpc.net` mengarah ke Azure VM, Caddy menangani reverse proxy dan TLS, HTTP dialihkan ke HTTPS, serta aplikasi dapat diakses dengan respons `200 OK` melalui [https://penpot-kdjk3.linkpc.net/](https://penpot-kdjk3.linkpc.net/).

Deployment dan pengamanan transport sudah selesai. Akses langsung port `9001` telah ditutup tanpa mengganggu akses HTTPS melalui Caddy. Fokus utama proyek selanjutnya beralih ke penggunaan dan evaluasi aplikasi. Kesimpulan akhir mengenai kualitas fitur, hasil pengujian, serta perbandingan dengan Figma akan ditambahkan setelah ketiga tahap tersebut selesai.

## 13. Referensi

1. [Penpot — Install with Docker](https://help.penpot.app/technical-guide/getting-started/docker/)
2. [Penpot — Configuration](https://help.penpot.app/technical-guide/configuration/)
3. [Caddy — Automatic HTTPS](https://caddyserver.com/docs/automatic-https)
4. [Caddy — Reverse Proxy](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy)
5. [Microsoft Azure Portal](https://portal.azure.com/)
6. [Catatan implementasi DNS dan Caddy](https://chatgpt.com/share/6ad9025f-a608-83ec-bcc4-994fe8d48b9a)
7. [Catatan troubleshooting deployment](https://chatgpt.com/share/6ad90284-65f8-83ec-913a-71fb1e251b8e)
