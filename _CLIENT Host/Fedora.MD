# Setting berguna saat konek dengan home server untuk Fedora Linux

## Manual DNS Server

Pertama setting dulu dns secara manual dari `Setting` dan ubah IPv6 Method ke Link-Local Only untuk mencegah menggunak ipv6 DNS

kemudian setting dengan DNS agar settingan pada menu Setting bekerja:

Check koneksi dengan nmcli

```sh
➜  ~  nmcli connection show

#Result 
NAME        UUID                                  TYPE      DEVICE 
SUPUTRA 5G  943ad40a-7d95-4289-98c7-b572d325d9c7  wifi      wlp2s0 
lo          5537e203-d737-467b-a038-41bf15a2ad6f  loopback  lo     
```

Kemudian ignore autodns dengan modifikasi

```sh
➜  ~  sudo nmcli connection modify "SUPUTRA 5G" ipv4.ignore-auto-dns yes
```

Restart Koneksi
```sh
➜  ~  sudo nmcli connection up "SUPUTRA 5G"
# Result
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)
```

Cek apakah DNS Sudah berubah:

```
➜  ~  resolvectl status | grep 'DNS Server'
Current DNS Server: 192.168.18.15
       DNS Servers: 192.168.18.15 1.1.1.1
```
