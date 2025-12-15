# Membuat tunel

## Membuat tunel Homeserver

Cloudflare baru-baru ini menyederhanakan proses pembuatan tunnek melalui Dashboard Web (disebut Cloudflare Zero Trust atau Cloudflare for Teams), yang memang melibatkan Network Connector.

Mari kita gunakan metode yang paling modern, yaitu `Cloudflare Zero Trust Dashboard`, yang mungkin lebih sesuai dengan apa yang Anda ingat.

### 🛠️ Ringkasan Setup Cloudflare Tunnel (Menggunakan Dashboard Zero Trust)

Proses ini memindahkan banyak langkah konfigurasi dari Host Debian ke antarmuka web Cloudflare.

#### Tahap 1: Instalasi cloudflared dan Login

Ini adalah satu-satunya bagian yang harus dilakukan di terminal Host Debian Anda.
Instalasi: Ikuti langkah instalasi cloudflared yang sama seperti sebelumnya di Host Debian Anda:

```sh
# (Ulangi Tahap 2 dari jawaban sebelumnya)
sudo apt update
sudo apt install cloudflared 
```

Otorisasi: Login dan otorisasi daemon cloudflared Anda ke akun Cloudflare Anda:

```sh
cloudflared tunnel login
# (Ikuti instruksi URL di browser untuk otorisasi).
```

#### Tahap 2: Pembuatan Tunnel di Dashboard Zero Trust

Ini adalah tahap di mana Anda menggunakan web interface Cloudflare (Dashboard Zero Trust).

- Akses Dashboard: Masuk ke akun Cloudflare Anda, lalu navigasikan ke bagian Zero Trust (biasanya di sebelah kiri).
- Navigasi ke Tunnels: Di Dashboard Zero Trust, pilih menu `Access` > `Tunnels`.
- Buat Tunnel Baru: Klik `Create a Tunnel`.
- Beri nama pada tunnel Anda (misalnya, nextcloud-tunnel).
- Klik `Save Tunnel`.
  
- Pilih Konektor (OS): Cloudflare akan menampilkan jendela yang meminta Anda memilih sistem operasi.
Pilih Debian.
- Cloudflare akan menampilkan command instalasi dan perintah untuk menjalankan Network Connector Anda.

#### Tahap 3: Menjalankan Network Connector (Menghubungkan Tunnel)

Anda harus menyalin dan menjalankan perintah yang diberikan Cloudflare di Host Debian Anda.
Jalankan Perintah Konektor:

- Cloudflare akan memberikan perintah yang panjang dan unik, yang akan terlihat seperti ini:

```sh
sudo cloudflared service install <TUNNEL-ID>
```
Jalankan perintah tersebut di Host Debian Anda.

- Setelah selesai, service cloudflared akan start secara otomatis, dan status tunnel Anda di dashboard web Cloudflare akan berubah menjadi `Active` (Connected).
- 
#### Tahap 4: Konfigurasi Rute (Public Hostnames)

Ini adalah tahap di mana Anda mendefinisikan routing untuk `cloud.jiksdi.my.id`.

- Navigasi ke Routing: Di Dashboard Cloudflare, di bagian tunnel yang baru Anda buat, klik Configure.
- Tambahkan Public Hostname: Di tab Public Hostnames, klik Add a public hostname.
- Definisikan Rute:
    - Domain: cloud.jiksdi.my.id
- Service:
    - Type: HTTP
    - URL: 192.168.18.15:81 (Ini mengarahkan traffic ke Nginx Proxy Manager Anda)
    - Klik Save Hostname.
  
Cara Kerja:
- Ketika pengguna mengakses https://cloud.jiksdi.my.id, Cloudflare akan menerima traffic.
- Cloudflare mengirimkannya melalui Tunnel yang aktif (Network Connector) di Host Debian Anda.
- Host Debian meneruskan traffic ke 192.168.18.15:81 (NPM).
- NPM kemudian meneruskan traffic ke container Nextcloud.
- Setelah langkah-langkah ini selesai, CGNAT Anda teratasi untuk layanan Nextcloud.~~

