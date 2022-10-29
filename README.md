PRAKTIKUM JARKOM MODUL 2

- Andi Muhammad Rafli 5025201089
- Achmad Ferdiansyah 5025201245
- Adinda Zahra Pamuji 5025201175

## Soal

Twilight (〈黄昏(たそがれ) 〉, <Tasogare>) adalah seorang mata-mata yang berasal dari negara Westalis. Demi menjaga perdamaian antara Westalis dengan Ostania, Twilight
dengan nama samaran Loid Forger (ロイド・フォージャー, Roido Fōjā) di bawah organisasi WISE menjalankan operasinya di negara Ostania dengan cara melakukan spionase, sabotase, penyadapan dan kemungkinan pembunuhan. Berikut adalah peta dari negara Ostania:
  ![image](https://user-images.githubusercontent.com/89954689/198832604-172f603c-3b22-430e-8093-dd5d00145952.png)
  
WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden. Semua
node terhubung pada router Ostania, sehingga dapat mengakses internet (1). Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise (2). Setelah itu ia juga ingin membuat subdomain eden.wise.yyy.com dengan alias www.eden.wise.yyy.com yang diatur DNS-nya di WISE dan mengarah ke Eden (3). Buat juga reverse domain untuk domain utama (4). Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama (5). Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu operation.wise.yyy.com dengan alias www.operation.wise.yyy.com yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation (6). Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses strix.operation.wise.yyy.com dengan alias www.strix.operation.wise.yyy.com yang mengarah ke Eden (7). Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.wise.yyy.com. Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com (8). Setelah itu, Loid juga membutuhkan agar url www.wise.yyy.com/index.php/home dapat menjadi menjadi www.wise.yyy.com/home (9). Setelah itu, pada subdomain www.eden.wise.yyy.com, Loid membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/eden.wise.yyy.com (10). Akan tetapi, pada folder /public, Loid ingin hanya dapat melakukan directory listing saja (11). Tidak hanya itu, Loid juga ingin menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache (12). Loid juga meminta Franky untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset www.eden.wise.yyy.com/public/js menjadi www.eden.wise.yyy.com/js (13). Loid meminta agar www.strix.operation.wise.yyy.com hanya bisa diakses dengan port 15000 dan port 15500 (14) dengan autentikasi username Twilight dan password opStrix dan file di /var/www/strix.operation.wise.yyy (15) dan setiap kali mengakses IP Eden akan dialihkan secara otomatis ke www.wise.yyy.com (16). Karena website www.eden.wise.yyy.com semakin banyak pengunjung dan banyak modifikasi sehingga banyak gambar-gambar yang random, maka Loid ingin mengubah request gambar yang memiliki substring “eden” akan diarahkan menuju eden.png. Bantulah Agent Twilight dan Organisasi WISE menjaga perdamaian! (17)

## Nomor 1
  
Buatlah topologi seperti pada soal

Jika sudah ganti konfigurasi pada Ostania dengan

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.14.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.14.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 10.14.3.1
	netmask 255.255.255.0
```
Ganti konfigurasi pada Wise dengan
```
auto eth0
iface eth0 inet static
	address 10.14.3.2
	netmask 255.255.255.0
	gateway 10.14.3.1
```
Lalu, selain itu mengikuti ethnya masing masing.

Lalu, pada Ostania ketikkan ini pada terminal (sekalian simpan di file /root/.bashrc)

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.14.0.0/16
echo nameserver 192.168.122.1 > /etc/resolv.conf
```

Pada node Wise, SSS, Eden, Garden, Berlint
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```

## Nomor 2: Membuat Website utama pada WISE
## Setting WISE menjadi DNS Master

Jadikan WISE sebagai master dengan mengetik ini dulu

```
 apt-get update
 apt-get install bind9 -y
```

Konfigurasi file

```
echo 'zone "wise.c09.com" {
        type master;
        file "/etc/bind/wise/wise.c09.com";
};' > /etc/bind/named.conf.local
```

### Membuat website utama dengan akses wise.yyy.com

Pada "WISE" buat folder di dalam etc/bind

```
mkdir /etc/bind/jarkom-c09
```

Copy file db.local dan ganti nama

```
cp /etc/bind/db.local /etc/bind/jarkom-c09/wise.c09.com
```

Buka file /etc/bind/jarkom-c09/wise.c09.com dan tuliskan

```
echo "
;
; BIND data file for local loopback interface
;
\$TTL    604800
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
" > /etc/bind/wise/wise.c09.com
```

Lalu restart bind9

```
service bind9 restart
```

## Setting SSS dan Garden sebagai Client

Pada client SSS dan Garden arahkan nameserver menuju IP WISE dengan mengedit file resolv.conf dengan mengetikkan perintah

```
echo "
nameserver 10.14.3.2 # IP WISE
" > /etc/resolv.conf
```

Untuk mencoba koneksi DNS, lakukan ping domain wise.c09.com dengan melakukan perintah berikut pada client SSS dan Garden

```
ping wise.c09.com -c 3
```

## Nomor 3: Membuat Subdomain dengan mengatur DNS di WISE dan Mengarah ke Eden

### Membuat Subdomain "eden.wise.c09.com"

Buka file /etc/bind/jarkom-c09/wise.c09.com pada WISE

```
echo "
;
; BIND data file for local loopback interface
;
\$TTL    604800
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
" > /etc/bind/wise/wise.c09.com
```

Restart bind9

```
service bind9 restart
```

Coba ping ke subdomain dari salah satu client

```
ping eden.wise.c09.com -c 5
```
ATAU
```
host -t A eden.wise.c09.com
```

## Nomor 4: Buat Reverse Domain untuk Domain Utama

Lalu tambahkan konfigurasi berikut ke dalam file named.conf.local. Tambahkan reverse dari 3 byte awal dari IP yang ingin dilakukan Reverse DNS. Karena di contoh saya menggunakan IP WISE (10.14.3) untuk IP dari records, maka reversenya adalah 3.14.10

```
echo 'zone "3.14.10.in-addr.arpa" {
    type master;
    file "/etc/bind/wise/3.14.10.in-addr.arpa";
};' >> /etc/bind/named.conf.local
```

Copykan file db.local pada path /etc/bind ke dalam folder jarkom yang baru saja dibuat dan ubah namanya menjadi 3.14.10.in-addr.arpa

```
cp /etc/bind/db.local /etc/bind/jarkom-c09/3.14.10.in-addr.arpa
```

Buka dan edit file /etc/bind/jarkom-c09/3.14.10.in-addr.arpa menjadi seperti ini

```
echo "
;
; BIND data file for local loopback interface
;
\$TTL    604800
@       IN      SOA     wise.c09.com. root.wise.c09.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
3.14.10.in-addr.arpa.   IN      NS      wise.c09.com.
2                       IN      PTR     wise.c09.com. ; Byte ke-4 WISE
" > /etc/bind/wise/3.14.10.in-addr.arpa
```
  
Lalu restart bind9

```
service bind9 restart
```

Untuk mengecek Konfigurasi lakukan perintah pada salah satu client

```
// pada client SSS atau Garden

// Install package dnsutils
// Pastikan nameserver di /etc/resolv.conf telah dikembalikan sama dengan nameserver dari Ostania
apt-get update
apt-get install dnsutils

//Kembalikan nameserver agar tersambung dengan WISE
host -t PTR "IP WISE"
```

## Nomor 5: Membuat Berlint menjadi Slave DNS

## Setting "Berlint" menjadi DNS Slave

Buka file /etc/bind/named.conf.local pada WISE dan tuliskan

```
echo "
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

zone "3.14.10.in-addr.arpa" {
    type master;
    file "/etc/bind/wise/3.14.10.in-addr.arpa";
};
" > /etc/bind/named.conf.local
```

Restart pada bind9

```
service bind9 restart
```

Lalu konfigurasi node "Berlint"
Pertama update dan install bind9 pada node Berlint

```
apt-get update
apt-get install bind9
```

Lalu edit /etc/bind/named.conf.local pada Berlint dan tambahkan seperti berikut

```
echo '
zone "wise.c09.com" {
    type slave;
    masters { 10.14.3.2; }; // Masukan IP WISE
    file "/var/lib/bind/wise.c09.com";
};
' > /etc/bind/named.conf.local
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

## Info

yyy adalah kode kelompok, jadi yyy kita = c09
