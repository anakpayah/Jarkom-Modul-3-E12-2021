# Jarkom-Modul-3-E12-2021

<hr/>

#### Anggota Kelompok :
 * Rahadian Adjie Mahesa &nbsp;(05111940000221)
 * Ahmad Luthfi Hanif &nbsp; (05111940000179)
 * Afifan Syafaqi Yahya &nbsp; (05111940000234)  

#### Prefix IP 
Address untuk Prefix IP Kelompok kami adalah `10.35`
  
<hr/>
  
## Topologi dan Persiapan

Membuat topologi sebagai berikut:  
![Topology](Image/TopologyPraktikum3.jpg)  
  
Mengatur _Network Configuration_ pada router `Foosha`, serta node `EniesLobby`. `Water7` dan `Jipangu`. pengaturan config nya sebagai berikut:  
  
1. Foosha  
```
 auto eth0
   iface eth0 inet dhcp

   auto eth1
   iface eth1 inet static
      address 10.35.1.1
      netmask 255.255.255.0

   auto eth2
   iface eth2 inet static
      address 10.35.2.1
      netmask 255.255.255.0

   auto eth3
   iface eth3 inet static
      address 10.35.3.1
      netmask 255.255.255.0
```  
    
2. EniesLobby  
```
   auto eth0
   iface eth0 inet static
      address 10.35.2.2
      netmask 255.255.255.0
      gateway 10.35.2.1
 ```  
   
3. Water7
```
   auto eth0
   iface eth0 inet static
      address 10.35.2.3
      netmask 255.255.255.0
      gateway 10.35.2.1
 ```  
   
4. Jipangu
```
   auto eth0
   iface eth0 inet static
      address 10.35.2.4
      netmask 255.255.255.0
      gateway 10.35.2.1
```  
  
Setelah semua node sudah desettig, lakukan command dibawah ini pada **Foosha** agar dapat mengakses internet.
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.35.0.0/16
```  
Kemudian, jalankan command pada node **EniesLobby**, **Water7**, dan **Jipangu**

```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```  
setalah itu dapat melakukan update pada node tersebut atau intallasi package lainnya karena sudah terakses oleh internet.  
  
## Soal 1  
  
Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server.  
  
**Pembahasan:**  
1. Intalasi ISC DHCP Server pada node **Jipangu** menggunakan command berikut:  
```
apt-get update
apt-get intall isc-dhcp-server -y
```  
Setlah itu, pastikan ISC DHCP server sudah teristall dengan menggunakan command `dhcpd --version`.  
  
2. Konfigurasi DHCP server pada **Jipangu** dengan meng-edit file konfigurasi _isc-dhcp-server_.
```
nano /etc/default/isc-dhcp-server
```  
Lalu, edit beberapa hal pada file tersebut. Pada interface yang diberikan layanan DHCP adalah **eth0**. Maka, pada bagian baris akhir di file `/etc/default/isc-dhcp-server` diubah menjadi
```
INTERFACES = "eth0"
```  
  
3. Konfigurasi Proxy Server pada node **Water7** yaitu install dan update terlebih dahulu package yang akan kita gunakan menggunakan command berikut:  
```
apt-get update
apt-get install squid -y
```  
untuk memastikan proxy server sudah ter-install dengan mengecek status squid tersebut apakah sudah _running_ menggunakan command 
```
service squid status
```  
Backup file konfigurasi default bawaan yang telah disediakan Squid dengan command:  
```
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
```  
Lalu, buat konfigurasi squid baru pada file tersebut.
```
nano /etc/squid/squid.conf
```  
Kemudian tambahkan
```
http_port 8080
visible_hostname Water7
```  
Setalah  itu, restart Suid tersebut menggunakan command `service squid restart` dan pastikan status Squid telah **OK**.  
  
## Soal 2
  
Foosha sebagai DHCP Relay  
  
**Pembahasan:**  
  
1. Instalasi ISC-DHCP-Relay pad router **Foosha** dengan melakukan commadn sebagai berikut:  
```
apt-get update
apt-get install isc-dhcp-relay
```  
Saat intalasi berlangsung, kita diminta untuk input `SERVER` yang akan kita isin dengan IP dari **Jipnagu**. ketik sebagai berikut:  
```
10.35.2.4
```  
Kemudain, diminta lagi unruk menginputkan `INTERFECES`, ketik sebagai berikut:  
```
eth1 eth2 eth3
```  
Setelah itu, bila ada hal yang diminta input lagi, lewati saja dengan klik tombol ENTER pada keyboard.  
**ATAU**  
Dapat diubah juga dengan mengedit file konfogurasinya _/etc/default/isc-dhcp-relay_. Dengan isi yang dapat diubah pada bagian `SERVERS` dan `INTERFACES` sebagaimana berikut:  
```
      # Defaults for isc-dhcp-relay initscript
      # sourced by /etc/init.d/isc-dhcp-relay
      # installed at /etc/default/isc-dhcp-relay by the maintainer scripts

      #
      # This is a POSIX shell fragment
      #

      # What servers should the DHCP relay forward requests to?
      SERVERS="10.35.2.4"

      # On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
      INTERFACES="eth1 eth2 eth3"

      # Additional options that are passed to the DHCP relay daemon?
      OPTIONS=""
