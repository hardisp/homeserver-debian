# Nextcloud Setup

Ini adalah nextcloud setup AWAL:


## 1️⃣ Siapkan folder project dan data

Di dalam Debian VM (sebagai jiksdi):

```sh
#sudo mkdir -p /mnt/cloud-mirror/nextcloud 

mkdir -p /projects/nextcloud
sudo mkdir -p /mnt/nextcloud-data/cloud-data
sudo mkdir -p /mnt/nextcloud-backup/cloudbackup
```

Beri permission ke Nextcloud (user www-data di dalam container):

```sh
sudo chown -R www-data:www-data /mnt/nextcloud-data/cloud-data
sudo chown -R root:root /mnt/nextcloud-backup/cloudbackup
```

HDD mirror sengaja root-owned, nanti dipakai rsync backup saja, bukan ditulis langsung oleh Nextcloud.

Lalu pindah ke folder project:

```sh
cd /projects/nextcloud
```

## 2️⃣ Buat file .env untuk password

Masih di /projects/nextcloud:

```sh
nano .env
```

Isi dengan (silakan ganti password sesuai keinginanmu 👇):

```sh
NEXTCLOUD_ADMIN_USER=admin
NEXTCLOUD_ADMIN_PASSWORD=GantiPasswordAdminYangKuat123
NEXTCLOUD_DB_USER=nextcloud
NEXTCLOUD_DB_PASSWORD=GantiPasswordDBYangKuat123
NEXTCLOUD_DB_ROOT_PASSWORD=GantiRootPasswordDBYangKuat123
NEXTCLOUD_TRUSTED_DOMAINS=192.168.1.50
```

`NEXTCLOUD_TRUSTED_DOMAINS` → isi IP Debian di LAN kamu (nanti bisa ditambah domain kalau pakai Cloudflare).

**Jangan pakai password contoh di atas, ganti yang bener-bener kuat.**

`Save (Ctrl+O, Enter) → Exit (Ctrl+X).`

## 3️⃣ Buat docker-compose.yml

Masih di /projects/nextcloud:

```sh
nano docker-compose.yml
```

Isi:

```yml
version: "3.9"

services:
  db:
    image: mariadb:11
    container_name: nc-db
    restart: unless-stopped
    command: >
      --transaction-isolation=READ-COMMITTED
      --binlog-format=ROW
      --innodb-file-per-table=1
      --skip-innodb-read-only-compressed
    environment:
      - MYSQL_ROOT_PASSWORD=${NEXTCLOUD_DB_ROOT_PASSWORD}
      - MYSQL_PASSWORD=${NEXTCLOUD_DB_PASSWORD}
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=${NEXTCLOUD_DB_USER}
    volumes:
      - db:/var/lib/mysql

  redis:
    image: redis:alpine
    container_name: nc-redis
    restart: unless-stopped

  app:
    image: nextcloud:latest
    container_name: nextcloud-app
    restart: unless-stopped
    depends_on:
      - db
      - redis
    ports:
      - "9003:80"
    environment:
      - MYSQL_HOST=db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=${NEXTCLOUD_DB_USER}
      - MYSQL_PASSWORD=${NEXTCLOUD_DB_PASSWORD}
      - REDIS_HOST=redis
      - NEXTCLOUD_ADMIN_USER=${NEXTCLOUD_ADMIN_USER}
      - NEXTCLOUD_ADMIN_PASSWORD=${NEXTCLOUD_ADMIN_PASSWORD}
      - NEXTCLOUD_TRUSTED_DOMAINS=${NEXTCLOUD_TRUSTED_DOMAINS}
    volumes:
      - nextcloud:/var/www/html
      - /mnt/cloud-data/nextcloud:/var/www/html/data

volumes:
  db:
  nextcloud:
```

`Save & keluar.`

## 4️⃣ Jalankan Nextcloud

Masih di /projects/nextcloud:

```sh
docker compose up -d
```

Cek kontainer:

```sh
docker compose ps
```

Harus ada tiga:

- `nc-db`
- `nc-redis`
- `nextcloud-app`

Semua statusnya running / up dan healthy.

## 5️⃣ Akses dari browser

Dari laptop/PC di jaringan yang sama:

Buka:

http://IP-DEBIAN-KAMU:9003


Misalnya http://192.168.1.50:9003

Karena kita sudah set `env`:

Admin user: pakai `NEXTCLOUD_ADMIN_USER` dari .env

Admin password: `NEXTCLOUD_ADMIN_PASSWORD`

Nextcloud akan langsung siap tanpa wizard panjang.
