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
  Start DHCP Server
  
  ```
  service isc-dhcp-server stop
  service isc-dhcp-server start
  service isc-dhcp-server status
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
  Start DHCP Relay

  ```
  service isc-dhcp-relay stop
  service isc-dhcp-relay start
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

## Soal 2
Kalian diminta untuk melakukan drop semua TCP dan UDP dari luar Topologi kalian pada server yang merupakan DHCP Server demi menjaga keamanan.

Penyelesaian:
- WISE
  ```
  iptables -A INPUT -p tcp --dport 80 -j DROP
  iptables -A INPUT -p udp --dport 80 -j DROP
  ```

- Testing
  - WISE
    ```
    apt-get install netcat -y
    nc -l -p 80
    ```
  - STRIX
    ```
    apt-get install netcat -y
    nc 192.179.0.2 80
    ```

  Hasil 

  ![Testing Soal 2](https://cdn.discordapp.com/attachments/818146232689098802/1050783882036912168/image.png)

## Soal 3
Loid meminta kalian untuk membatasi DHCP dan DNS Server hanya boleh menerima maksimal 2 koneksi ICMP secara bersamaan menggunakan iptables, selebihnya didrop.

Penyelesaian:
- WISE
  ```
  iptables -A INPUT -p icmp -m connlimit --connlimit-above 2 --connlimit-mask 0 -j DROP
  ```
- Eden
  ```
  iptables -A INPUT -p icmp -m connlimit --connlimit-above 2 --connlimit-mask 0 -j DROP
  ```

## Soal 4
Akses menuju Web Server hanya diperbolehkan disaat jam kerja yaitu Senin sampai Jumat pada pukul 07.00 - 16.00.

Penyelesaian:
- Garden & SSS
  ```
  iptables -A INPUT -d 192.179.0.80/29 -m time --timestart 07:00 --timestop 16:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT
  iptables -A INPUT -d 192.179.0.80/29 -j REJECT
  ```

  Hasil
  ![Testing Soal 4](https://cdn.discordapp.com/attachments/818146232689098802/1050788702131081331/image.png)
  ![Testing Soal 4](https://cdn.discordapp.com/attachments/818146232689098802/1050789293792182272/image.png)

## Soal 5
Karena kita memiliki 2 Web Server, Loid ingin Ostania diatur sehingga setiap request dari client yang mengakses Garden dengan port 80 akan didistribusikan secara bergantian pada SSS dan Garden secara berurutan dan request dari client yang mengakses SSS dengan port 443 akan didistribusikan secara bergantian pada Garden dan SSS secara berurutan.

Penyelesaian:
- Ostania
  ```
  iptables -A PREROUTING -t nat -p tcp -d 192.179.0.82 --dport 80 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.179.0.82
  iptables -A PREROUTING -t nat -p tcp -d 192.179.0.82 --dport 80 -j DNAT --to-destination 192.179.0.83
  iptables -A PREROUTING -t nat -p tcp -d 192.179.0.83 --dport 443 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.179.0.82
  iptables -A PREROUTING -t nat -p tcp -d 192.179.0.83 --dport 443 -j DNAT --to-destination 192.179.0.83

  iptables -t nat -A POSTROUTING -p tcp -d 192.179.0.82 -j SNAT --to-source 192.179.0.0
  iptables -t nat -A POSTROUTING -p tcp -d 192.179.0.83 -j SNAT --to-source 192.179.0.0
  ```
