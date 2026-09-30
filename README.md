# Jarkom-Modul-2-2026-K-27

## Member

| Nama | NRP |
| :--- | :--- |
| Muhammad Nadhif Pasya Ikhsan | 5027251084 |
| Akhdan Hafiz Anugrah | 5027251094 |


## Soal 1
Sebagai pusat kesadaran The Mesh, rootkit harus merentangkan koneksinya ke lima gerbang utama (Switch). Tetapkan alamat IP dan default gateway untuk seluruh Entitas, mulai dari para operator (alpha, beta, gamma), penjaga directory (prab, tedd), gerbang penyaring (abbey, penny), hingga repository (obladi, desmond, oblada, molly) sesuai dengan topologi pembagian switch yang dirancang. 

Konfigurasi IP pada setiap klien dan server dilakukan menggunakan skrip. Contoh konfigurasi pada klien (misal: `Alpha`):
```bash
ip addr add 10.77.6.2/24 dev eth0
ip link set eth0 up
ip route add default via 10.77.6.1
```
Kemudian setelah melakukan konfigurasi pada tiap klien lakukan pengujian dengan konfigurasi:
``` bash
ping 10.77.7.2 -c 2
ping 10.77.1.2 -c 2
ping 10.77.4.2 -c 2
````
![alt text](<foto 1.png>)

![alt text](<foto 2.png>)

![alt text](<foto 3.png>)

![alt text](<foto 4.png>)

## Soal 2
Meskipun The Mesh beroperasi dalam bayang-bayang, Rootkit menyadari bahwa Entitas di dalamnya masih membutuhkan asupan paket dari dunia luar. Buka jalur menuju NAT dengan memastikan antarmuka WAN di router rootkit aktif. Konfigurasikan NAT agar dapat meneruskan lalu lintas keluar bagi seluruh alamat internal, sehingga semua host di dalam jaringan dapat menjangkau internet publik menggunakan IP address.

Mengonfigurasi `rootkit` sebagai router sentral dan mengaktifkan NAT (Network Address Translation) agar seluruh entitas di dalam jaringan The Mesh dapat mengakses internet.

Langkah pengerjaannya adalah mengaktifkan IP forwarding dan aturan masquerade pada iptables di node `rootkit`:
``` bash
sysctl -w net.ipv4.ip_forward=1
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -P FORWARD ACCEPT
```
Kemudian lakukan verifikasi `ping` di `rootkit` dan `alpha`:
``` bash
ping 8.8.8.8 -c 3
```
![alt text](<foto 5.png>)

![alt text](<foto 6.png>)

## Soal 3
Jaringan rahasia tidak akan berfungsi tanpa sinkronisasi antar divisi. Pastikan seluruh Entitas dapat saling terhubung dan berkomunikasi lintas jalur (routing internal via rootkit berfungsi). Untuk menghindari fragmentasi saat persiapan, pastikan setiap host non-router menambahkan **resolver 192.168.122.1 (tambah di file /etc/resolv.conf, kalau sudah pakai resolver itu tidak perlu memasukkan resolver google)** saat antarmukanya aktif agar akses untuk mengunduh paket instalasi dari internet tersedia sejak awal beroperasi.

Menambahkan nameserver `192.168.122.1` ke dalam berkas resolver setiap entitas non-router agar terhubung ke DNS internet untuk kebutuhan instalasi paket di awal.

Perintah berikut dijalankan pada semua entitas selain `rootkit`:
``` bash
echo "nameserver 192.168.122.1" > /etc/resolv.conf
apt-get update
```
![alt text](<foto 7.png>)

![alt text](<foto 8.png>)

Kemudian lakukan verifikasi dengan konfigurasi berikut:
``` bash
# A. Ping ke (delta - 10.77.7.2)
ping 10.77.7.2 -c 3

# B. Ping ke (penny - 10.77.4.2 & abbey - 10.77.5.2)
ping 10.77.4.2 -c 3
ping 10.77.5.2 -c 3

# C. Ping ke (prab - 10.77.1.2)
ping 10.77.1.2 -c 3

