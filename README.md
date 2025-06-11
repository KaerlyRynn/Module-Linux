> [!NOTE]
> ==Gunakan User ROOT saat login==
## 🔧 Update sistem
```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🖥️ cara melihat spek/infomasi linux
- apt install neofetch
- neofetch

---

btop

---

## 👤 User

#### 1. Cara Menambah kan User
```bash
sudo adduser <username>
```

#### 2. Cara Menambakan Ke Grup root
```bash
usermod -aG sudo <username>
```

---

## 🛜 SSH (Secure Shell)
### ✅ 1. Instalasi OpenSSH Server
Pertama, buka terminal Anda dan instal paket openssh-server dan openssh-client:
```bash
sudo apt update
sudo apt install openssh-server openssh-client -y
```
Setelah instalasi, SSH server akan otomatis berjalan. Anda bisa memeriksanya dengan:
```bash
sudo systemctl status ssh
```
Jika statusnya active (running), berarti SSH server sudah berjalan.
### ✅ 2. Konfigurasi Dasar SSH
File konfigurasi utama SSH terletak di /etc/ssh/sshd_config. Penting untuk selalu membuat cadangan file konfigurasi sebelum melakukan perubahan:
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```
Sekarang, buka file konfigurasi dengan editor teks favorit Anda (misalnya nano atau vim):
```bash
sudo nano /etc/ssh/sshd_config
```
Berikut beberapa konfigurasi penting yang perlu Anda perhatikan:
- Port: Secara default, SSH menggunakan port 22. Untuk meningkatkan keamanan, disarankan untuk mengubah port ini ke nomor lain yang tidak standar (misalnya 2222, 2022, dll.). Cari baris #Port 22 dan ubah menjadi: Port 2222 # Ganti 2222 dengan port pilihan Anda Pastikan Anda menghapus tanda pagar # di depannya.

- ListenAddress: Jika Anda memiliki beberapa IP address di server Anda dan ingin SSH hanya mendengarkan pada IP tertentu, Anda bisa mengaturnya di sini. Jika tidak, biarkan saja.
#ListenAddress 0.0.0.0 # Mendengarkan di semua IP.

### ✅ 3. Mematikan Root Login (Sangat Disarankan!)

Mematikan root login melalui SSH adalah langkah keamanan yang sangat penting. Dengan begitu, penyerang tidak bisa langsung mencoba masuk sebagai root. Mereka harus masuk dengan user biasa terlebih dahulu, baru kemudian menggunakan sudo atau su untuk mendapatkan hak akses root. Di dalam file /etc/ssh/ sshd_config, cari baris PermitRootLogin. Secara default, mungkin terlihat seperti ini:

```bash
#PermitRootLogin prohibit-password
```
Atau mungkin:
```bash
PermitRootLogin yes
```
Ubah atau tambahkan baris berikut agar menjadi:
```bash
PermitRootLogin no
```
Pastikan Anda menghapus tanda pagar # jika ada.

---

### ☑️ 4. Menggunakan Autentikasi Kunci SSH (Sangat Disarankan!)

Meskipun mematikan root login, tetap disarankan untuk menggunakan autentikasi kunci SSH dari pada password. Ini jauh lebih aman karena tidak ada password yang perlu ditebak.

#### 🛜 A. Membuat Pasangan Kunci SSH (di komputer lokal Anda, bukan server)

Jika Anda belum memiliki pasangan kunci SSH, buatlah di komputer lokal Anda:
```bash
ssh-keygen -t rsa -b 4096
```
Perintah ini akan meminta Anda untuk menyimpan kunci di lokasi default (~/.ssh/id_rsa untuk kunci privat, dan ~/.ssh/id_rsa.pub untuk kunci publik). Anda juga bisa memberikan passphrase untuk kunci privat Anda sebagai lapisan keamanan tambahan.

#### 🛜 B. Menyalin Kunci Publik ke Server

Setelah membuat kunci, salin kunci publik Anda ke server. Anda bisa menggunakan ssh-copy-id:

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub <username>@<alamat_ip_server> -p <port_ssh_anda>
```

Ganti <username> dengan nama user Anda di server, <alamat_ip_server> dengan IP server Anda, dan <port_ssh_anda> dengan port SSH yang sudah Anda konfigurasikan (jika Anda mengubahnya). Jika ssh-copy-id tidak tersedia atau Anda lebih suka cara manual:

```bash
cat ~/.ssh/id_rsa.pub | ssh <username>@<alamat_ip_server> -p <port_ssh_anda> "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

Anda akan diminta untuk memasukkan password user Anda di server.

#### 🛜 C. Mengkonfigurasi Server untuk Hanya Mengizinkan Autentikasi Kunci (Opsional, tapi Direkomendasikan)

