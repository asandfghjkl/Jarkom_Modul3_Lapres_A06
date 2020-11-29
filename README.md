# Jarkom_Modul3_A06
Kelompok A06<br>
Muhammad Fikri Rabbani 	(05111840000165)<br>
Sandra Agnes Oktaviana 	(05111840000124)

#### 1.) membuat topologi jaringan
`nano topologi.sh`

![1](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/1.png)

```
# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &
uml_switch -unix switch3 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.72.29 eth1=daemon,,,switch1 eth2=daemon,,,switch3 eth3=daemon,,,switch2 mem=256M &

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=160M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
xterm -T TUBAN -e linux ubd0=TUBAN,jarkom umid=TUBAN eth0=daemon,,,switch2 mem=128M &

# Klien
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch1 mem=64M &
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch1 mem=64M &
xterm -T BANYUWANGI -e linux ubd0=BANYUWANGI,jarkom umid=BANYUWANGI eth0=daemon,,,switch3 mem=64M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch3 mem=64M &

```

konfigurasi interfaces pada setiap uml:

`nano /etc/network/interfaces`

- surabaya (router)

![2](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/2.png)

- mojokerto (proxy server)

![3](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/3.png)

- tuban (dhcp-server)

![4](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/4.png)

- malang (dns server)

![5](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/5.png)

- gresik, sidoarjo, banyuwangi, madiun (client server --- ip dari dhcp server)

![6](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/6.png)

![7](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/7.png)

![8](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/8.png)

![9](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/9.png)


lakukan:
```
iptables –t nat –A POSTROUTING –o eth0 –j MASQUERADE –s 192.168.0.0/16
```
pada router SURABAYA agar client bisa terhubung dengan internet.

#### 2.) dhcp-relay (surabaya)
- `apt-get update`
- `apt-get install isc-dhcp-relay`

<br> muncul window konfigurasi (biru-biru)<br>

- ip server: 10.151.73.60 (tuban)
- interfaces: eth1 eth2 eth3 (interface yang dipakai untuk menghubungkan ke router-dhcp relay surabaya)
- `/etc/init.d/isc-dhcp-relay restart` (restart dhcp relay)

#### konfigurasi dhcp server (tuban)
- `apt-get update`
- `apt-get install isc-dhcp-server`
- `nano /etc/default/isc-dhcp-server` ------------- set interface: eth0 (yang dipakai untuk menghubungkan ke switch2 yang selanutnya dihubungkan ke router/dhcp-relay surabaya)
- `nano /etc/dhcp/dhcpd.conf` --------------------- konfigurasi dhcp

#### 3.) Client pada subnet 1 mendapatkan range IP dari 192.168.0.10 sampai 192.168.0.100 dan 192.168.0.110 sampai 192.168.0.200.
#### 4) Client pada subnet 3 mendapatkan range IP dari 192.168.1.50 sampai 192.168.1.70.
#### 5.) Client mendapatkan DNS Malang dan DNS 202.46.129.2 dari DHCP
#### 6.) Client di subnet 1 mendapatkan peminjaman alamat IP selama 5 menit, sedangkan client pada subnet 3 mendapatkan peminjaman IP selama 10 menit.

```
#subnet 1
subnet 192.168.0.0 netmask 255.255.255.0{
    range 192.168.0.10 192.168.0.100;
    range 192.168.0.110 192.168.0.200;
    option routers 192.168.0.1;
    option broadcast-address 192.168.0.255;
    option domain-name-servers 10.151.73.58 202.46.129.2; 
    default-lease-time 300;
    max-lease-time 300;
}

#subnet 3
subnet 192.168.1.0 netmask 255.255.255.0{
    range 192.168.1.50 sampai 192.168.1.70;
    option routers 192.168.1.1;
    option broadcast-address 192.168.1.255;
    option domain-name-servers 10.151.73.58 202.46.129.2;
    default-lease-time 600;
    max-lease-time 600;
}

#subnet nid dmz
subnet 10.151.73.56 netmask 255.255.255.0 {
    
}
```
- #subnet 1 
- #subnet 3 
- #subnet 2 / nid dmz -------- perlu mengenali posisi dhcp server dimana, maka dilakukan register nid agar client tau letak dhcp-server tuban / subnet 2

![10](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/10.png)

- `service isc-dhcp-server restart` (pada tuban)

- selesai dilakukan konfigurasi interfaces pada client, lakukan `service networking restart` pada masing - masing client server

- testing dengan melakukan `ifconfig` pada client

![11](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/11.png)



