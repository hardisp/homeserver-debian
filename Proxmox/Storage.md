# Setup Storage pada Proxmox

## 1. Cek disk dengan `lsblk`

```sh
~: lsblk
sda   128G  (SSD dedicated Proxmox)
nvme0n1  1T (VM storage)
sdb   2T   (WD Blue)
sdc   2T   (WD Purple)
```

## 2. Format HDD Blue sebagai cloud-data
   
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

## 3. Format HDD Purple sebagai cloud-mirror

⚠ Ini akan menghapus semua data di HDD Purple.

```sh
parted /dev/sdc -- mklabel gpt
parted /dev/sdc -- mkpart primary ext4 1MiB 100%
mkfs.ext4 /dev/sdc1 -L CLOUDMIRROR
mkdir -p /mnt/cloud-mirror
echo 'UUID='$(blkid -s UUID -o value /dev/sdc1)' /mnt/cloud-mirror ext4 defaults 0 2' >> /etc/fstab
mount -a
```

## 4. Tambahkan storage ke Proxmox GUI

Sekarang kita beri tahu Proxmox bahwa 2 folder ini adalah storage.

1) Tambah cloud-data:

```
Datacenter → Storage → Add → Directory

ID: cloud-data

Directory: /mnt/cloud-data

Content:

Disk Image

Container

VZDump backup file

ISO image (optional)

Snippets (optional)

Nodes: pilih server kamu
```

```
2) Tambah cloud-mirror:

Datacenter → Storage → Add → Directory

ID: cloud-mirror

Directory: /mnt/cloud-mirror

Content:

VZDump backup file

Snippets (optional)

Nodes: server kamu

```

## 5. Verifikasi Storage

Di Proxmox → Storage:

Harus terlihat:

local (SSD 128GB) → OS, ISO, snippets

local-lvm (NVMe 1TB) → VM disks

cloud-data (HDD Blue)

cloud-mirror (HDD Purple)

Jika semuanya muncul → storage sudah siap 🎉

## Troubleshooting

### 1. Cek apakah partisi benar-benar terbentuk

Jalankan:

```sh
lsblk -f
```

Kamu harus melihat sesuatu seperti:

```sh
sdb
└─sdb1  ext4   CLOUDDATA
```

atau kalau itu HDD Purple:

```sh
sdc
└─sdc1  ext4   CLOUDMIRROR
```

Kalau muncul seperti di atas, semuanya OK.