```sh
# Untuk membuat Nextcloud mengetahui perubahan, Anda harus menjalankan file scan:
docker compose exec --user www-data nextcloud php occ files:scan --all
```
Backup Manual Dengan RSYNC

```sh
sudo rsync -aHAX --delete --info=progress2 \
  /mnt/nextcloud-data/cloud-data/ \
  /mnt/nextcloud-backup/cloudbackup/
```
install rsync jika belum
```sh 
 sudo apt update
sudo apt install rsync -y
```
