# Jarkom-Modul-5-B13-2022

### Kelompok B13
| **No** | **Nama** | **NRP** | 
| ------------- | ------------- | --------- |
| 1 | Amsal Herbert  | 5025201182 | 
| 2 | Elbert Dicky Aristyo | 5025201231 |
| 3 | Nathanael Roviery | 5025201258 |

## A
Tugas pertama kalian yaitu membuat topologi jaringan sesuai dengan rancangan yang diberikan Loid
![topologi](https://cdn.discordapp.com/attachments/818146232689098802/1050770244588810300/image.png)

Keterangan:
- Eden (DNS Server)
- WISE (DHCP Server)
- Garden & SSS (Web Server)
- Westalis, Strix, Ostania (DHCP Relay)
- Forger (62 Host)
- Desmond (700 Host)
- Briar (200 Host)
- Blackbell (255 Host)

## B
- Pembagian subnet menggunakan teknik VLSM
![subnet vlsm](https://cdn.discordapp.com/attachments/818146232689098802/1050770244588810300/image.png)

- Pembagian IP  
![Pembagian IP](https://cdn.discordapp.com/attachments/818146232689098802/1050772302113034320/image.png)
![Pembagian IP](https://cdn.discordapp.com/attachments/818146232689098802/1050772539971993621/image.png)

## C

### **Strix**
```
route add -net 192.179.0.0 netmask 255.255.255.248 gw 192.179.0.9
route add -net 192.179.0.128 netmask 255.255.255.128 gw 192.179.0.9
route add -net 192.179.4.0 netmask 255.255.252.0 gw 192.179.0.9
route add -net 192.179.2.0 netmask 255.255.254.0 gw 192.179.0.66
route add -net 192.179.1.0 netmask 255.255.255.0 gw 192.179.0.66
route add -net 192.179.0.80 netmask 255.255.255.248 gw 192.179.0.66
```

## D
> DHCP Server
- **WISE**
  
  ```
  echo "nameserver 192.179.122.1" > /etc/resolv.conf
  apt-get update
  apt-get install isc-dhcp-server -y
  ```
  isc-dhcp-server
  ```
  INTERFACES="eth0"
  ```

  dhcpd.conf
  ```
  # Forger
  subnet 192.179.0.128 netmask 255.255.255.128{
          range 192.179.0.130 192.179.0.254;
          option routers 192.179.0.129;
          option broadcast-address 192.179.0.255;
          option domain-name-servers 192.179.0.3;
          default-lease-time 600;
          max-lease-time 7200;
  }

  # Desmond
  subnet 192.179.4.0 netmask 255.255.252.0{
          range 192.179.4.2 192.179.7.254;
          option routers 192.179.4.1;
          option broadcast-address 192.179.7.255;
          option domain-name-servers 192.179.0.3;
          default-lease-time 600;
          max-lease-time 7200;
  }

  # Blackbell
  subnet 192.179.2.0 netmask 255.255.254.0{
          range 192.179.2.2 192.179.3.254;
          option routers 192.179.2.1;
          option broadcast-address 192.179.3.255;
          option domain-name-servers 192.179.0.3;
          default-lease-time 600;
          max-lease-time 7200;
  }

  # Briar
  subnet 192.179.1.0 netmask 255.255.255.0{
          range 192.179.1.2 192.179.1.254;
          option routers 192.179.1.1;
          option broadcast-address 192.179.1.255;
          option domain-name-servers 192.179.0.3;
          default-lease-time 600;
          max-lease-time 7200;
  }

  # Subnet DHCP Server
  subnet 192.179.0.0 netmask 255.255.255.248{
          option routers 192.179.0.1;
  }
  ```
> DHCP Relay
- **Westalis, Strix, Ostania**
  ```
  echo "nameserver 192.168.122.1" > /etc/resolv.conf
  apt-get update
  apt-get install isc-dhcp-relay -y
  ```
  isc-dhcp-relay
  ```
  SERVERS="192.179.0.2"
  INTERFACES="eth0 eth1 eth2 eth3"
  OPTIONS=""
  ```

> IP Dinamis (DHCP)
- **Forger, Desmond, Briar, Blackbell**  

  Network Configuration
  ```
  auto eth0
  iface eth0 inet dhcp
  ```

## Soal 1
Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Strix menggunakan iptables, tetapi Loid tidak ingin menggunakan MASQUERADE.

Penyelesaian:
- Strix
  ```
  iptables -t nat -A POSTROUTING -s 192.179.0.0/21 -o eth0 -j SNAT --to-source 192.168.122.155
  ```