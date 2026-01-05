
# Instalasi EPrints di Debian

## Langkah Instalasi

### 1. Update Sistem
```bash
sudo apt-get update
sudo apt-get upgrade -y
```

### 2. Install Dependencies
```bash
sudo apt-get install -y git apache2 mariadb-server mariadb-client \
  libmariadb-dev perl libapache2-sitecontrol-perl libxml2-dev libxml-libxml-perl \
  libunicode-string-perl libterm-readkey-perl libmime-lite-perl \
  libdigest-sha-perl libdbd-mysql-perl libxml-parser-perl \
  libxml2-utils libarchive-zip-perl libjson-perl libyaml-perl \
  libsearch-xapian-perl libtex-encode-perl libio-string-perl \
  libdata-uuid-perl imagemagick xpdf antiword elinks poppler-utils \
  libapache2-mod-perl2 libapache2-request-perl libapache2-reload-perl \
  libapache2-sitecontrol-perl libxml-libxslt-perl libfile-mimeinfo-perl \
  liblingua-stem-perl libtext-unidecode-perl
```

### 3. Konfigurasi Apache Modules
```bash
sudo a2enmod perl
sudo a2enmod rewrite
sudo systemctl restart apache2
```

### 4. Membuat User EPrints
```bash
sudo useradd -m -s /bin/bash eprints
```

### 5. Konfigurasi Database MariaDB
```bash
# Login ke MariaDB
sudo mysql -u root

# Jalankan query berikut di dalam MySQL prompt:
```
```sql
CREATE USER 'eprints'@'localhost' IDENTIFIED BY 'password_anda';
GRANT ALL PRIVILEGES ON *.* TO 'eprints'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

**Atau gunakan satu baris command:**
```bash
sudo mysql -u root -e "CREATE USER 'eprints'@'localhost' IDENTIFIED BY 'password_anda';"
sudo mysql -u root -e "GRANT ALL PRIVILEGES ON *.* TO 'eprints'@'localhost' WITH GRANT OPTION;"
sudo mysql -u root -e "FLUSH PRIVILEGES;"
```

> ⚠️ **Catatan:** Ganti `password_anda` dengan password yang aman

### 6. Persiapan Direktori EPrints
```bash
sudo mkdir -p /opt/eprints3
sudo chown eprints:eprints /opt/eprints3
sudo chmod 2775 /opt/eprints3
```

### 7. Clone EPrints dari GitHub
```bash
# Clone repository
cd /tmp
git clone https://github.com/eprints/eprints3.4.git
cd eprints3.4

# Checkout versi 3.4.6
git checkout tags/v3.4.6

# Copy ke direktori instalasi
sudo cp -r * /opt/eprints3/
sudo chown -R eprints:eprints /opt/eprints3
```

**Alternatif (clone langsung):**
```bash
sudo su - eprints
git clone https://github.com/eprints/eprints3.4.git /opt/eprints3
cd /opt/eprints3
git checkout tags/v3.4.6
exit
```

### 8. Verifikasi File EPrints
```bash
# Cek apakah file epadmin ada
ls -la /opt/eprints3/bin/

# Output yang diharapkan:
# -rwxr-xr-x 1 eprints eprints  9996 Nov 27 22:43 epadmin
# -rwxr-xr-x 1 eprints eprints  4149 Nov 27 22:43 epindexer
# -rwxr-xr-x 1 eprints eprints  7176 Nov 27 22:43 export
# ... dan file lainnya
```

### 9. Membuat Repository EPrints
```bash
# Switch ke user eprints
sudo su - eprints

# Masuk ke direktori EPrints
cd /opt/eprints3