```  
  
## Soal 3-6  
  
- **No 3**
  Client yang melalui Switch1 mendapatkan range IP dari 10.36.1.20 - 10.36.1.99 dan 10.36.1.150 - 10.36.1.169  
- **No 4**
  Client yang melalui Switch3 mendapatkan range IP dari 10.36.3.30 - 10.36.3.50  
- **No 5** Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.  
- **No 6** Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.  
  
**Pembahasan:**  

1. install `bind9` pada **EniesLobby** dan jadikan sebagai DNS Forwarders dengn commdan sebagai berikut:  
```
apt-get update
apt-get install bind9 -y
```  
Lalu, Edit file _/etc/bind/named.conf.options_ pada server **EniesLobby** dengan uncomment pada:  
```
forwarders {
  192.168.122.1;
};
```  
Dan edit dan tambah sebagai berikut:  
```
//dnssec-validation auto;
allow-query{any;};
```  
  
2. Setting DHCP Server pada **Jipangu** dengan mengedit isi file pada _/etc/dhcp/dhcpd.conf_ menggunakan command `nano /etc/dhcp/dhcpd.conf`. lalu tambahkan script sebagai berikut:  
```
subnet 10.36.2.0 netmask 255.255.255.0{
}

subnet 10.36.1.0 netmask 255.255.255.0{
         range 10.36.1.20 10.36.1.99;
         range 10.36.1.150 10.36.1.169;
         option routers 10.36.1.1;
         option broadcast-address 10.36.1.255;
         option domain-name-servers 10.36.2.2;
         default-lease-time 360;
         max-lease-time 7200;
}

subnet 10.36.3.0 netmask 255.255.255.0{
         range 10.36.3.30 10.36.3.50;
         option routers 10.36.3.1;
         option broadcast-address 10.36.3.255;
         option domain-name-servers 10.36.2.2;
         default-lease-time 720;
         max-lease-time 7200;
}
```  
  
**Keterangan :** <br>
a. **Jawaban dari No 3** (Switch 1)
```
     range 10.36.1.20 10.36.1.99;
     range 10.36.1.150 10.36.1.169;
```  
b. **Jawaban dari No 4** (Switch 3)

```
     range 10.36.3.30 10.36.3.50;
```  
c. **Jawaban dari No 6**  
- Switch 1
 ```
       default-lease-time 360;
       max-lease-time 7200;
```  
- Switch 3
```
        default-lease-time 720;
        max-lease-time 7200;
