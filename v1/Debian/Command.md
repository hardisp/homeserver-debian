Cek disk dengan `lsblk`

```sh
~: lsblk
sda   128G  (SSD dedicated Proxmox)
nvme0n1  1T (VM storage)
sdb   2T   (WD Blue)
sdc   2T   (WD Purple)
```

Format HDD:
### dari lsblk kita bisa format

!!! PENTING - JALANKAN PER BARIS !!!

```sh
# !!! PENTING - JALANKAN PER BARIS !!!
parted /dev/sdb -- mklabel gpt
parted /dev/sdb -- mkpart primary ext4 1MiB 100%
mkfs.ext4 /dev/sdb1 -L CLOUDDATA
mkdir -p /mnt/cloud-data
echo 'UUID='$(blkid -s UUID -o value /dev/sdb1)' /mnt/cloud-data ext4 defaults 0 2' >> /etc/fstab
mount -a
```