# Jalankan wizard create repository
./bin/epadmin create pub
```

**Wizard akan menanyakan:**

| Prompt | Contoh Jawaban | Keterangan |
|--------|----------------|------------|
| Archive ID | `myrepo` | ID repository (huruf kecil, tanpa spasi) |
| Archive Name | `My University Repository` | Nama lengkap repository |
| Hostname | `eprints.local` | Domain/hostname (jangan gunakan `localhost`) |
| Port | `80` | Port default HTTP |
| Database Name | `myrepo` | Biasanya sama dengan Archive ID |
| Database Host | `localhost` | Host database |
| Database User | `eprints` | User MySQL yang dibuat sebelumnya |
| Database Password | `password_anda` | Password MySQL |
| Admin Email | `admin@example.com` | Email administrator |
| Admin Username | `admin` | Username untuk login admin |
| Admin Password | `(password)` | Password admin interface |

### 10. Konfigurasi Apache Virtual Host
```bash
# Keluar dari user eprints
exit

# Link konfigurasi Apache
# Ganti 'myrepo' dengan Archive ID yang Anda buat
sudo ln -s /opt/eprints3/archives/myrepo/cfg/apache.conf \
  /etc/apache2/sites-available/eprints-myrepo.conf

# Enable site
sudo a2ensite eprints-myrepo

# Reload Apache
sudo systemctl reload apache2
```

### 11. Konfigurasi Hostname (Optional)

Jika menggunakan hostname lokal seperti `eprints.local`:
```bash
echo "127.0.0.1    eprints.local" | sudo tee -a /etc/hosts
```

### 12. Generate Static Pages dan Views
```bash
sudo su - eprints
cd /opt/eprints3

# Generate static pages
./bin/generate_static myrepo

# Generate views
./bin/generate_views myrepo

# Generate abstracts
./bin/generate_abstracts myrepo

exit
```

### 13. Restart Apache
```bash
sudo systemctl restart apache2
```

---

## Troubleshooting

### Problem 1: Repository APT Tidak Tersedia

**Error:**
```
--2025-11-27 22:18:37--  http://deb.eprints.org/keyFile
ERROR 410: Gone.
```

**Solusi:** Repository EPrints sudah tidak tersedia. Gunakan instalasi dari source (GitHub) seperti yang dijelaskan di atas.

---

### Problem 2: `apt-key` Command Not Found

**Error:**
```
sudo: apt-key: command not found
```

**Penjelasan:** `apt-key` deprecated di Debian 11+ dan Ubuntu 22.04+

**Solusi:** Tidak perlu GPG key untuk instalasi dari source.

---

### Problem 3: Apache2/Const.pm Not Found

**Error:**
```
Can't locate Apache2/Const.pm in @INC
```

**Solusi:**
```bash
sudo apt-get install -y libapache2-mod-perl2
sudo systemctl restart apache2
```

---

### Problem 4: Permission Denied saat Clone

**Error:**
```
fatal: could not create work tree dir '/opt/eprints3': Permission denied
```

**Solusi:**
```bash
# Clone di direktori temporary dulu
cd /tmp
git clone https://github.com/eprints/eprints3.4.git
cd eprints3.4
git checkout tags/v3.4.6

# Copy dengan sudo
sudo cp -r * /opt/eprints3/
sudo chown -R eprints:eprints /opt/eprints3
```

---

### Problem 5: Hostname localhost Warning

**Error:**
```
Warning! Some browsers don't support setting cookies on 'localhost' 
or IP-addresses, please provide a different hostname.
```

**Solusi:** Gunakan hostname seperti:
- `eprints.local`
- `localhost.localdomain`
- Domain nyata jika sudah ada

Tambahkan ke `/etc/hosts`:
```bash
echo "127.0.0.1    eprints.local" | sudo tee -a /etc/hosts
```

---

### Problem 6: Database Connection Failed

**Error:**
```
DBI connect failed: Access denied for user 'eprints'@'localhost'
```

**Solusi:**
```bash
# Reset password MySQL user eprints
sudo mysql -u root

# Di MySQL prompt:
ALTER USER 'eprints'@'localhost' IDENTIFIED BY 'password_baru';
FLUSH PRIVILEGES;
EXIT;
```

---

### Problem 7: Apache Port 80 Already in Use

**Error:**
```
(98)Address already in use: AH00072: make_sock: could not bind to address
```

**Solusi:**
```bash
# Cek proses yang menggunakan port 80
sudo netstat -tulpn | grep :80