```  
  
Setelah semua sudah teredit, laukan restart DHCP server menggunakan command:  
```
service isc-dhcp-server restart
```  
  
3. Testing pada node Client (**Loguetown**, **Alabasta**, **TottoLand**, dan **Skypie**) dengan mengedit _Network Configuration_ pada tiap node client menjadi sebagai berikut:  
```
auto eth0
iface eth0 inet dhcp
```  
Kemudian, restart setiap client dengan cara buka GNS3 → klik kanan node → klik Stop → klik kanan kembali node → klik Start. Lalu periksa IP pada setiap node dengan command:  
```
ip a
```  
  
a. Loguetown  
![test_loguetown](Image/no5_test_Loguetown.png)  
b. Alabasta  
![test_alabasta](Image/no5_test_Alabasta.png)  
c. TottoLand  
![test_tottoland](Image/no5_test_TottoLand.png)  
d. Skypie  
![test_skypie](Image/no5_test_Skypie.png)  
  
  
## Soal 7

Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69. Dalam kasus ini, IP yang digunakan adalah 10.35.3.69.

**Pembahasan:**

1. Lakukan ```ip a``` pada client Skypie untuk melakukan pengecekkan link/ether pada eth0 Skypie. Pada kasus ini, link/ether pada eth0 client Skypie adalah sebagai berikut.
![link_ether_skypie](https://github.com/anakpayah/Jarkom-Modul-3-E12-2021/blob/main/Image/ip_a_skypie_1.JPG)

2. Buka client Skypie dan lakukan konfigurasi sebagai berikut pada file ```/etc/network/interfaces```.
```
auto eth0
iface eth0 inet dhcp
hwaddress ether f6:9a:aa:fb:c7:78
```

3. Buka Jipangu lalu edit file ```/etc/dhcp/dhcpd.conf``` dan tambahkan konfigurasi sebagai berikut.
```
host Skypie {
    hardware ethernet f6:9a:aa:fb:c7:78;
    fixed-address 10.35.3.69;
}
```
hardware ethernet yang digunakan merupakan link/ether dari eth0 Skypie, lalu fixed-address menyesuaikan dengan instruksi yang diberikan dari soal. Setelah melakukan konfigurasi, restart dhcp server dengan perintah ```service isc-dhcp-server restart```.

4. Lakukan pengecekkan dengan menggunakan ```ip a```. Apabila ip inet pada eth0 Skypie telah berubah menjadi ```10.35.3.69```, maka Luffy dan Zoro telah berhasil menjadikan Skypie sebagai server untuk jual beli kapal.
![ip_skypie](https://github.com/anakpayah/Jarkom-Modul-3-E12-2021/blob/main/Image/ip_a_skypie_2.JPG)

## Soal 8

Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi. Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000.

**Pembahasan:**
1. Langkah pertama, kita akan menggunakan node `EnniesLobby` untuk membuat **DNS** untuk `jualbelikapal.e12.com` dimana kita akan merubah configurasi pada `/etc/bind/named.conf.local` seperti berikut 
![image](https://user-images.githubusercontent.com/55140514/141610177-484d0e0f-1766-4ae6-90bc-78bd3eab2963.png)

2. Lalu, kita membuat *directory* sesuai dengan yang kita buat di konfigurasi `/etc/bind/named.conf.local` dengan `mkdir` dan salin *template* **db.local** dari directory bind dan diubah sesuai dengan soal seperti berikut
![image](https://user-images.githubusercontent.com/55140514/141610291-e55bdd44-2b05-42bd-8e67-7776588a5a75.png)

Setelah itu dilakukan `restart bind9`

3. Selanjutnya, kita menggunakan node `Water7` sebagai proxy server untuk melakukan konfigurasi proxy. Pada awalnya kita akan menyimpan konfigurasi awal `squid.conf` ke `squid.conf.bak` sebagai backup. Lalu, dirubah agar menggunakan `port 5000` dan merubah nama yang mengakses menjadi `jualbelikapal.e12.com` seperti berikut
![image](https://user-images.githubusercontent.com/55140514/141610529-eabf1787-a11b-452b-9b84-b7b03a672008.png)

4. Untuk melakukan testing kita akan pindah ke *proxy client* dimana pada soal diminta adalah `Loguetown`. Disini kita akan menginstall *lynx* dengan 
`apt-get install lynx -y` 
Setelah itu kita memasangkan proxy pada client ini dengan mengarah ke proxy yang telah kita buat yaitu `jualbelikapal.e12.com` dan menggunakan perintah berikut
`export http_proxy="http://jualbelikapal.e12.com:5000"`

5. Terakhir kita lakukan perintah `lynx google.com` untuk mengakses google.com. Hasilnya adalah berikut
![image](https://user-images.githubusercontent.com/55140514/141610697-a3bb933e-c92c-4552-9104-6da969f946ed.png)

Bisa dilihat bahwa nama pengakses berubah menjadi `jualbelikapal.e12.com` pada di bawah

### Note
1. Karena kita tidak menggunakan perintah untuk mengakses web maka kita tidak bisa masuk ke `google.com`. Untuk itu kita merubahkan `squid.conf` dengan menambah line `http_access allow all`

## Soal 9

Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy dipasang autentikasi user proxy dengan enkripsi MD5 dengan dua username, yaitu luffybelikapalyyy dengan password luffy_yyy dan zorobelikapalyyy dengan password zoro_yyy.

**Pembahasan:**
1. Langkah pertama membuat autentikasi adalah menggunakan fungsi `htpasswd`. Untuk soal ini ada dua user yaitu **luffybelikapalE12** dan **zorobelikapalE12** sebagai berikut
```
  htpasswd -c /etc/squid/passwd luffybelikapalE12
  password : luffy_e12

  htpasswd /etc/squid/passwd zorobelikapalE12
  password : zoro_e12
