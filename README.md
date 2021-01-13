# SDI-Project

## Judul 
Memproteksi Nginx dengan menggunakan NAXSI

## Deskrpsi
Nginx adalah salah satu web server paling populer dan berkinerja tinggi di dunia. Jadi, ini adalah bagian yang sangat penting dari administrator sistem untuk meningkatkan keamanan server Nginx. Naxsi "Nginx Anti XSS & SQL Injection" adalah firewall aplikasi web gratis, open-source, dan berkinerja tinggi yang dapat digunakan untuk melindungi server web Anda dari berbagai jenis serangan seperti SQL Injections dan Cross-Site Scripting. Naxsi bekerja dengan mendeteksi karakter tak terduga dalam permintaan HTTP GET dan POST.

## Langkah Kerja
Sebelum memulai, disarankan untuk memperbarui paket sistem Anda ke versi terbaru. Anda dapat memperbaruinya dengan perintah berikut:
```
apt-get update -y
apt-get upgrade -y
```

Setelah semua paket diperbarui, mulai ulang server Anda untuk menerapkan semua perubahan konfigurasi.

Selanjutnya, Anda perlu menginstal semua dependensi yang diperlukan untuk menginstal Naxsi. Anda dapat menginstal semuanya dengan perintah berikut:
```
apt-get install zlib1g zlib1g-dev build-essential bzip2 unzip libpcre3-dev libssl-dev libgeoip-dev wget unzip libxslt-dev libgd-dev -y
```
Setelah semua dependensi diinstal, Anda dapat melanjutkan ke langkah berikutnya.

### Kompilasi Nginx dengan Dukungan Naxsi
Secara default, Nginx sudah diinstal sebelumnya di Webdock Ubuntu 18.04 instance. Tapi, Naxsi adalah modul Nginx pihak ketiga yang tidak disertakan dengan paket Nginx.
Jadi, Anda perlu mengunduh sumber Nginx dan Naxsi, dan mengkompilasi Nginx dengan dukungan Naxsi.
Pertama, periksa versi Nginx yang diinstal pada instance Webdock Setelah Anda mendapatkan versi Nginx, buka situs web resmi Nginx dan unduh versi Nginx yang sama dengan perintah berikut:
```
wget http://nginx.org/download/nginx-1.17.0.tar.gz
```
Selanjutnya, unduh Naxsi versi terbaru dengan perintah berikut:
```
wget https://github.com/nbs-system/naxsi/archive/master.zip
```
Selanjutnya, ekstrak file sumber yang diunduh dengan perintah berikut:
```
tar -xvzf nginx-1.17.0.tar.gz
unzip master.zip
```
Selanjutnya, hentikan layanan Nginx yang sedang berjalan dengan perintah berikut:
```
systemctl stop nginx
```
Selanjutnya, hapus semua modul dinamis, tambahkan argumen baru –add-module = / root / naxsi-master / naxsi_src / dan --sbin-path = / usr / sbin / nginx dan tambahkan ./configure di awal argumen konfigurasi.

Setelah melakukan semua perubahan, salin perintah ini dan jalankan di terminal seperti yang ditunjukkan di bawah ini:
```
cd nginx-1.17.0
./configure --with-cc-opt='-g -O2 -fdebug-prefix-map=/build/nginx-RFWPEB/nginx-1.17.0=. -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro -Wl,-z,now -fPIC' --add-module=../naxsi-master/naxsi_src/ --sbin-path=/usr/sbin/nginx --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --with-http_addition_module --with-http_geoip_module=dynamic --with-http_gunzip_module --with-http_gzip_static_module --with-http_image_filter_module=dynamic --with-http_sub_module --with-http_xslt_module=dynamic --with-stream=dynamic --with-stream_ssl_module --with-stream_ssl_preread_module --with-mail=dynamic --with-mail_ssl_module
```
Selanjutnya, jalankan perintah make untuk menjalankan serangkaian tugas yang ditentukan di Makefile:
```
make
```
Selanjutnya, jalankan perintah make install untuk menyalin semua file konfigurasi ke lokasi yang benar:
```
make install
```
Setelah perintah di atas berjalan dengan sukses itu berarti Nginx telah dikompilasi ulang dengan dukungan Naxsi.

