# Jarkom_Modul3_Lapres_A06
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

- `bash topologi.sh`
- pada router surabaya bash `nano /etc/sysctl.conf` hapus # pada bagian `net.ipv4.ip_forward=1`
- `sysctl -p`

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
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.168.1.0/16
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


#### proxy - squid (mojokerto)
- `apt-get install squid`
- `service squid status` (cek status squid)
- `mv /etc/squid/squid.conf /etc/squid/squid.conf.bak`

#### 7.) User autentikasi memiliki format:
<br>User : userta_a06
<br>Password : inipassw0rdta_a06

- `apt-get install apache2-utils`
- `htpasswd -c /etc/squid/passwd userta_a06` (menambahkan user baru)
- pass: inipassw0rdta_a06
- `nano /etc/squid/squid.conf` (konfigurasi squid)

```
http_port 8080
visible_hostname mojokerto

auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Proxy
auth_param basic credentialsttl 2 hours
auth_param basic casesensitive on
acl USERS proxy_auth REQUIRED

http_access allow USERS
```
#### 8.) Pembatasan waktu akses hari Selasa-Rabu pukul 13.00-18.00.
- `nano /etc/squid/acl.conf`
```
acl JADWAL_TA time TW 13:00-18:00
```

#### 9.) Pembatasan waktu akses hari Selasa-Kamis pukul 21.00 - 09.00 keesokan harinya (sampai Jumat jam 09.00). 
```
acl JADWAL_BIMBINGAN1 time TWH 21:00-23:59
acl JADWAL_BIMBINGAN2 time WHF 00:00-09:00
```

#### 10.) setiap mengakses google.com, maka akan di redirect menuju monta.if.its.ac.id (revisi)

tambahkan:
```
acl BLOCKGOOGLE dstdomain .google.com
deny_info http://monta.if.its.ac.id/ BLOCKGOOGLE

http_access deny BLOCKGOOGLE

```

#### 11.) mengubah error page default squid
- `wget 10.151.36.202/ERR_ACCESS_DENIED`
- `cp -r ERR_ACCESS_DENIED /usr/share/squid/errors/*/ERR_ACCESS_DENIED`


sehingga file /etc/squid/acl.conf berisi:

![12](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/12.png)

dan file /etc/squid/squid.conf berisi: 

![13](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/13.png)

p.s setting http_access diminta untuk setiap jadwal yang sesuai diminta autentifikasi user, maka digunakan AND

#### 12.) domain janganlupa-ta.a06.pw dan memasukkan port 8080. 
pada dns server (malang)
- `apt-get update`
- `apt-get install bind9 -y`
- `nano /etc/bind/named.conf.local`
tambahkan:

```
zone "janganlupa-ta.a06.pw" {
	type master;
	file "/etc/bind/jarkom/janganlupa-ta.a06.pw";
};
```

![16](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/16.png)

- `mkdir /etc/bind/jarkom`
- `cp /etc/bind/db.local /etc/bind/jarkom/janganlupa-ta.a06.pw`
- `nano /etc/bind/jarkom/janganlupa-ta.a06.pw`

![19](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/19.png)

- `named -g`

#### testing proxy
- di firefox proxy setting:
<br>HTTP proxy 10.151.73.59 (ip MOJOKERTO)
<br>port: 8080

- akses its.ac.id

--- jika waktu akses memenuhi jadwal (JADWAL_TA / JADWAL_BIMBINGAN1 / JADWAL_BIMBINGAN2)
    akan diminta autentifikasi user

![14](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/14.png)

jika berhasil masuk dengan user yang telah didaftarkan, maka akan berhasil mengakses its.ac.id

![15](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/15.png)

--- jika waktu akses tidak sesuai jadwal akan menampilkan page error seperti yang diminta nomor 11<br>

--- jika akses google.com akan di redirect ke monta.if.ac.id<br>


#### Revisi dan Kendala Lain
- no. 10 waktu pengerjaan soal shift belum sama sekali. 
setelah revisi, google.com tidak bisa diakses, error page:

![17](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/17.png)

- no. 11 sebelumnya dan waktu demo bisa, namun saat hendak dicoba untuk testing screenshot lapres tidak bisa, error page: <br>

![18](https://github.com/asandfghjkl/Jarkom_Modul3_Lapres_A06/blob/main/pics/18.png)

- no. 12 domain belum berhasil digunakan, tidak bisa di ping. setelah revisi masih tidak bisa digunakan.