```
2. Lalu, kita implementasikan user tersebut dengan mengubah konfigurasi pada `squid.conf` menjadi berikut
![image](https://user-images.githubusercontent.com/55140514/141611100-93b5ebda-d0fe-43ab-a281-6826e357aa46.png)

   Setelah itu dilakukan `service squid restart`
3. Terakhir kita testing dengan `lynx google.com` lagi. Hasilnya akan menjadi berikut
![image](https://user-images.githubusercontent.com/55140514/141611187-53b96d81-9a21-4e78-8d7b-c2f7e327fac3.png)
![image](https://user-images.githubusercontent.com/55140514/141611194-013ddb69-8f89-4867-931e-1c18ab6f2030.png)

   Untuk mengakses hanya bisa jika telah memasukkan `user` dan `password` yang sudah di implementasikan

## Soal 10

Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari Senin-Kamis pukul 07.00-11.00 dan setiap hari Selasa-Jum’at pukul 17.00-03.00 keesokan harinya (sampai Sabtu pukul 03.00).

**Pembahasan:**
1. Pada soal ini diminta untuk mengubah waktu akses / pembatasan waktu akses ke web yg dituju. Untuk melakukan itu maka kita akan merubahkan `squid.conf`. Pertama kita membuat sebuah file yg akan berisi fungsi `acl` dengan perintah `time` untuk mengatur waktunya sesuai dengan yang diinginkan soal. Dilakukan `nano /etc/squid/acl.conf` dengan isinya sebagai berikut
```
acl AVAILABLE_WORKING1 time MTWH 07:00-11:00
acl AVAILABLE_WORKING2 time TWHF 17:00-24:00
acl AVAILABLE_WORKING3 time WHFA 00:00-03:00
```
2. Setelah itu akan kita masukkan file `acl.conf` tersebut ke dalam `squid.conf` dengan menambah berikut
```
include /etc/squid/acl.conf
http_access allow AVAILABLE_WORKING1 USERS
http_access allow AVAILABLE_WORKING2 USERS
http_access allow AVAILABLE_WORKING3 USERS
http_access deny all
```
   Menjadi seperti berikut
   
![image](https://user-images.githubusercontent.com/55140514/141611675-99373b2a-df22-42f0-8e7e-3dd8b42e1fa8.png)
    
   Disini kita mengganti `http_access allow all` dan diganti menjadi `http_acces AVAILABLE_WORKINGx` untuk tiap grup waktu serta menambah `http_access deny all` agar hanya yang mempunyai **user** atau yang termasuk dari grup **acl USERS** saja yang bisa mengakses. Setelah itu dilakukan `service squid restart`.
   
3. Dilakukan testing dengan mengatur **date** pada client dengan 2 waktu yang berbeda. Satu untuk waktu yang bisa diberi akses dan satu lagi untuk diluar waktu akses. Waktu tersebut adalah Jum'at, 11 NOV 2021 11:00 dan Jum'at, 11 NOV 18:00
```
date --set "11 Nov 2021 11:00:00"
date --set "11 Nov 2021 18:00:00"
```
**Diluar range waktu akses**

![image](https://user-images.githubusercontent.com/55140514/141612002-913b33e6-a06a-42ec-8c88-cede26f4aabb.png)

![image](https://user-images.githubusercontent.com/55140514/141612023-7d0479b9-549a-4299-9bfa-906845535f95.png)
![image](https://user-images.githubusercontent.com/55140514/141612032-2bfff301-be0b-49e0-8b4a-175456fb5205.png)

**Didalam range waktu akses**

![image](https://user-images.githubusercontent.com/55140514/141612061-d0957eb4-fbe3-4a56-97c4-90e017f117fa.png)
![image](https://user-images.githubusercontent.com/55140514/141612083-9143a595-035c-4632-a657-be24abfd156b.png)
![image](https://user-images.githubusercontent.com/55140514/141612093-c45ec6a4-2899-47f6-b96d-7d8fa5793fbd.png)

## Soal 11

Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap mengakses google.com, akan diredirect menuju super.franky.yyy.com dengan website yang sama pada soal shift modul 2. Web server super.franky.yyy.com berada pada node Skypie.

**Pembahasan:**
1. Langkah pertama yang kita lakukan adalah untuk membuat **DNS** `super.franky.e12.com` seperti pada nomor 8. pertama mengubah `/etc/bind/named.conf.local` menjadi berikut
![image](https://user-images.githubusercontent.com/55140514/141612758-0c6cf7d9-01df-428a-875d-f09604792554.png)

2. Lalu, membuat directory `sunnygo` sesuai yang di konfigurasi dan membuat file untuk `super.franky.e12.com` di directory tersebut dengan konfigurasi berikut
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     super.franky.e12.com. root.super.franky.e12.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      super.franky.e12.com.
@       IN      A       10.35.3.69 ; IP Skypie
www     IN      CNAME   super.franky.e12.com.
```
Setelah itu kita `service bind9 restart`