# D. Ping ke (obladi - 10.77.1.4 & oblada - 10.77.1.6)
ping 10.77.1.4 -c 3
ping 10.77.1.6 -c 3

ping google.com -c 3 
```
![alt text](<foto 9.png>)

![alt text](<foto 10.png>)

![alt text](<foto 11.png>)

![alt text](<foto 12.png>)

![alt text](<foto 13.png>)

## Soal 4
Penjaga Direktori mulai menuliskan hukum The Mesh. Pada node `prab`, bangun zona `<xxxx>.com` sebagai authoritative dengan SOA yang menunjuk ke `prab.<xxxx>.com`, serta tambahkan catatan NS untuk `prab.<xxxx>.com` dan `tedd.<xxxx>.com`. Buat A record untuk `prab.<xxxx>.com` dan `tedd.<xxxx>.com` yang mengarah ke alamat IP mereka masing-masing, serta A record apex `<xxxx>.com` yang mengarah ke gerbang aplikasi dinamis (penny). Aktifkan fitur notify dan allow-transfer ke `tedd`, lalu set forwarders ke `192.168.122.1`. Di node tedd, tarik zona `<xxxx>.com` dari master dan pastikan server menjawab secara authoritative. Setelah fondasi nama ini berdiri kokoh, perbarui urutan resolver pada seluruh Entitas non-router menjadi: IP `prab`, IP `tedd`, lalu `192.168.122.1`. Verifikasi bahwa query ke domain apex maupun hostname di dalam zona dijawab dengan benar oleh `prab` atau `tedd`.

Membangun DNS Server internal untuk domain utama `k-27.com`. Node `prab` bertugas sebagai Authoritative Nameserver Master, sedangkan `tedd` bertugas sebagai DNS Slave.

Skrip Pengerjaan pada Node `prab` (Master):
``` bash
apt-get install bind9 bind9utils dnsutils -y

# Mendaftarkan zona k-27.com
cat << 'EOF' > /etc/bind/named.conf.local
zone "k-27.com" {
    type master;
    file "/etc/bind/db.k-27.com";
    allow-transfer { 10.77.1.3; };
    also-notify { 10.77.1.3; };
    notify yes;
};
EOF

# Membuat database zona (Forward DNS)
cat << 'EOF' > /etc/bind/db.k-27.com
$TTL    604800
@       IN      SOA     prab.k-27.com. admin.k-27.com. ( 2026092801 604800 86400 2419200 604800 )
@       IN      NS      prab.k-27.com.
@       IN      NS      tedd.k-27.com.
prab    IN      A       10.77.1.2
tedd    IN      A       10.77.1.3
EOF

service bind9 restart
```

Skrip Pengerjaan pada Node `tedd` (Slave):
``` bash
apt-get install bind9 bind9utils dnsutils -y

cat << 'EOF' > /etc/bind/named.conf.local
zone "k-27.com" {
    type slave;
    masters { 10.77.1.2; };
    file "/var/lib/bind/db.k-27.com";
};
EOF

service bind9 restart
```
Kemudian lakukan verifikasi dengan konfigurasi berikut pada node `alpha`:
``` bash
dig @10.77.1.2 k-27.com
dig @10.77.1.2 prab.k-27.com

dig @10.77.1.3 k-27.com
dig @10.77.1.3 tedd.k-27.com

