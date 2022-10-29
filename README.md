<p align="center">
  <a href="" rel="noopener">
 <img width=200px height=200px src="https://i.imgur.com/6wj0hh6.jpg" alt="Project logo"></a>
</p>

<h3 align="center">Project Title</h3>

<div align="center">

[![Status](https://img.shields.io/badge/status-active-success.svg)]()
[![GitHub Issues](https://img.shields.io/github/issues/kylelobo/The-Documentation-Compendium.svg)](https://github.com/kylelobo/The-Documentation-Compendium/issues)
[![GitHub Pull Requests](https://img.shields.io/github/issues-pr/kylelobo/The-Documentation-Compendium.svg)](https://github.com/kylelobo/The-Documentation-Compendium/pulls)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](/LICENSE)

</div>

---

PRAKTIKUM JARKOM MODUL 2

- Andi Muhammad Rafli 5025201089
- Achmad Ferdiansyah
- Adinda Zahra Pamuji

## Nomor 1

ikuti modul GNS3.

Jika sudah ganti nama tiap node dan konfigurasi network WISE

```
auto eth0
iface eth0 inet static
	address [Prefix IP].3.2
	netmask 255.255.255.0
	gateway [Prefix IP].3.1
```

lalu pada Ostania ketikkan ini pada terminal (sekalian simpan di file /root/.bashrc)

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s [Prefix IP].0.0/16

dan

echo nameserver [IP DNS] > /etc/resolv.conf
```

Jangan lupa pada node lain ketik dan simpan di /root/.bashrc

```
echo nameserver [IP DNS] > /etc/resolv.conf
```

## Setting WISE menjadi DNS Master

Jadikan WISE sebagai master dengan mengetik ini dulu

```
 apt-get update
 dan
 apt-get install bind9 -y
```

Lalu buka file pada WISE

```
nano /etc/bind/named.conf.local
```

dan configurasi file dengan mengetik

```
zone "wise.c09.com" {
	type master;
	file "/etc/bind/wise/wise.c09.com";
};
```

### Membuat website utama dengan akses wise.yyy.com (no. 2)

Pada "WISE" buat folder di dalam etc/bind

```
mkdir /etc/bind/wise
```

copy file db.local dan ganti nama

```
cp /etc/bind/db.local /etc/bind/wise/wise.c09.com
```

Buka file yang telah dibuat

```
nano /etc/bind/wise/wise.c09.com
```

edit filenya seperti ini dan save

```

;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.c09.com. root.wise.c09.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      wise.c09.com.
@       IN      A       10.14.3.2       ; IP WISE masing-masing
@       IN      AAAA    ::1
```

lalu restart bind9

```
service bind9 restart
```

## Setting SSS dan Garden sebagai Client

Pada client SSS dan Garden arahkan nameserver menuju IP WISE dengan mengedit file resolv.conf dengan mengetikkan perintah

```
 nano /etc/resolv.conf

 lalu tambahkan

 nameserver 10.14.3.2 # IP WISE
```

Untuk mencoba koneksi DNS, lakukan ping domain wise.c09.com dengan melakukan perintah berikut pada client SSS dan Garden

```
ping wise.c09.com -c 3
```

## Setting "Berlint" menjadi DNS Slave (no. 5)

Buka WISE dan edit file /etc/bind/named.conf.local menjadi seperti ini

```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "wise.c09.com" {
        type master;
        notify yes;
        also-notify { 10.14.2.2; }; // Masukan IP Berlint
        allow-transfer { 10.14.2.2; }; // Masukan IP Berlint
        file "/etc/bind/wise/wise.c09.com";
};
```

restart pada bind9

```
servive bind9 restart
```

Lalu konfigurasi node "Berlint"
Pertama update dan install bind9 pada node Berlint

```
apt-get update
apt-get install bind9 -y
```

Lalu edit /etc/bind/named.conf.local pada Berlint dan tambahkan seperti berikut

```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
// include "/etc/bind/zones.rfc1918";

zone "wise.c09.com" {
    type slave;
    masters { 10.14.3.2; }; // Masukan IP WISE
    file "/var/lib/bind/wise.c09.com";
};
```

Lalu restart service bind9 Berlint

```
service bind9 restart
```

### Testing Koneksi Setelah Set DNS

untuk testing coba matikan service bind9 pada "WISE"

```
service bind9 stop
```

lalu coba ping wise.c09.com dari salah satu client

## Nomor 2: Membuat Website utama pada WISE

### alias: www.wise.yyy.com

tambahkan alias "www.wise.c09.com" pada file /etc/bind/wise/wise.c09.com

```
nano /etc/bind/wise/wise.c09.com
```

lalu edit file seperti dibawah ini

```

;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.c09.com. root.wise.c09.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      wise.c09.com.
@       IN      A       10.14.3.2       ; IP WISE masing-masing
www     IN      CNAME   wise.c09.com.   ; Alias
@       IN      AAAA    ::1
```

## Nomor 3: Membuat Subdomain dengan mengatur DNS di WISE dan Mengarah ke Eden

### Membuat Subdomain "eden.wise.c09.com" dan alias "www.eden.wise.c09.com"

Buka file /etc/bind/wise/wise.c09.com pada WISE

```
nano /etc/bind/wise/wise.c09.com
```

Lalu tambahkan subdomain untuk wise.c09.com dan aliasnya lalu arahkan ke IP address Eden

```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.c09.com. root.wise.c09.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@            IN      NS      wise.c09.com.
@            IN      A       10.14.3.2             ; IP WISE masing-masing
www          IN      CNAME   wise.c09.com.         ; Alias
eden         IN      A       10.14.2.3             ; IP Eden
www.eden     IN      CNAME   eden.wise.c09.com.    ; Alias
@            IN      AAAA    ::1
```

Restart bind9

```
service bind9 restart
```

Coba ping ke subdomain dari salah satu client

```
ping eden.wise.c09.com -c 5

ATAU

host -t A eden.wise.c09.com
```

## Nomor 4: Buat Reverse Domain untuk Domain Utama

Lalu tambahkan konfigurasi berikut ke dalam file named.conf.local di WISE. Tambahkan reverse dari 3 byte awal dari IP yang ingin dilakukan Reverse DNS. Karena di contoh saya menggunakan IP WISE (10.14.3) untuk IP dari records, maka reversenya adalah 3.14.10

```
zone "3.14.10.in-addr.arpa" {
    type master;
    file "/etc/bind/wise/3.14.10.in-addr.arpa";
};
```

Copykan file db.local pada path /etc/bind ke dalam folder jarkom yang baru saja dibuat dan ubah namanya menjadi 3.14.10.in-addr.arpa

```
cp /etc/bind/db.local /etc/bind/wise/3.14.10.in-addr.arpa
```

Buka dan edit file /etc/bind/wise/3.14.10.in-addr.arpa menjadi seperti ini

```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.c09.com. root.wise.c09.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
3.14.10.in-addr.arpa.   IN      NS      wise.c09.com.
2                       IN      PTR     wise.c09.com. ; Byte ke-4 WISE
```

Lalu restart bind9

```
service bind9 restart
```

Untuk mengecek Konfigurasi lakukan perintah pada salah satu client

```
// pada client SSS atau Garden

// Install package dnsutils dan lynx
// Pastikan nameserver di /etc/resolv.conf telah dikembalikan sama dengan nameserver dari Ostania
apt-get update
apt-get install dnsutils -y
apt-get install lynx -y

//Kembalikan nameserver agar tersambung dengan WISE
host -t PTR "IP WISE"
```

## Nomor 5: Membuat Berlint menjadi Slave DNS

cek keatas bagian setting DNS

## Nomor 6: Buat Subdomain yang Khusus untuk Operation yaitu "operation.wise.yyy.com" dengan Alias "www.operation.wise.yyy.com" yang Didelegasikan dari WISE ke Berlint dengan IP Menuju ke Eden Dalam Folder operation

Pada "Slave" buka

```
nano /etc/bind/named.conf.options
```

lalu edit filenya seperti ini dan simpan

```
options {
        directory \"/var/cache/bind\";
        allow-query{any;};
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

lalu pada file /etc/bind/named.conf.local ubah isinya menjadi

```
zone "wise.c09.com" {
    type slave;
    masters { 10.14.3.2; }; // Masukan IP WISE
    file "/var/lib/bind/wise.c09.com";
};

zone "operation.wise.c09.com"{
        type master;
        file "/etc/bind/operation/operation.wise.c09.com";
};
```

Lalu pada Berlint buat folder operation

```
mkdir /etc/bind/operation
```

dan ubah dan simpan isinya menjadi seperti ini

```
;
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA     operation.wise.c09.com. root.operation.wise.c09.com. (
                                2         ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
;
@               IN      NS              operation.wise.c09.com.
@               IN      A               10.14.2.3               ;ip Eden
www             IN      CNAME           operation.wise.c09.com.
```

lalu restart bind9

```
service bind9 restart
```

# nomor 7: buatlah subdomain melalui Berlint dengan akses strix.operation.wise.yyy.com dengan alias www.strix.operation.wise.yyy.com yang mengarah ke Eden

Pada node Berlint buka file /etc/bind/operation/operation.wise.c09.com dan edit seperti berikut

```
;
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA     operation.wise.c09.com. root.operation.wise.c09.com. (
                                2         ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
;
@               IN      NS              operation.wise.c09.com.
@               IN      A               10.14.2.3               ;ip Eden
www             IN      CNAME           operation.wise.c09.com.
strix           IN      A               10.14.2.3               ;IP Eden
www.strix       IN      CNAME           strix.operation.wise.c09.com.
```

lalu restart bind9

```
service bind9 restart
```

# nomor 8: Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.wise.yyy.com. Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com

Pertama pada Eden install environtment yang dibutuhkan

```
apt-get update
apt-get install apache2 -y

service apache2 start

apt-get install php -y
apt-get install libapache2-mod-php7.0 -y

service apache2

apt-get install ca-certificates openssl -y
apt-get install unzip -y
apt-get install git -y
```

lalu clone file dari github

```

```

## Info

yyy adalah kode kelompok, jadi yyy kita = c09


