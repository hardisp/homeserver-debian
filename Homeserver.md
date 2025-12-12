# 📝 Ringkasan Setup Homeserver Pribadi

Berikut adalah rangkuman lengkap dari konfigurasi *homeserver* berbasis Proxmox dan Docker yang telah kita selesaikan.

---

## 1. Dasar Infrastruktur (Proxmox & Debian VM)

| Komponen | Status | Detail & Konfigurasi |
| :--- | :--- | :--- |
| **Hypervisor** | **OK** | Proxmox VE 9.1.1. |
| **Host Server** | **OK** | Virtual Machine (VM) Debian 12 (Bookworm) diinstal dengan SSH, berfungsi sebagai *host* untuk semua layanan Docker. |
| **Docker Engine** | **OK** | Docker dan Docker Compose diinstal untuk mengelola *container*. |
| **Shell CLI** | **OK** | Oh My Zsh diinstal di Host Debian untuk *shell experience* yang lebih baik (termasuk perbaikan *path* NVM). |

---

## 2. Jaringan & Akses Jarak Jauh (VPN & DDNS)

**Tujuan:** Akses aman dari mana saja.

| Komponen | Status | Detail & Konfigurasi |
| :--- | :--- | :--- |
| **WireGuard VPN** | **OK** (Server Side) | VPN *tunnel* dikonfigurasi menggunakan Docker. |
| **DDNS** | **OK** | Layanan DDNS (`hardi.ddns.net`) dikonfigurasi di *router* untuk melacak IP Publik dinamis. |
| **NAT/Iptables** | **OK** | Aturan `iptables` (*Masquerading*/NAT) ditambahkan di Host Debian untuk merutekan *traffic* dari klien VPN (`10.x.x.x`) keluar ke internet. |
| **Port Forwarding** | **BLOKADE CGNAT** | Port 51820 (UDP) **tertutup** dari internet publik. **Perlu Aksi:** Hubungi ISP untuk membuka *port* atau meminta IP Publik Non-CGNAT. |

---

## 3. Layanan Aplikasi (Akses Internal)

**Tujuan:** Fungsionalitas inti *homeserver*.

| Komponen | Status | Detail & Konfigurasi |
| :--- | :--- | :--- |
| **AdGuard Home (AGH)** | **OK** | Berfungsi penuh sebagai *server* DNS utama (`192.168.18.15`) untuk *ad blocking* di jaringan lokal. |
| **Nginx Proxy Manager (NPM)** | **OK** | Berfungsi untuk *reverse proxy* dan menerbitkan domain lokal (`adguard.jiksdi.my.id`). |
| **Nextcloud** | **PERLU DIINSTAL ULANG** | Diperlukan instalasi ulang bersih setelah konflik *volume mounting* dan *Internal Server Error*. **Aksi:** Siapkan Bind Mount (`/mnt/nextcloud-data`) dan instal ulang. |
| **Grafana** | **TIDAK DITERAPKAN** | Sudah dibuat rencana konfigurasi Proxy Host di NPM menggunakan `grafana.jiksdi.my.id` dan Port 3000 internal. |

---

## 4. Manajemen Daya (WIP)

| Komponen | Status | Detail & Konfigurasi |
| :--- | :--- | :--- |
| **APC UPS & NUT** | **BLOKADE FISIK** | Rencana konfigurasi NUT (Network UPS Tools) di Proxmox VE Host untuk *auto shutdown*. **Status Saat Ini:** Kabel USB data UPS tidak terdeteksi oleh sistem Proxmox. **Perlu Aksi:** Periksa dan pastikan koneksi kabel USB data dari UPS ke *host* Proxmox. |

---

## 🚀 Prioritas Berikutnya

1.  **Selesaikan Blokade Jaringan:** Hubungi ISP untuk mengatasi **Blokade CGNAT** (Port 51820 UDP).
2.  **Instalasi Aplikasi:** Lakukan *deployment* bersih untuk **Nextcloud** dan **Grafana**.
3.  **Amankan Daya:** Perbaiki koneksi **UPS USB** untuk mengaktifkan *auto shutdown* NUT.