nslookup k-27.com
nslookup prab.k-27.com
nslookup tedd.k-27.com
```
![alt text](<foto 14.png>)

![alt text](<foto 15.png>)

![alt text](<foto 16.png>)

![alt text](<foto 17.png>)

![alt text](<foto 18.png>)

## Soal 5
"Entitas tanpa identitas adalah anomali," pesan Rootkit. Namai semua Entitas (hostname) sesuai glosarium: `rootkit, alpha, beta, gamma, delta, epsilon, prab, tedd, abbey, penny, obladi, desmond, oblada, molly`, dan verifikasi bahwa setiap host mengenali hostname tersebut secara system-wide. Buat setiap domain untuk masing-masing node sesuai dengan namanya (contoh: `alpha.<xxxx>.com`) dan assign IP masing-masing juga. Lakukan pengecualian untuk node yang bertanggung jawab atas `prab` dan `tedd`.

Menamai seluruh 14 entitas secara system-wide sesuai glosarium, mendaftarkan record A masing-masing ke dalam zona DNS `k-27.com`, dan menata ulang urutan resolver.

- Di setiap node, atur hostname: `hostname <nama_node> && echo "<nama_node>" > /etc/hostname`.

- Di node prab, tambahkan IP A Record semua entitas ke dalam `/etc/bind/db.k-27.com`:
``` bash
rootkit IN      A       10.77.1.1
penny   IN      A       10.77.4.2
abbey   IN      A       10.77.5.2
alpha   IN      A       10.77.6.2
delta   IN      A       10.77.7.2
obladi  IN      A       10.77.1.4
desmond IN      A       10.77.1.5
oblada  IN      A       10.77.1.6
molly   IN      A       10.77.1.7
# (Dan entitas klien lainnya)
```
Kemudian di semua klien, lakukan perubahan urutan resolver:
``` bash
cat << 'EOF' > /etc/resolv.conf
nameserver 10.77.1.2
nameserver 10.77.1.3
nameserver 192.168.122.1
EOF
```

Kemudian lakukan verifikasi dengan konfigurasi berikut:
``` bash
# Di alpha dan rootkit lakukan
hostname

# Kemudian lakukan di Alpha dan Delta
dig delta.k-27.com +short
dig penny.k-27.com +short
dig abbey.k-27.com +short
dig obladi.k-27.com +short
dig oblada.k-27.com +short

ping delta.k-27.com -c 2
ping abbey.k-27.com -c 2
```
![alt text](<foto 19.png>)

![alt text](<foto 20.png>)

![alt text](<foto 21.png>)

![alt text](<foto 22.png>)

## Soal 6
Pastikan zone transfer berjalan, pastikan `tedd` telah menerima salinan zona terbaru dari `prab`. Nilai serial SOA di keduanya harus sama karena `keduanya tidak bisa dipisahkan dan saling melengkapi`.

Memastikan mekanisme replikasi basis data DNS (Zone Transfer) dari Master (prab) ke Slave (tedd) berjalan secara konsisten dan Serial SOA di keduanya sama persis.

- Pada `prab`, ubah nilai Serial SOA pada file `/etc/bind/db.k-27.com` (dinaikkan).

- Terapkan perubahan dengan `rndc reload` atau `service bind9 restart`.

- Gunakan perintah `dig AXFR` dari klien untuk memverifikasi penarikan zona lengkap.

Kemudian setelah konfigurasi diatas lakukan verifikasi dengan konfigurasi:
``` bash
# Jalankan di prab
named-checkzone k-27.com /etc/bind/db.k-27.com
rndc reload || service bind9 reload

# Jalankan di alpha/beta/delta
dig SOA k-27.com @10.77.1.2 +short
dig SOA k-27.com @10.77.1.3 +short

# Jalankan di tedd
dig AXFR k-27.com @10.77.1.2
grep -i "transfer of 'k-27.com'" /var/log/syslog 2>/dev/null || rndc status
```
![alt text](<foto 23.png>)

![alt text](<foto 24.png>)

![alt text](<foto 25.png>)

![alt text](<foto 26.png>)

## Soal 7
`abbey` dan `penny` sebagai gerbang utama, `obladi dan desmond sebagai web statis, oblada dan molly sebagai web dinamis`. Tambahkan pada zona `<xxxx>.com` A record untuk `vault.<xxxx>.com` `(IP obladi & desmond)`, dan `core.<xxxx>.com` `(IP oblada & molly)`. Tetapkan CNAME:
- `www.<xxxx>.com → penny.<xxxx>.com`
- `static.<xxxx>.com → abbey.<xxxx>.com`

Verifikasi dari dua klien berbeda bahwa seluruh hostname tersebut ter-resolve ke tujuan yang benar dan konsisten.

Menambahkan catatan Multi-A (DNS Round-Robin) untuk domain repositori data `(vault & core)` serta Canonical Name (CNAME) untuk gerbang proxy `(www & static)`.

Skrip Pengerjaan pada Node `prab`:
Tambahkan record berikut ke `/etc/bind/db.k-27.com`:
``` bash
; Area Vault & Core (Multi-A Record / Round-Robin)
vault   IN      A       10.77.1.4
vault   IN      A       10.77.1.5
core    IN      A       10.77.1.6
core    IN      A       10.77.1.7

