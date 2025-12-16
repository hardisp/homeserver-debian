# Script Backup Otomatis

## Jalankan backup manual pertama

Ini akan:

membuat mirror penuh

mengecek struktur folder

memastikan tidak ada error permission

Jalankan:

```sh
sudo rsync -aHAX --delete --info=progress2 \
  /mnt/nextcloud-data/cloud-data/ \
  /mnt/nextcloud-mirror/cloudbackup/

sudo rsync -a --delete --info=progress2 --exclude-from='/home/hardi/generals/rsync/exclusions.txt' \
 /data/volumes/projects/ \
 /mnt/nextcloud-backup/projects-backup
```

Keterangan:


- `-a` → archive mode (semua permission, timestamp, symlink aman)

- `-HAX` → simpan hardlink, ACL, extended attributes (Nextcloud butuh ini)

- `--delete` → hapus file yang dihapus di source (mirror akurat)

- `--info=progress2` → progres rapi

Backup pertama biasanya paling lama (tergantung isi).

Buat script backup otomatis

## SCRIPT OTOMATIS

Jika secara manual sudah siap, kita siap membuat script **OTOMATIS**

- Buat file script:

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
  - → log ditulis ke `/var/log/nextcloud-backup.log`

Jadi HDD 2:

- tidak pernah bisa ditulis oleh Nextcloud atau aplikasi lain
- hanya bisa diubah saat script jalan
- lebih aman dari kerusakan akibat bug / ransomware di VM

Backup for Projects:

```sh
sudo nano /usr/local/bin/projects-backup.sh
```

```sh
#!/bin/bash

EXCLUSIONFILE="/home/hardi/generals/rsync/exclusions.txt"
LOGFILE="/var/log/projects-backup.log"
SOURCE="/data/volumes/projects/"
DEST="/mnt/nextcloud-backup/projects-backup/"

echo "===== Backup started: $(date) =====" >> $LOGFILE
rsync -a --stats --delete --exclude-from=$EXCLUSIONFILE $SOURCE $DEST >> $LOGFILE 2>&1  
echo "===== Backup finished: $(date) =====" >> $LOGFILE
echo "" >> $LOGFILE
```

Save → keluar.

Beri izin:

```sh 
sudo chmod +x /usr/local/bin/projects-backup.sh
```

Tes manual sekali

Jalankan script:

```sh
sudo /usr/local/bin/projects-backup.sh
```

Lalu lihat log:

```sh
sudo tail -n 50 /var/log/projects-backup.log
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
  - → log ditulis ke `/var/log/nextcloud-backup.log`

Jadi HDD 2:

- tidak pernah bisa ditulis oleh Nextcloud atau aplikasi lain
- hanya bisa diubah saat script jalan
- lebih aman dari kerusakan akibat bug / ransomware di VM

## ambahkan cron job otomatis

Misalnya backup setiap malam jam 02:00:

```sh
sudo crontab -e
```

Tambahkan:

```sh
0 2 * * * /usr/local/bin/nextcloud-backup.sh
```

Kalau mau tiap 6 jam:

```sh
0 */6 * * * /usr/local/bin/nextcloud-backup.sh
```

Jika Nextcloud kamu dipakai banyak orang → saya sarankan:

✔ 1× sehari cukup
✔ atau setiap 12 jam