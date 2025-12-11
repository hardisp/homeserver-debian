# Applikasi Recomended Nextcloud

## A. Nextcloud Office (OnlyOffice)

### 1️⃣ Buat folder project OnlyOffice

Di Debian VM:

```sh
mkdir -p /projects/onlyoffice
cd /projects/onlyoffice
```

### 2️⃣ Buat docker-compose

```sh
nano docker-compose.yml
```

Isi:

```yml

services:
  documentserver:
    image: onlyoffice/documentserver:latest
    container_name: onlyoffice
    restart: unless-stopped
    ports:
      - 8082:80
    environment:
      - JWT_ENABLED=true
      - JWT_SECRET=SuperSecretKey123
    networks:
      # Ganti 'proxy-net' dengan nama jaringan yang digunakan oleh NPM Anda.
      - proxy-net

networks:
  proxy-net:
    external: true
```

Save & exit.

### 3️⃣ Jalankan OnlyOffice

```sh
docker compose up -d
```

Cek:

```sh
docker ps | grep onlyoffice
# ATAU di path langsung
docker compose ps
```

Jika berjalan → lanjut.

### 4️⃣ Integrasikan di Nextcloud

Masuk `Nextcloud` → `Apps` → `Office & Text` → `OnlyOffice` → `Install`.

Lalu buka:

`Settings` → `Administration` → `ONLYOFFICE`

Isi:

Document Editing Service Address:

```sh
http://<IP_VM_KAMU>:8082 # Sesuai port di docker-compose.yml
# ATAU Kalau sudah daftar proxy host nginx proxy managet
https://onlyoffice.domainku.com
```

Contoh:

`http://192.168.18.56:8082`

Secret key:
`SuperSecretKey123`


Klik Save.

Test:
Coba buka file .docx atau .xlsx → harus muncul editor OnlyOffice.

## B. Notes (Catatan pribadi, sync antar perangkat)

Seperti Apple Notes atau Google Keep.

### 1️⃣ Install Notes App

`Nextcloud → top-right menu → Apps → Organization → Notes → Install`

Sudah langsung bisa dipakai.

## C. Install Memories App di Nextcloud

Masuk Nextcloud (web):

`Apps` → `Multimedia` → `Memories` → `Install`

Jika tidak ada, aktifkan:

`Apps` → `Featured` →` Show Unsupported Apps` (karena beberapa versi Nextcloud menyembunyikan Memories).

Setelah install:

- ✔ Menu “Memories” muncul di sidebar
- ✔ API Memories sudah aktif
- ✔ Tidak perlu restart Nextcloud