; Canonical Name (CNAME)
www     IN      CNAME   penny.k-27.com.
static  IN      CNAME   abbey.k-27.com.
```
Jalankan `service bind9 restart`.

Lakukan verifikasi pada `prab` untuk memvalidasi perubahannya:
``` bash
named-checkzone k-27.com /etc/bind/db.k-27.com
rndc reload || service bind9 reload
rndc notify k-27.com
```

Lakukan verifikasi pada node `alpha & delta`:
``` bash
# Uji di alpha
dig vault.k-27.com +short
dig core.k-27.com +short
dig www.k-27.com
dig static.k-27.com

# UJi di delta
dig vault.k-27.com +short
dig core.k-27.com +short
nslookup www.k-27.com
nslookup static.k-27.com
ping www.k-27.com -c 2
ping static.k-27.com -c 2
```
![alt text](<foto 27.png>)

![alt text](<foto 28.png>)

![alt text](<foto 29.png>)

## Soal 8
Di `prab (master)` deklarasikan reverse zone untuk segmen jaringan  tempat `abbey, penny, area vault, dan area core` berada. Di `tedd (slave)` tarik reverse zone tersebut sebagai slave, isi PTR untuk keempat hostname itu agar pencarian balik IP address mengembalikan hostname yang benar, lalu pastikan query reverse untuk alamat `abbey, penny, area vault, dan area core` dijawab authoritative.

Mendeklarasikan Reverse DNS Zone pada master `prab` dan ditarik ke slave `tedd` untuk menerjemahkan IP Address entitas abbey, penny, vault, dan core menjadi hostname (PTR Record).

Langkah Pengerjaan pada Node `prab`:

Tambahkan deklarasi zona reverse di `/etc/bind/named.conf.local`:
``` bash
zone "1.77.10.in-addr.arpa" { type master; file "/etc/bind/db.10.77.1"; allow-transfer { 10.77.1.3; }; };
zone "4.77.10.in-addr.arpa" { type master; file "/etc/bind/db.10.77.4"; allow-transfer { 10.77.1.3; }; };
zone "5.77.10.in-addr.arpa" { type master; file "/etc/bind/db.10.77.5"; allow-transfer { 10.77.1.3; }; };
```
Buat file PTR Record `/etc/bind/db.10.77.1` (Vault & Core):
``` bash
4       IN      PTR     vault.k-27.com.
5       IN      PTR     vault.k-27.com.
6       IN      PTR     core.k-27.com.
7       IN      PTR     core.k-27.com.
```
*(Hal serupa dilakukan untuk IP Penny dan Abbey pada filenya masing-masing).*

Kemudian lakukan verifikasi dengan konfigurasi berikut:
``` bash
# Pengujian di klien alpha
dig -x 10.77.5.2 @10.77.1.2
dig -x 10.77.4.2 @10.77.1.2
dig -x 10.77.1.4 @10.77.1.2
dig -x 10.77.1.6 @10.77.1.2

# Pengujian di klien tedd
dig -x 10.77.5.2 @10.77.1.3
dig -x 10.77.1.4 @10.77.1.3

