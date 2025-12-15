# Mengamankan akses SSH
Dengan setup ini:

- Server kamu hanya bisa diakses dengan key, tidak bisa dibobol brute-force
- Password login dinonaktifkan total (opsional setelah kita pastikan SSH key bekerja)
- Homeserver aman walaupun menggunakan WiFi dan terhubung ke cloudflare tunnel
- Fail2ban hanya jadi lapisan tambahan

## Akses Hanya dari SSH SAJA (Passowd mati)

### STEP 1 — Generate SSH Key (di device yang kamu gunakan untuk mengakses server)
Jika kamu pakai macOS / Linux / WSL:

Jalankan:

```sh
ssh-keygen -t ed25519 -C "homeserver"
```

Tekan ENTER untuk semua pertanyaan (biarkan default), atau isi passphrase jika mau lebih aman.

Key akan tersimpan di:

```yml
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

Jika kamu pakai Windows (PuTTY):

Kita bisa generate via PuTTYgen, tapi biasanya kamu sudah memakai macOS atau terminal Linux, jadi konfirmasi dulu:
Kamu generate SSH key dari apa? Mac? Linux? Windows?

Untuk sekarang saya asumsi kamu gunakan Linux/macOS.

### 🟩 STEP 2 — Kirim public key ke server (Debian VM)

Di local computer (yang kamu gunakan buat akses server):

```sh
ssh-copy-id hardi@192.168.18.14 # sesuaikan dengan user dan IP
# Akan muncul prompt minta password user jiksdi.
# Masukkan password → selesai.
```

Kalau pakai port custom (tidak), skip.

#### !!!ALTERNATIVE!!! 

Jika ssh-copy-id tidak ada, kita lakukan manual:

Manual copy:

Di local komputer:

```sh
cat ~/.ssh/id_ed25519.pub
```

Copy outputnya.

Di server Debian:

```sh
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
```

Paste key ke dalam file.

Save → `exit`.

Set permission:

```sh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### 🟩 STEP 3 — Test login pakai key (WAJIB sebelum disable password)

Di local komputer:

```sh
ssh hardi@192.168.18.15 # sesuaikan user dan IP_Address
```

Jika berhasil login tanpa diminta password, berarti sukses.

Tolong coba ini SEBELUM kita mematikan password login.

Kalau gagal login dan masih minta password coba ADD SSH

```sh
ssh-add ~/.ssh/id_ed25519.pub # sesuaikan dengan file yang kamu buat tadi
```

🟩 STEP 4 — Disable login password (SETELAH key bisa masuk)

Setelah login via key berhasil, kita disable password agar aman.

Edit SSH config:

```sh
sudo nano /etc/ssh/sshd_config
````

Ubah atau tambah:

```sh
PasswordAuthentication no
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no
UsePAM yes
```

Pastikan:

```sh
PermitRootLogin no
```

Save → `exit`.

Restart SSH:

```sh
sudo systemctl restart ssh
```

🟩 STEP 5 — Test login lagi (password harus ditolak)

Di local komputer lain:

```sh
ssh jiksdi@192.168.18.56
```

Jika key benar → langsung login
Jika key tidak ada → akan ditolak (bukan minta password)

Ini tanda server sudah 100% aman.

## 🟦 STEP OPSIONAL (Recommended) — Tambah Firewall Rule “Hanya izinkan LAN” untuk SSH

```sh
sudo ufw allow from 192.168.18.0/24 to any port 22 proto tcp
sudo ufw delete allow ssh
```

Ini membuat SSH hanya bisa diakses dari LAN, bukan internet.