Setelah Anda berhasil masuk ke server menggunakan kunci SSH, Anda bisa menonaktifkan autentikasi password untuk keamanan yang lebih tinggi. Di file /etc/ssh/sshd_config, cari baris PasswordAuthentication. Ubah atau tambahkan baris berikut:
```bash
PasswordAuthentication no
```

#### 🛜 5. Mengatur Firewall (UFW) untuk Port SSH

Jika Anda menggunakan Uncomplicated Firewall (UFW) di server Anda (sangat direkomendasikan), Anda perlu mengizinkan port SSH yang baru.

Jika Anda menggunakan port 22:
```bash
sudo ufw allow ssh
```

Jika Anda mengubah port SSH ke 2222 (seperti contoh di atas):
```bash
sudo ufw allow 2222/tcp
```

Pastikan UFW sudah diaktifkan:
```bash
sudo ufw enable
```

#### 🛜 6. Restart Layanan SSH

Setelah semua perubahan di file /etc/ssh/sshd_config, Anda harus me-restart layanan SSH agar perubahan diterapkan:
```bash
sudo systemctl restart ssh
```

#### 🛜 7. Pengujian Koneksi SSH

Dari komputer lokal Anda, coba koneksi ke server menggunakan user biasa dan port SSH yang baru:
```bash
ssh <username>@<alamat_ip_server> -p <port_ssh_anda>
```
Jika Anda berhasil masuk tanpa diminta password (jika Anda mengaktifkan autentikasi kunci) dan tidak bisa masuk sebagai root, berarti konfigurasi Anda berhasil.

#### 😎 Ringkasan Langkah-langkah Penting:
1. Instalasi OpenSSH Server.
2. Edit /etc/ssh/sshd_config:

- Ubah Port ke nomor yang tidak standar.
- Set PermitRootLogin no.
- Set PasswordAuthentication no (setelah Anda yakin autentikasi kunci berfungsi).

3. Buat pasangan kunci SSH (di komputer lokal Anda).
4. Salin kunci publik ke server.
5. Konfigurasi UFW untuk mengizinkan port SSH baru.
6. Restart layanan SSH.
7. Uji koneksi.
Dengan mengikuti langkah-langkah ini, Anda akan memiliki instalasi dan konfigurasi SSH yang lebih aman, dengan root login dinonaktifkan dan (opsional) autentikasi kunci diaktifkan.


---

## 🔍 Web Server

### 🧱 Nginx
```bash
sudo apt install nginx -y
```
#### ⚙️ Konfigurasi Nginx

Buat file konfigurasi baru:

```bash
sudo nano /etc/nginx/sites-available/wordpress
```

```bash
cd /etc/nginx/sites-available/
nano namafile
```

Isi dengan konfigurasi berikut:

```nginx
server {
    listen 80;
    server_name domain.anda.com;
    root /var/www/wordpress;

    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires max;
        log_not_found off;
    }

    location = /favicon.ico { log_not_found off; access_log off; }
    location = /robots.txt  { log_not_found off; access_log off; allow all; }

    error_page 404 /index.php;
}
```

> [!NOTE]
> ubah `domain.anda.com` menjadi domain sendiri. (server_name domain.anda.com;)

> [!NOTE]
> Ubah jalur direktori folder `wordpress` menjadi folder anda. (root /var/www/wordpress;)

Aktifkan site dan restart Nginx:

