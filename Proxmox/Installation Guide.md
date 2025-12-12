# Proxmox 9.1 Installation Guide

## STEP 1 – INSTALL PROXMOX VE di SSD SATA 128GB

🧩 1.1. Persiapan USB Installer

Pastikan kamu punya:

```yml
USB 8GB+

File ISO Proxmox terbaru

Laptop/PC untuk membuat USB installer
```

Cara membuat installer:

```sh
Download ISO Proxmox VE dari website resmi.

Jalankan Rufus (Windows):

Select ISO Proxmox

Mode: GPT / UEFI

Start → tunggu selesai
```

Jika sudah → colok USB ke server kamu.

🧩 1.2. Boot dari USB

Nyalakan PC server kamu.

Tekan DEL atau F2 untuk masuk BIOS.

Masuk ke menu Boot, lalu pilih USB sebagai urutan boot pertama.

Save & Restart.

Jika berhasil, kamu akan melihat:

👉 “Proxmox VE Installer”

🧩 1.3. Mulai Instalasi Proxmox

Pilih: `Install Proxmox VE`

`Accept license.`

Nant Rekomendasi hostname untuk server kamu: 
```
homeserver
````
Dan untuk FQDN (`Full Hostname`) isi:

```
homeserver.local
```

⚙️ 1.4. Pilih Disk untuk Instalasi

PENTING:

`Pastikan kamu memilih SSD SATA 128GB, BUKAN NVMe.`

Biasanya terdeteksi sebagai:

`/dev/sda atau /dev/sdb`

dengan nama: "ATA … 128GB" atau “SATA SSD”

⚠ Jangan pilih NVMe, karena NVMe untuk VM storage nanti.

Pilih SSD tersebut.

🗄️ 1.5. Pilih File System

Di menu file system:

`ext4` → paling aman, paling mudah

(`ZFS mirror` atau `RAID` tidak perlu, ini disk khusus OS)

Rekomendasi: `ext4`

🌍 1.6. Lokasi & Keyboard

Country: Indonesia

Timezone: Asia/Makassar

Keyboard: default

Lanjut.

🔐 1.7. Set Password & Email

Masukkan password root untuk Proxmox.

`Pastikan kamu catat ya`.

Email bisa pakai email pribadi (penting untuk notification).

🌐 1.8. Network Setup (LAN sementara)

Meskipun kamu akan pakai WiFi pada sistem final, saat instalasi kita tetap pakai LAN.

Interface: pilih LAN (eth0/enp3s0)

Set IP:

Bisa DHCP otomatis

Atau set static (lebih baik):

Contoh static:

```
IP: 192.168.1.50
Netmask: 255.255.255.0
Gateway: 192.168.1.1
DNS: 1.1.1.1
```
Lanjut dan Install.

⏳ 1.9. Tunggu Instalasi Selesai

Biasanya 5–10 menit.

Setelah selesai:

Proxmox minta reboot

`Cabut USB installer`

`Reboot.`

💻 1.10. Login ke Web UI Proxmox

Dari laptop/PC yg sama jaringan (via LAN):

Buka browser → akses:

`https://192.168.1.50:8006`


Atau IP yang muncul di layar server setelah reboot.

Login:

User: root

Password: sesuai instalasi

Realm: Linux PAM

Jika kamu sudah bisa masuk → STEP 1 selesai sukses 🎉