# Stop service yang bentrok
sudo systemctl stop nginx  # jika ada nginx
```

---

## Verifikasi

### Cek Instalasi EPrints
```bash
# Cek file EPrints
ls -la /opt/eprints3/bin/

# Cek repository yang dibuat
ls -la /opt/eprints3/archives/

# Cek Apache config
ls -la /etc/apache2/sites-available/ | grep eprints
```

### Cek Service
```bash
# Cek status Apache
sudo systemctl status apache2

# Cek status MariaDB
sudo systemctl status mariadb

# Test Apache config
sudo apache2ctl configtest
```

### Test Akses
```bash
# Test dari command line
curl http://eprints.local/

# Atau
curl http://localhost/
```

**Output yang diharapkan:** HTML dari halaman EPrints

---

## Akses EPrints

Setelah instalasi selesai, akses EPrints melalui browser:

### Public Interface
```
http://eprints.local/
```

### Admin Interface
```
http://eprints.local/cgi/users/home
```

### Login Admin
- **Username:** Sesuai yang dibuat saat wizard
- **Password:** Sesuai yang dibuat saat wizard

---

## Command Ringkasan (Copy-Paste Ready)
```bash
# Full installation script
sudo apt-get update && sudo apt-get upgrade -y

sudo apt-get install -y git apache2 mariadb-server mariadb-client \
  libmariadb-dev perl libxml2-dev libxml-libxml-perl \
  libunicode-string-perl libterm-readkey-perl libmime-lite-perl \
  libdigest-sha-perl libdbd-mysql-perl libxml-parser-perl \
  libxml2-utils libarchive-zip-perl libjson-perl libyaml-perl \
  libsearch-xapian-perl libtex-encode-perl libio-string-perl \
  libdata-uuid-perl imagemagick xpdf antiword elinks poppler-utils \
  libapache2-mod-perl2 libapache2-request-perl libapache2-reload-perl \
  libapache2-sitecontrol-perl libxml-libxslt-perl libfile-mimeinfo-perl \
  liblingua-stem-perl libtext-unidecode-perl

sudo a2enmod perl && sudo a2enmod rewrite && sudo systemctl restart apache2

sudo useradd -m -s /bin/bash eprints

sudo mysql -u root -e "CREATE USER 'eprints'@'localhost' IDENTIFIED BY 'eprints123';"
sudo mysql -u root -e "GRANT ALL PRIVILEGES ON *.* TO 'eprints'@'localhost' WITH GRANT OPTION;"
sudo mysql -u root -e "FLUSH PRIVILEGES;"

sudo mkdir -p /opt/eprints3
sudo chown eprints:eprints /opt/eprints3
sudo chmod 2775 /opt/eprints3

cd /tmp
git clone https://github.com/eprints/eprints3.4.git
cd eprints3.4
git checkout tags/v3.4.6
sudo cp -r * /opt/eprints3/
sudo chown -R eprints:eprints /opt/eprints3

echo "127.0.0.1    eprints.local" | sudo tee -a /etc/hosts

echo "Instalasi dependencies selesai!"
echo "Selanjutnya jalankan: sudo su - eprints"
echo "Lalu: cd /opt/eprints3 && ./bin/epadmin create pub"
```

---

## Catatan Penting

1. **Keamanan Database:** 
   - Ganti password default `eprints123` dengan password yang lebih aman
   - Jangan gunakan password yang sama untuk production

2. **Firewall:**
```bash
   # Jika menggunakan UFW
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
```

3. **SSL/HTTPS:**
   - Untuk production, selalu gunakan HTTPS
   - Install Let's Encrypt:
```bash
   sudo apt-get install certbot python3-certbot-apache
   sudo certbot --apache -d eprints.yourdomain.com
```

4. **Backup:**
   - Backup database secara berkala
   - Backup direktori `/opt/eprints3/archives/`

5. **Update:**
```bash
   cd /opt/eprints3
   git pull origin master
   git checkout tags/v3.4.x  # versi terbaru
```

---
