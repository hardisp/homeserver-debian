# Disable Enterprise

# 1. Tambahkan No-Subscription Repository (gratis)

Tambahkan repo community: (sesuaikan dengan versi proxmox (9), search lagi kalo update)

```sh
cat << 'EOF' > /etc/apt/sources.list.d/pve-no-subscription.list
deb http://download.proxmox.com/debian/pve trixie pve-no-subscription
EOF
```

# 2. Disable melalui web UI

Akses WEB UI Proxmox https://192.168.18.54:8006

Pilih Node -> Updates -> Repositry.

Disable Componen Enterprise

# 3. Nonaktifkan Enterprise Repository (berbayar)

```sh
nano /etc/apt/sources.list.d/pve-enterprise.sources
```

Kemudian tambahkan atau update key Enabled:

```yml
Types: deb
URIs: https://enterprise.proxmox.com/debian/pve
Suites: trixie
Components: pve-enterprise
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
Enabled: no
```

Test dengan `apt update`