Selanjutnya, mulai layanan Nginx dengan perintah berikut:
```
systemctl start nginx
```
## Konfigurasi Naxsi
Naxsi hadir dengan seperangkat aturan inti yang dapat digunakan untuk menentukan bagaimana permintaan diblokir dari server. Jadi, Anda perlu menyalin aturan inti Naxsi ke direktori konfigurasi Nginx. Anda dapat menyalinnya dari direktori sumber Naxsi dengan perintah berikut:
```
cp -r /root/naxsi-master/naxsi_config/naxsi_core.rules /etc/nginx/
```
Selanjutnya, Anda perlu mengaktifkan aturan Naxsi dan menentukan jenis serangan berbeda yang dapat diblokir oleh Naxsi. Anda dapat melakukannya dengan membuat file naxsi.rules:
```
nano /etc/nginx/naxsi.rules
```
Tambahkan baris berikut:
```
SecRulesEnabled;
DeniedUrl "/error.html";

## Check Naxsi rules
CheckRule "$SQL >= 8" BLOCK;
CheckRule "$RFI >= 8" BLOCK;
CheckRule "$TRAVERSAL >= 4" BLOCK;
CheckRule "$EVADE >= 4" BLOCK;
CheckRule "$XSS >= 8" BLOCK;
```
Simpan dan tutup file setelah Anda selesai.

Selanjutnya, Anda perlu membuat file error.html untuk mengarahkan permintaan yang diblokir ke file error.html.
```
nano /usr/share/nginx/html/error.html
```
tambahkan perintah berikut
```
<html>
<head>
<title>Blocked By NAXSI</title>
</head>
<body>
<div style="text-align: center">
<h1>Malicious Request</h1>
<hr>
<p>This Request Has Been Blocked By NAXSI.</p>
</div>
</body>
</html>
```
Simpan dan tutup file. Kemudian, konfigurasikan server Nginx untuk menyertakan aturan Naxsi dengan mengedit file nginx.conf:
```
nano /etc/nginx/nginx.conf
```
Tambahkan baris berikut dalam bagian http {}:
```
include /etc/nginx/naxsi_core.rules;
```
Simpan dan tutup file.

Selanjutnya, buat file host virtual Nginx dan sertakan naxsi.rules. Anda harus menyertakan file naxsi.rules di setiap host virtual yang ingin Anda aktifkan Naxsi.
```
nano /etc/nginx/sites-available/naxsi.conf
```
tambahkan lines berikut
```
server {
listen 80;

root /usr/share/nginx/html;
index index.html index.htm;

# Make site accessible from http://localhost/
server_name 136.243.240.38;

location / {
include /etc/nginx/naxsi.rules;

try_files $uri $uri/ =404;

}

}
```
Simpan dan tutup file. Kemudian, periksa Nginx untuk setiap kesalahan sintaks dengan perintah berikut:
nginx -t
Jika semuanya berjalan dengan baik, Anda harus mendapatkan output berikut:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Terakhir, restart layanan Nginx untuk menerapkan semua perubahan konfigurasi:
```
systemctl restart nginx
```
### Test Naxsi Firewall
Firewall Naxsi sekarang telah diinstal dan dikonfigurasi, saatnya untuk mengujinya terhadap serangan SQLi dan XSS.

Untuk menguji Naxsi melawan serangan XSS, buka browser web Anda dari sistem jarak jauh dan ketik URL public ip address/q?= "> <script> alert (1) </script>. Sesuai dengan aturan Naxsi yang mana Anda telah mengkonfigurasi sebelumnya, Anda akan diarahkan ke halaman error.html dengan pesan berikut:
![Alt text](https://webdock.io/application/files/6615/6621/3971/naxsi1.png)

Anda juga dapat memverifikasi ini dengan memeriksa log Nginx seperti yang ditunjukkan di bawah ini:
```
tail -f /var/log/nginx/error.log
```





