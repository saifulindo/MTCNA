# Langkah Konfigurasi Materi MTCNA
## Konfigurasi PPTP Server
### Topologi
![Topologi PPTP Tunnel](https://github.com/saifulindo/MTCNA/raw/main/topologi-pptp.jpg)
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
#### PC-2 ke R-client
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

### Bagian Kelima: Verifikasi Tunneling
#### R-Server
Verifikasi Interface dan IP Address
```bash
interface pr
Flags: D - dynamic, X - disabled, R - running, S - slave
 #     NAME                                TYPE       ACTUAL-MTU L2MTU  MAX-L2MTU MAC-ADDRESS
 0  R  ether1                              ether            1500                  0C:25:6E:30:00:00
 1  R  ether2                              ether            1500                  0C:25:6E:30:00:01
 2  R  ether3                              ether            1500                  0C:25:6E:30:00:02
 3  R  ether4                              ether            1500                  0C:25:6E:30:00:03
 4 DR  <pptp-nama-kalian>                  pptp-in          1450
ip address pr
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0   192.168.34.1/24    192.168.34.0    ether2
 1 D 192.168.255.145/24 192.168.255.0   ether1
 2 D 172.16.34.1/32     172.16.34.2     <pptp-nama-kalian>
```
#### R-Client
Verifikasi interface dan IP Address Tunnel
```bash
interface pr
Flags: D - dynamic, X - disabled, R - running, S - slave
 #     NAME                                TYPE       ACTUAL-MTU L2MTU  MAX-L2MTU MAC-ADDRESS
 0  R  ether1                              ether            1500                  0C:5C:16:CD:00:02
 1  R  ether2                              ether            1500                  0C:5C:16:CD:00:03
 2  R  ether3                              ether            1500                  0C:5C:16:CD:00:00
 3  R  ether4                              ether            1500                  0C:5C:16:CD:00:01
 4  R  pptp-out1                           pptp-out         1450
ip address pr
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0   192.168.67.1/24    192.168.67.0    ether4
 1 D 192.168.255.146/24 192.168.255.0   ether3
 2 D 172.16.34.2/32     172.16.34.1     pptp-out1
```

### Bagian Keenam: Konfiruasi Routing Static
#### R-Client
```bash
ip route add dst-address=192.168.XX.0/24 gateway=172.16.XX.1
ip route pr
Flags: X - disabled, A - active, D - dynamic, C - connect, S - static, r - rip, b - bgp, o - ospf, m - mme,
B - blackhole, U - unreachable, P - prohibit
 #      DST-ADDRESS        PREF-SRC        GATEWAY            DISTANCE
 0 ADS  0.0.0.0/0                          192.168.255.2             1
 1 ADC  172.16.34.1/32     172.16.34.2     pptp-out1                 0
 2 A S  192.168.34.0/24                    172.16.34.1               1
 3 ADC  192.168.67.0/24    192.168.67.1    ether4                    0
 4 ADC  192.168.255.0/24   192.168.255.146 ether3                    0
```
#### R-Server
```bash
ip route add dst-address=192.168.XX+33.0/24 gateway=172.16.XX.2
ip route pr
Flags: X - disabled, A - active, D - dynamic, C - connect, S - static, r - rip, b - bgp, o - ospf, m - mme,
B - blackhole, U - unreachable, P - prohibit
 #      DST-ADDRESS        PREF-SRC        GATEWAY            DISTANCE
 0 ADS  0.0.0.0/0                          192.168.255.2             1
 1 ADC  172.16.34.2/32     172.16.34.1     <pptp-nama-kalian>        0
 2 ADC  192.168.34.0/24    192.168.34.1    ether2                    0
 3 A S  192.168.67.0/24                    172.16.34.2               1
 4 ADC  192.168.255.0/24   192.168.255.145 ether1                    0
```

### Baian Ketujuh: Verifikasi route Tunnel
#### Uji Konektivitas PC-1 ke PC-2
```bash
ping 192.168.XX+33.2
trace 192.168.XX+33.2
trace to 192.168.67.2, 8 hops max, press Ctrl+C to stop
 1   192.168.34.1   12.650 ms  6.025 ms  2.978 ms
 2   172.16.34.2   32.057 ms  6.994 ms  12.127 ms
 3   *192.168.67.2   17.213 ms (ICMP type:3, code:3, Destination port unreachable)
```

## Konfigurasi L2TP Server
### Bagian Pertama: Topologi
![Topologi L2TP](https://raw.githubusercontent.com/saifulindo/MTCNA/refs/heads/main/topologi-l2tp.jpg)

### Bagian Kedua: Konfigurasi IP Address
#### Office-Center
Verifikasi Mac Address untuk mengetahui posisi ethernet
```bash
[admin@MikroTik] > interface ethernet pr
Flags: X - disabled, R - running, S - slave
 #    NAME                                                                            MTU MAC-ADDRESS       ARP
 0 R  ether1                                                                         1500 0C:81:7E:C4:00:00 enabled
 1 R  ether2                                                                         1500 0C:81:7E:C4:00:01 enabled
 2 R  ether3                                                                         1500 0C:81:7E:C4:00:02 enabled
 3 R  ether4                                                                         1500 0C:81:7E:C4:00:03 enabled
```
Perikas IP DHCP Client
```bash
[admin@MikroTik] > ip dhcp-client pr
Flags: X - disabled, I - invalid, D - dynamic
 #   INTERFACE                                              USE-PEER-DNS ADD-DEFAULT-ROUTE STATUS        ADDRESS
 0   ether1                                                 yes          yes               bound         192.168.255.147/24
```
Tambahkan IP Addrees pada interface dan verifikasi
```bash
[admin@MikroTik] > ip address add address=172.16.35.1/24 interface=ether2
[admin@MikroTik] > ip address pr
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0 D 192.168.255.147/24 192.168.255.0   ether1
 1   172.16.35.1/24     172.16.35.0     ether2
```

#### Office-Branch
Verifikasi Mac Address untuk mengetahui posisi ethernet
```bash
[admin@MikroTik] > interface ethernet pr
Flags: X - disabled, R - running, S - slave
 #    NAME                                                                            MTU MAC-ADDRESS       ARP
 0 R  ether1                                                                         1500 0C:E7:1E:E8:00:00 enabled
 1 R  ether2                                                                         1500 0C:E7:1E:E8:00:01 enabled
 2 R  ether3                                                                         1500 0C:E7:1E:E8:00:02 enabled
 3 R  ether4                                                                         1500 0C:E7:1E:E8:00:03 enabled
```
Perikas IP DHCP Client
```bash
[admin@MikroTik] > ip dhcp-client pr
Flags: X - disabled, I - invalid, D - dynamic
 #   INTERFACE                                              USE-PEER-DNS ADD-DEFAULT-ROUTE STATUS        ADDRESS
 0   ether1                                                 yes          yes               bound         192.168.255.148/24
```
Uji Konektifitas dengan Office Center
```bash
[admin@MikroTik] > ping 192.168.255.147
  SEQ HOST                                     SIZE TTL TIME  STATUS
    0 192.168.255.147                            56  64 43ms
    1 192.168.255.147                            56  64 13ms
    2 192.168.255.147                            56  64 7ms
    sent=3 received=3 packet-loss=0% min-rtt=7ms avg-rtt=21ms max-rtt=43ms
```
Tambahkan IP Addrees pada interface dan verifikasi
```bash
[admin@MikroTik] > ip address add address=172.16.68.1/23 interface=ether2
[admin@MikroTik] > ip address pr
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0 D 192.168.255.148/24 192.168.255.0   ether1
 1   172.16.68.1/24     172.16.68.0     ether2
```
#### VPCS PC1
Konfigurasi IP Address
```bash
PC1> ip 172.16.35.2/24 172.16.35.1
Checking for duplicate address...
PC1 : 172.16.35.2 255.255.255.0 gateway 172.16.35.1
```
Uji KOnektifitas deng Interface Office Center
```bash
PC1> ping 172.16.35.1
84 bytes from 172.16.35.1 icmp_seq=1 ttl=64 time=5.031 ms
84 bytes from 172.16.35.1 icmp_seq=2 ttl=64 time=1.729 ms
84 bytes from 172.16.35.1 icmp_seq=3 ttl=64 time=1.589 ms
84 bytes from 172.16.35.1 icmp_seq=4 ttl=64 time=1.015 ms
84 bytes from 172.16.35.1 icmp_seq=5 ttl=64 time=1.372 ms
```

### VPCS PC2 atau PC3
Konfigurasi IP Address
```bash
PC1> ip 172.16.68.2/23 172.16.68.1
Checking for duplicate address...
PC1 : 172.16.68.2 255.255.254.0 gateway 172.16.68.1
```
Uji Konektifitas deng Interface Office Branch
```bash
PC2> ping 172.16.68.1
84 bytes from 172.16.68.1 icmp_seq=1 ttl=64 time=4.896 ms
84 bytes from 172.16.68.1 icmp_seq=2 ttl=64 time=1.171 ms
84 bytes from 172.16.68.1 icmp_seq=3 ttl=64 time=1.222 ms
84 bytes from 172.16.68.1 icmp_seq=4 ttl=64 time=2.348 ms
84 bytes from 172.16.68.1 icmp_seq=5 ttl=64 time=1.457 ms
```

### Bagian ketiga: Konfigurasi L2TP
#### Office Center
Aktifkan L2TP Server
```bash
[admin@MikroTik] > interface l2tp-server server set enabled=yes
```
Tambah user secret
```bash
[admin@MikroTik] > ppp secret add name=nama-kalian service=l2tp password=12345678 local-address=10.0.0.1 remote-address=10.0.0.2
```

#### Office Brance
Konfigurasi L2TP Client 
```bash
[admin@MikroTik] > interface l2tp-client add connect-to=192.168.255.147 user=nama-kalian password=12345678
```
Verifikasi L2TP Client
```bash
Flags: X - disabled, R - running
 0 X  name="l2tp-out1" max-mtu=1450 max-mru=1450 mrru=disabled connect-to=192.168.255.147 user="nama-kalian"
      password="12345678" profile=default-encryption keepalive-timeout=60 use-peer-dns=no use-ipsec=no ipsec-secret=""
      allow-fast-path=no add-default-route=no dial-on-demand=no allow=pap,chap,mschap1,mschap2
```
Enable L2TP Client yang baru saja ditambahkan dan verifikasi
```bash
[admin@MikroTik] > interface l2tp-client set disabled=no numbers=0
Flags: X - disabled, R - running
 0  R name="l2tp-out1" max-mtu=1450 max-mru=1450 mrru=disabled connect-to=192.168.255.147 user="nama-kalian"
      password="12345678" profile=default-encryption keepalive-timeout=60 use-peer-dns=no use-ipsec=no ipsec-secret=""
      allow-fast-path=no add-default-route=no dial-on-demand=no allow=pap,chap,mschap1,mschap2
```

#### Verifikasi Dial-up Interface dan IP Address
Office Center
```bash
[admin@MikroTik] > interface pr
Flags: D - dynamic, X - disabled, R - running, S - slave
 #     NAME                                TYPE       ACTUAL-MTU L2MTU  MAX-L2MTU MAC-ADDRESS
 0  R  ether1                              ether            1500                  0C:81:7E:C4:00:00
 1  R  ether2                              ether            1500                  0C:81:7E:C4:00:01
 2  R  ether3                              ether            1500                  0C:81:7E:C4:00:02
 3  R  ether4                              ether            1500                  0C:81:7E:C4:00:03
 4 DR  <l2tp-nama-kalian>                  l2tp-in          1450
[admin@MikroTik] > ip address pr
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0 D 192.168.255.147/24 192.168.255.0   ether1
 1   172.16.35.1/24     172.16.35.0     ether2
 2 D 10.0.0.1/32        10.0.0.2        <l2tp-nama-kalian>
```

Office Branch
```bash
[admin@MikroTik] > interface pr
Flags: D - dynamic, X - disabled, R - running, S - slave
 #     NAME                                TYPE       ACTUAL-MTU L2MTU  MAX-L2MTU MAC-ADDRESS
 0  R  ether1                              ether            1500                  0C:E7:1E:E8:00:00
 1  R  ether2                              ether            1500                  0C:E7:1E:E8:00:01
 2  R  ether3                              ether            1500                  0C:E7:1E:E8:00:02
 3  R  ether4                              ether            1500                  0C:E7:1E:E8:00:03
 4  R  l2tp-out1                           l2tp-out         1450
[admin@MikroTik] > ip address pr
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0 D 192.168.255.148/24 192.168.255.0   ether1
 1   172.16.68.1/23     172.16.68.0     ether2
 2 D 10.0.0.2/32        10.0.0.1        l2tp-out1
```


### Bagian Kelima: Aktifkan L2TP IPSec(Opsional)
#### Konfigurasi Profile PPP Secret L2TP
Buat IP Pool untuk L2TP
```bash
[admin@MikroTik] > ip pool add name=L2TP-Pool ranges=10.0.0.2-10.0.0.100
```
Buat Profile PPP
```bash
[admin@MikroTik] > ppp profile add name=L2TP-Profile local-address=10.0.0.1 remote-address=L2TP-Pool
```
Atur ppp secret ke profile yang telah dibuat
```bash
[admin@MikroTik] > ppp secret set numbers=0 profile=L2TP-Profile local-address=0.0.0.0 remote-address=0.0.0.0
```

#### Aktifkan IPSec pada kedua Router
**Office Center**
```bash
[admin@MikroTik] > interface l2tp-server server set use-ipsec=yes ipsec-secret=rahasia
```
**Office Branch**
```bash
[admin@MikroTik] > interface l2tp-client set use-ipsec=yes ipsec-secret=rahasia numbers=0
```
#### Lakukan disable enable untuk Dial-up Ulang
Office Center
```bash
[admin@MikroTik] > interface l2tp-server server set enabled=no
[admin@MikroTik] > interface pr
Flags: D - dynamic, X - disabled, R - running, S - slave
 #     NAME                                TYPE       ACTUAL-MTU L2MTU  MAX-L2MTU MAC-ADDRESS
 0  R  ether1                              ether            1500                  0C:81:7E:C4:00:00
 1  R  ether2                              ether            1500                  0C:81:7E:C4:00:01
 2  R  ether3                              ether            1500                  0C:81:7E:C4:00:02
 3  R  ether4                              ether            1500                  0C:81:7E:C4:00:03
[admin@MikroTik] > interface l2tp-server server set enabled=yes
[admin@MikroTik] > interface pr
Flags: D - dynamic, X - disabled, R - running, S - slave
 #     NAME                                TYPE       ACTUAL-MTU L2MTU  MAX-L2MTU MAC-ADDRESS
 0  R  ether1                              ether            1500                  0C:81:7E:C4:00:00
 1  R  ether2                              ether            1500                  0C:81:7E:C4:00:01
 2  R  ether3                              ether            1500                  0C:81:7E:C4:00:02
 3  R  ether4                              ether            1500                  0C:81:7E:C4:00:03
```
#### Verifikasi kedua Ruter
Office Center
```bash
[admin@MikroTik] > ip address pr
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0 D 192.168.255.147/24 192.168.255.0   ether1
 1   172.16.35.1/24     172.16.35.0     ether2
 2 D 10.0.0.1/32        10.0.0.100      <l2tp-nama-kalian>
```
Office Branch
```bash
[admin@MikroTik] > ip address pr
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0 D 192.168.255.148/24 192.168.255.0   ether1
 1   172.16.68.1/23     172.16.68.0     ether2
 2 D 10.0.0.100/32      10.0.0.1        l2tp-out1
```

### Bagian Keenam: Konfigurasi Routing
#### Office Center
**Note**: Gateway menggunakan `10.0.0.100` sesuai yang didapatkan. jika mengunakan IP Pool. ip yang didapatkan yang paling terakhir. jika tanpa ip pool sesuai konfigurasi PPP Secret-nya 10.0.0.2.
```bash
[admin@MikroTik] > ip route add dst-address=172.16.68.0/23 gateway=10.0.0.100
[admin@MikroTik] > ip route pr
Flags: X - disabled, A - active, D - dynamic, C - connect, S - static, r - rip, b - bgp, o - ospf, m - mme,
B - blackhole, U - unreachable, P - prohibit
 #      DST-ADDRESS        PREF-SRC        GATEWAY            DISTANCE
 0 ADS  0.0.0.0/0                          192.168.255.2             1
 1 ADC  10.0.0.100/32      10.0.0.1        <l2tp-nama-kalian>        0
 2 ADC  172.16.35.0/24     172.16.35.1     ether2                    0
 3 A S  172.16.68.0/23                     10.0.0.100                1
 4 ADC  192.168.255.0/24   192.168.255.147 ether1                    0
```
#### Office Branch
```bash
[admin@MikroTik] > ip route add dst-address=172.16.35.0/24 gateway=10.0.0.1
[admin@MikroTik] > ip route pr
Flags: X - disabled, A - active, D - dynamic, C - connect, S - static, r - rip, b - bgp, o - ospf, m - mme,
B - blackhole, U - unreachable, P - prohibit
 #      DST-ADDRESS        PREF-SRC        GATEWAY            DISTANCE
 0 ADS  0.0.0.0/0                          192.168.255.2             1
 1 ADC  10.0.0.1/32        10.0.0.100      l2tp-out1                 0
 2 A S  172.16.35.0/24                     10.0.0.1                  1
 3 ADC  172.16.68.0/23     172.16.68.1     ether2                    0
 4 ADC  192.168.255.0/24   192.168.255.148 ether1                    0
```

### Bagian Ketujuh: Uji Konektifitas
Uji Konektifitas Ping PC1 - PC2
```bash
PC1> ping 172.16.68.2
84 bytes from 172.16.68.2 icmp_seq=1 ttl=62 time=17.834 ms
84 bytes from 172.16.68.2 icmp_seq=2 ttl=62 time=15.312 ms
84 bytes from 172.16.68.2 icmp_seq=3 ttl=62 time=9.745 ms
84 bytes from 172.16.68.2 icmp_seq=4 ttl=62 time=19.429 ms
84 bytes from 172.16.68.2 icmp_seq=5 ttl=62 time=8.247 ms
```
## Konfigurasi PPPoE Server
### Bagian Pertama: Topologi
![TOpologi PPPoE Server](https://raw.githubusercontent.com/saifulindo/MTCNA/refs/heads/main/topologi-pppoe.jpg)
### Bagian Kedua: Konfigurasi IP Address
#### R-IP
Konfiruasi IP Address melalui `ip dhcp-client`:
**Note:** Sesuaikan dengan inteface yang terpasang.
```bash
[admin@MikroTik] > ip dhcp-client add interface=ether4 use-peer-dns=yes add-default-route=yes
[admin@MikroTik] > ip dhcp-client pr
Flags: X - disabled, I - invalid, D - dynamic
 #   INTERFACE                                              USE-PEER-DNS ADD-DEFAULT-ROUTE STATUS        ADDRESS
 0   ether4                                                 yes          yes               bound         192.168.255.150/24
```
Verifikasi IP Address, dns dan, big-zero
```bash
[admin@MikroTik] > ip address pr
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0 D 192.168.255.150/24 192.168.255.0   ether4
[admin@MikroTik] > ip dns pr
                      servers:
              dynamic-servers: 192.168.255.2
               use-doh-server:
              verify-doh-cert: no
        allow-remote-requests: yes
          max-udp-packet-size: 4096
         query-server-timeout: 2s
          query-total-timeout: 10s
       max-concurrent-queries: 100
  max-concurrent-tcp-sessions: 20
                   cache-size: 2048KiB
                cache-max-ttl: 1w
                   cache-used: 25KiB
[admin@MikroTik] > ip route pr
Flags: X - disabled, A - active, D - dynamic, C - connect, S - static, r - rip, b - bgp, o - ospf, m - mme,
B - blackhole, U - unreachable, P - prohibit
 #      DST-ADDRESS        PREF-SRC        GATEWAY            DISTANCE
 0 ADS  0.0.0.0/0                          192.168.255.2             1
 1 ADC  192.168.255.0/24   192.168.255.150 ether4                    0
```

### R-Customer-1
Konfigurasi IP Address
```bash
[admin@MikroTik] > ip address add address=192.168.35.1/24 interface=ether2
[admin@MikroTik] > ip address pr
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0   192.168.35.1/24    192.168.35.0    ether2
 1 D 10.10.10.64/32     10.10.10.1      ppoe-cliet-wates
```

### PC2
Konfigurasi IP Address
```bash
PC2> ip 192.168.35.2/24 192.168.35.1
Checking for duplicate address...
PC1 : 192.168.35.2 255.255.255.0 gateway 192.168.35.1

PC2> ip dns 10.10.10.1

PC2> sh ip

NAME        : PC2[1]
IP/MASK     : 192.168.35.2/24
GATEWAY     : 192.168.35.1
DNS         : 10.10.10.1
MAC         : 00:50:79:66:68:01
LPORT       : 10038
RHOST:PORT  : 127.0.0.1:10039
MTU:        : 1500
```

## Bagian Ketiga: Konfigurasi PPPoE Server
### R-ISP
Konfigurasi PPPoE Server
```bash
interface pppoe-server server add service-name=pppoe-server-wates interface=ether1 keepalive-timeout=900000
ip pool add name=pool-pppoe-wates ranges=10.10.10.2-10.10.10.64
ppp secret add name=nama-saya password=12345678 service=pppoe
ppp profile set numbers=0 local-address=10.10.10.1 remote-address=pool-pppoe-wates dns-server=10.10.10.1
```
Sharing internet
```bash
[admin@MikroTik] > ip firewall nat add chain=srcnat out-interface=ether4 action=masquerade
[admin@MikroTik] > ip firewall nat pr
Flags: X - disabled, I - invalid, D - dynamic
 0    chain=srcnat action=masquerade out-interface=ether4
```
## Bagian Keempat: Konfigurasi PPPoE Client
### Customer-1
Konfigurasi PPPoE Client
```bash
interface pppoe-client add name=pppoe-client-water user=nama-saya password=12345678 interface=ether1 use-peer-dns=yes add-default-route=yes
[admin@MikroTik] > interface pppoe-client monitor numbers=0
          status: connected
          uptime: 54m
    active-links: 1
        encoding:
    service-name: pppoe-server-wates
         ac-name: MikroTik
          ac-mac: 0C:26:EA:43:00:01
             mtu: 1480
             mru: 1480
   local-address: 10.10.10.64
  remote-address: 10.10.10.1
-- [Q quit|D dump|C-z pause]
```
Verifikasi IP Address, DNS dan, Big zero
```bash
[admin@MikroTik] > ip address pr
Flags: X - disabled, I - invalid, D - dynamic
 #   ADDRESS            NETWORK         INTERFACE
 0   192.168.35.1/24    192.168.35.0    ether2
 1 D 10.10.10.64/32     10.10.10.1      ppoe-cliet-wates
[admin@MikroTik] > ip dns pr
                      servers:
              dynamic-servers: 10.10.10.1
               use-doh-server:
              verify-doh-cert: no
        allow-remote-requests: no
          max-udp-packet-size: 4096
         query-server-timeout: 2s
          query-total-timeout: 10s
       max-concurrent-queries: 100
  max-concurrent-tcp-sessions: 20
                   cache-size: 2048KiB
                cache-max-ttl: 1w
                   cache-used: 25KiB
[admin@MikroTik] > ip route pr
Flags: X - disabled, A - active, D - dynamic, C - connect, S - static, r - rip, b - bgp, o - ospf, m - mme,
B - blackhole, U - unreachable, P - prohibit
 #      DST-ADDRESS        PREF-SRC        GATEWAY            DISTANCE
 0 ADS  0.0.0.0/0                          ppoe-cliet-wates          1
 1 ADC  10.10.10.1/32      10.10.10.64     ppoe-cliet-wates          0
 2 ADC  192.168.35.0/24    192.168.35.1    ether2                    0
```
## Bagian Kelima: Sharing internet
### Customer-1
```bash
[admin@MikroTik] > ip firewall nat add chain=srcnat out-interface=ppoe-cliet-wates action=masquerade
```
## Bagian Keenam: Pengujian
### Customer-1
```bash
[admin@MikroTik] > ping google.com
  SEQ HOST                                     SIZE TTL TIME  STATUS
    0 216.239.38.120                             56 127 83ms
    1 216.239.38.120                             56 127 43ms
    2 216.239.38.120                             56 127 48ms
    3 216.239.38.120                             56 127 30ms
    sent=4 received=4 packet-loss=0% min-rtt=30ms avg-rtt=51ms max-rtt=83ms
```
### PC2
```bash
PC2> ping google.com
google.com ->> forcesafesearch.google.com
forcesafesearch.google.com resolved to 216.239.38.120
84 bytes from 216.239.38.120 icmp_seq=1 ttl=126 time=36.585 ms
84 bytes from 216.239.38.120 icmp_seq=2 ttl=126 time=32.692 ms
84 bytes from 216.239.38.120 icmp_seq=3 ttl=126 time=40.286 ms
84 bytes from 216.239.38.120 icmp_seq=4 ttl=126 time=32.784 ms
84 bytes from 216.239.38.120 icmp_seq=5 ttl=126 time=59.466 ms
```

[def]: https://raw.githubusercontent.com/saifulindo/MTCNA/main/topologi-pptp.jpg