3. Lalu, kita ke node `skypie` dan menginstall `apache2`, `wget`, dan `unzip`. Pertama yang kita lakukan adalah mengambil file yang diminta soal dan melakukan unzip pada file tersebut
```
wget https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip
unzip super.franky.zip
```

4. Setelah itu kita membuat file konfigurasi `super.franky.e12.com.conf` untuk web server kita di `etc/apache2/sites-available`. Pertama kita *copy* file `000.default.conf` ke file `super.franky.e12.com.conf`
```
cp -r /etc/apache2/sites-available/000.default.conf /etc/apache2/sites-available/super.franky.e12.com.conf
```
5. Lalu, kita masuk ke file `super.franky.e12.com.conf` dan mengubah konfigurasi `DocumentRoot`, `ServerName`, dan `ServerAlias` seperti berikut
 ![image](https://user-images.githubusercontent.com/55140514/141613479-b634f83c-ddf7-43fb-8b81-d2909abff4e9.png)

6. Selanjutnya kita membuat directory untuk isi web kita seperti berikut 
`mkdir /var/www/super.franky.e12.com`

7. Lalu, hasil *unzip* kita dipindah kan ke dalam directory tersebut 
`mv super.franky/error super.franky/public /var/www/super.franky.e12.com`

![image](https://user-images.githubusercontent.com/55140514/141613697-5ad055ea-4fbd-42df-b7d4-c239c34fa1f7.png)

8. Terakhir untuk web server nya adalah untuk melakukan `service apache2 restart` serta mengaktifkan website dengan `a2ensite super.franky.e12.com.conf`

9. Pindah ke `Proxy Server (Water7)` untuk melakukan *redirect* dari `google.com` ke `super.franky.e12.com`. Disini kita akan mengubah konfigurasi `squid.conf` dengan menambahkan line-line berikut
```
acl BLACKLIST dstdomain google.com
http_access deny BLACKLIST
deny_info http://super.franky.e12.com/ BLACKLIST
```
Disini kita membuat acl grup bernama `BLACKLIST` yang berisikan domain name yang diinginkan. Lalu, domain yang ada di grup BLACKLIST ini akan diblokir / deny access nya dan return ke http://super.franky.e12.com/. Setelah itu, dilakukan restart pada Proxy Server agar jalan redirect-nya.

10. Untuk testing kita pindah ke `Loguetown` dan mencoba `lynx google.com`. Hasil sebagai berikut
![image](https://user-images.githubusercontent.com/55140514/141613947-e6a2ea22-db92-4425-bd42-a728cd99d549.png)

![image](https://user-images.githubusercontent.com/55140514/141614022-25ddd874-c6bb-4180-89ca-7b06bf1921c5.png)

![image](https://user-images.githubusercontent.com/55140514/141614014-6c05720f-828f-4257-bf9b-383e21b6e526.png)

## Soal 12-13

- **No 12** : Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk mencari harta karun di super.franky.yyy.com. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk mendapatkan gambar (.png, .jpg), sedangkan Zoro mendapatkan sisanya. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan 10 kbps.
- **No 13** : Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya.

**Pembahasan:**
