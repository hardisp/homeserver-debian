# Script Backup Otomatis

Buat script backup otomatis

Buat file script:

```sh
sudo nano /usr/local/bin/nextcloud-backup.sh
```

Isi:

```sh
#!/bin/bash

LOGFILE="/var/log/nextcloud-backup.log"
SOURCE="/mnt/nextcloud-data/cloud-data/"
DEST="/mnt/nextcloud-backup/cloudbackup/"

echo "===== Backup started: $(date) =====" >> $LOGFILE

rsync -aHAX --delete --info=progress2 $SOURCE $DEST >> $LOGFILE 2>&1

echo "===== Backup finished: $(date) =====" >> $LOGFILE
echo "" >> $LOGFILE
```

Save → keluar.

Beri izin:

```sh
sudo chmod +x /usr/local/bin/nextcloud-backup.sh
```

Tes manual sekali

Jalankan script:

```sh
sudo /usr/local/bin/nextcloud-backup.sh
```

Lalu lihat log:

```sh
sudo tail -n 50 /var/log/nextcloud-backup.log
```

Cek juga status mount:

```bash
mount | grep nextcloud-backup
```

## Ringkas alur kerja script

Setiap kali cron jalan:

- Disk mirror /mnt/cloud-backup → remount RW

- rsync mirror-kan:
  - `/mnt/nextcloud-data/cloud-data/`
  - ke `/mnt/nextcloud-backup/cloudbackup`

- Setelah selesai (atau error):
  - → disk mirror di-remount kembali RO
  - → log ditulis ke /var/log/nextcloud-backup.log

Jadi HDD 2:

- tidak pernah bisa ditulis oleh Nextcloud atau aplikasi lain
- hanya bisa diubah saat script jalan
- lebih aman dari kerusakan akibat bug / ransomware di VM