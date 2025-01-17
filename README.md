# Langkah Konfigurasi Materi MTCNA
## Konfigurasi PPTP Server
### Topologi
[Topologi PPTP Tunnel][def]
### Bagian Pertama: Konfigurasi IP Address
#### R-Server
Konfigurasi IP Address Router Server
**Note:** Sesuaikan interface link-nya yang terpasang
```bash
ip dhcp-client pr
ip address add address=192.168.XX.1/24 interface=ether2
ip address pr
```
#### R-client
Konfigurasi IP Address Router client
```bash
ip dhcp-client pr
ip address add address=192.168.XX+33.1/24 interface=ether2
ip address pr
```

#### PC-1
```bash
ip 192.168.XX.2/24 192.168.XX.1
ip dns 192.168.XX.1
```
#### PC-3
```bash
ip 192.168.XX.3/24 192.168.XX.1
ip dns 192.168.XX.1
sh ip
save
```
#### PC-2
```bash
ip 192.168.XX+33.2/24 192.168.XX+33.1
ip dns 192.168.XX+33.1
sh ip
save
```
### Bagian Kedua Uji Konektifitas
#### PC-1 ke R-Server
```bash
ping 192.168.XX.1
```
#### PC-3 ke R-client
```bash
ping 192.168.XX+33.1
```
#### R-Server ke R-client
```bash
ping 192.168.255.146
```

### Bagian Ketiga: KOnfigurasi PPTP-Server
#### R-Server
Konfigurasi PPTP pada Router Server
```bash
interface pptp-server server set enabled=yes
interface pptp-server pr
```
Tambahkan ppp secret
```bash
ppp secret add name=nama-kalian password=12345678 service=pptp local-address=172.16.XX.1 remote-address=172.16.XX.2
```

### Bagian Keempat: Konfigurasi PPTP-client
#### R-client
Knfigruasi PPTP Client pada R-Client
```bash
interface pptp-client add user=nama-kalian password=12345678 connect-to=192.168.255.145
interface pptp-client pr
```


[def]: https://raw.githubusercontent.com/saifulindo/MTCNA/main/topologi-pptp.jpg