dig -x 10.77.5.2 +short
dig -x 10.77.4.2 +short
dig -x 10.77.1.4 +short
dig -x 10.77.1.5 +short
dig -x 10.77.1.6 +short
dig -x 10.77.1.7 +short
```
![alt text](<foto 30.png>)

![alt text](<foto 31.png>)

![alt text](<foto 32.png>)

![alt text](<foto 33.png>)

## Soal 9
Jalankan layanan web statis pada hostname di node `area vault` (menggunakan apache). Buka folder direktori `/arsip/` dan aktifkan fitur `autoindex (directory listing)` pada konfigurasi Apache sehingga seluruh daftar file di dalamnya dapat ditelusuri langsung dari browser. Akses pengujian harus dilakukan melalui hostname, bukan IP address.

Membangun web server statis di Area Vault `(obladi & desmond)` menggunakan Apache2, dan mengaktifkan fitur Autoindex pada folder `/arsip/`.

Langkah Pengerjaan pada `obladi` dan `desmond`:
``` bash
apt-get install apache2 -y
a2enmod autoindex
mkdir -p /var/www/html/arsip
echo "Dokumen Rahasia" > /var/www/html/arsip/dokumen.txt

cat << 'EOF' > /etc/apache2/sites-available/vault.conf
<VirtualHost *:80>
    ServerName obladi.k-27.com
    ServerAlias vault.k-27.com
    DocumentRoot /var/www/html
    <Directory /var/www/html/arsip>
        Options +Indexes +FollowSymLinks
        Require all granted
    </Directory>
</VirtualHost>
EOF

a2dissite 000-default.conf
a2ensite vault.conf
service apache2 restart
```

Kemudian bisa melakukan verifikasi dengan beberapa konfigurasi:
``` bash
curl -i http://obladi.k-27.com/arsip/

curl http://obladi.k-27.com/arsip/dokumen_vault_01.txt

curl http://desmond.k-27.com/arsip/dokumen_vault_02.txt

curl -i http://desmond.k-27.com/arsip/

curl http://vault.k-27.com/arsip/

curl -i http://localhost/arsip/

ss -tlpn | grep 80

curl -i http://localhost/arsip/
```
![alt text](<foto 34.png>)

![alt text](<foto 35.png>)

![alt text](<foto 36.png>)

![alt text](<foto 37.png>)

![alt text](<foto 38.png>)

## Soal 10
Jalankan layanan web dinamis (PHP-FPM) pada hostname di node `core` (menggunakan nginx). Buat sebuah aplikasi sederhana yang memuat halaman beranda dan halaman `profil`. Terapkan aturan rewrite pada server sehingga akses ke `/profil` dapat berfungsi dengan URL bersih `(tanpa akhiran .php)`. Akses pengujian wajib dilakukan melalui hostname.

Membangun web server dinamis menggunakan Nginx dan PHP-FPM di Area Core `(oblada & molly)`, serta menerapkan URL Rewrite agar halaman profil berformat Clean URL `(/profil)`.

Langkah Pengerjaan pada `oblada` dan `molly`:
``` bash
apt-get install nginx php-fpm -y
mkdir -p /var/www/html
echo "<?php echo 'Halaman Profil Dinamis'; ?>" > /var/www/html/profil.php

cat << 'EOF' > /etc/nginx/sites-available/core-app
server {
    listen 80;
    server_name oblada.k-27.com core.k-27.com;
    root /var/www/html;
    index index.php index.html;

    # Aturan Clean URL
    location /profil {
        rewrite ^/profil/?$ /profil.php last;
    }

    # Handler PHP-FPM
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php-fpm.sock;
    }
}
EOF

ln -s /etc/nginx/sites-available/core-app /etc/nginx/sites-enabled/
service nginx restart
```

Lakukan verifikasi dengan beberapa konfigurasi berikut:
``` bash
# Jalankan di node oblada
curl -i http://localhost/
curl -i http://localhost/profil

# Jalankan di node molly
curl -i http://localhost/
curl -i http://localhost/profil

# Jalankan di node alpha
curl -i http://oblada.k-27.com/
curl -i http://oblada.k-27.com/profil
curl -i http://molly.k-27.com/profil
curl http://core.k-27.com/
curl http://core.k-27.com/profil
```
![alt text](<foto 39.png>)

![alt text](<foto 40.png>)

![alt text](<foto 41.png>)

![alt text](<foto 42.png>)

![alt text](<foto 43.png>)

![alt text](<foto 44.png>)

## Soal 11