```bash
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

> [!NOTE]
> Saat 
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/.  Ubahlah jalur folder `wordpress` menjadi `folder kalian`.
---

### 🧱 Apache2
```bash
sudo apt install apache2 -y
```
---

## ✅Install certbot (SSL)

### ✅ Langkah 1: Install Certbot dan plugin Nginx

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

---

### ✅ Langkah 2: Pastikan domain sudah mengarah ke server

Pastikan DNS domain kamu (contoh: `example.com`) sudah diarahkan ke IP publik server (A record).

---

### ✅ Langkah 3: Pastikan Nginx aktif dan sudah ada server block

> [!NOTE]
> Misalnya file kamu ada di:

```bash
/etc/nginx/sites-available/wordpress
```

Pastikan ada `server_name` / `full domain` yang sesuai:

```nginx
server_name sub.domain.com;
```
> [!NOTE]
> Ubah `sub.domain.com` sesuai full domain masing-masing

Uji konfigurasi:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

### ✅ Langkah 4: Jalankan Certbot

```bash
sudo certbot --nginx -d sub.namadomain.com
```
> [!NOTE]
> Ganti nama `sub.namadomain.com` menjadi `domain sendiri`

Certbot akan:

-   Mendeteksi server block
    
-   Menambahkan konfigurasi SSL secara otomatis
    
-   Menanyakan domain dan email untuk notifikasi pembaruan SSL
    

Contoh output interaktif:

```pgsql
Enter email address: kamu@email.com
Agree to terms? Y
Share email with EFF? N
Select domain: [pilih 1 atau lebih dengan spasi]
```

---

### ✅ Langkah 5: Uji SSL

Setelah selesai, akses:

```arduino
https://example.com
```

SSL sudah aktif jika muncul gembok 🔒 di browser.

---

### ✅ cara certbot & melihat certbot : 

- Menambah certbot (SSL).

   ```bash
   certbot --nginx -d domainkamu.com
   ```

- Menampilkan certbot (SSL) yang sudah dibuat.

   ```bash
   certbot certificates
   ```

---

### 🔁 Perpanjangan Otomatis

Certbot memasang cron otomatis untuk memperpanjang setiap 60 hari. Kamu bisa tes manual:

```bash
sudo certbot renew --dry-run
```

---

### 🧼 Catatan:

-   Jika menggunakan firewall:
    
    ```bash
    sudo ufw allow 'Nginx Full'
    sudo ufw delete allow 'Nginx HTTP'
    ```
    
-   Jika kamu ingin mengarahkan HTTP ke HTTPS otomatis:  
    Certbot biasanya menanyakan ini, tapi kalau tidak, kamu bisa tambahkan redirect di block HTTP:
    
    ```nginx
    server {
        listen 80;
        server_name example.com www.example.com;
        return 301 https://$host$request_uri;
    }
    ```

---

## 🛢️ MariaDB/mySQL
### 🛢️ Install MariaDB

```bash
sudo apt install mariadb-server -y
```

#### a. Amankan MariaDB

```bash
sudo mysql_secure_installation
```

> [!NOTE]
> Ikuti wizard: atur password root, hapus user anonymous, hapus database test, dan nonaktifkan remote login root.

#### b. Buat database

> [!NOTE]
> Jika menggunakan mySQL liat command `pertama`, Jika menggunakan MariaDB liat command `kedua`

`1. mySQL`
```bash
sudo mysql -u root -p
```

`2. MariaDB`
```bash
sudo mariadb -u root -p
```

> [!NOTE]
> Login Menggunakan Password root

```sql
CREATE DATABASE nama_db DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'nama_user'@'localhost' IDENTIFIED BY 'pw_user';
GRANT ALL PRIVILEGES ON nama_db.* TO 'nama_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

> [!NOTE]
> Yang harus di ubah.

DATABASE : `nama_db`

USERNAME : `nama_user`

PASSWORD : `pw_user`

------

### ✅ Menghapus database dan user

#### 1.  Masuk ke MariaDB/MySQL
```bash
sudo mariadb -u root -p
```

> Masukan password root

#### 2.  Hapus database
```sql
DROP DATABASE nama_database;
```

#### 3.  Hapus user
```sql
DROP USER 'nama_user'@'localhost';
```

#### 4. Hapus hak akses yang mungkin masih tersisa (opsional tapi dianjurkan)
```sql
FLUSH PRIVILEGES;
```

#### 5. Keluar dari MariaDB
```sql
EXIT;
```

---
#### ☑️Cek DATABASE & USER☑️
Jika kamu ingin memastikan user atau database telah benar-benar dihapus, kamu bisa cek:

-   Daftar database:
    
    ```sql
    SHOW DATABASES;
    ```
    
-   Daftar user:
    
    ```sql
    SELECT User, Host FROM mysql.user;
    ```
---

## 🌐 WordPress (Debian 12)
### 🌐 1. Download dan atur WordPress

```bash
cd /var/www/
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xvzf latest.tar.gz
sudo chown -R www-data:www-data wordpress
sudo chmod -R 755 wordpress
```

### ⚙️ 2. Konfigurasi Nginx

Buat file konfigurasi baru:

```bash
sudo nano /etc/nginx/sites-available/wordpress
```

Isi dengan konfigurasi berikut:

```nginx
server {
    listen 80;
    server_name domain_anda.com;
    root /var/www/wordpress;

    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires max;
        log_not_found off;
    }

    location = /favicon.ico { log_not_found off; access_log off; }
    location = /robots.txt  { log_not_found off; access_log off; allow all; }

    error_page 404 /index.php;
}
```

Aktifkan site dan restart Nginx:

```bash
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### 🔚 3. Akses WordPress

Buka browser dan akses:

```cpp
http://IP_SERVER_ANDA
```

Ikuti wizard instalasi WordPress: pilih bahasa, masukkan info database (`wordpress`, `wpuser`, `passwordku`), dan selesaikan setup.

### ✅ Selesai!

WordPress sekarang telah terinstal di server Debian 12 dengan PHP 8.0, MariaDB, dan Nginx.

Jika kamu ingin menambahkan HTTPS menggunakan **Let's Encrypt**, saya bisa bantu juga.

Ingin dilanjutkan dengan konfigurasi HTTPS?

---
