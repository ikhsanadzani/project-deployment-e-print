# Dependencies and Pre-configuration

```
apt install perl libncurses5 libselinux1 apache2 libapache2-mod-perl2 libxml-libxml-perl \
  libunicode-string-perl libterm-readkey-perl libmime-lite-perl libmime-types-perl libdigest-sha-perl \
  libdbd-mysql-perl libxml-parser-perl libxml2-dev libxml-twig-perl libarchive-any-perl libjson-perl \
  liblwp-protocol-https-perl libtext-unidecode-perl lynx wget ghostscript poppler-utils antiword elinks \
  texlive-base texlive-binaries psutils imagemagick adduser tar gzip unzip libsearch-xapian-perl \
  libtex-encode-perl libio-string-perl python3-html2text make libexpat1-dev libxslt1-dev

```
## installing on Debian, install MariaDB server and client:

```
  apt install mariadb-server mariadb-client libmariadb-dev

```

## create the eprints user

```
adduser eprints

```

## Now add the eprints user to the www-data group and vice-versa

```
usermod -a -G eprints www-data
usermod -a -G www-data eprints

```

# Downloading and Deploying EPrints Source
## EPrints 3.4.x for GitHub

```
apt install git
mkdir /opt/eprints3
chown eprints:eprints /opt/eprints3
chmod 2775 /opt/eprints3
su eprints
git clone https://github.com/eprints/eprints3.4.git /opt/eprints3
cd /opt/eprints3/
git checkout tags/v3.4.7
```

## EPrints 3.4.x from files.eprints.org

```
cd /tmp/
wget https://files.eprints.org/3288/1/eprints-3.4.7.tar.gz
tar -xzvf eprints-3.4.7.tar.gz
```

## Then put in the source code in place:

```
mv eprints-3.4.7 /opt/eprints3
chmod 2775 /opt/eprints3
chown -R eprints:eprints /opt/eprints3
```

## If you want a publications flavoured repository, then also:

```
wget https://files.eprints.org/3288/2/eprints-3.4.7-flavours.tar.gz
tar -xzvf eprints-3.4.7-flavours.tar.gz
mv eprints-3.4.7/flavours/pub_lib /opt/eprints3/flavours
chmod -R g+w /opt/eprints3/flavours/pub_lib
chown -R eprints:eprints /opt/eprints3/flavours/pub_lib
```

## berikutnya sebagai user eprints copy 

```
EPRINTS_PATH/perl_lib/EPrints/SystemSettings.pm.tmpl
```
ke
```
EPRINTS_PATH/perl_lib/EPrints/SystemSettings.pm
```
jika belum ada.

#### maka sekarang seharusnya EPRINTS_PATH sekarang selalu ada di /opt/eprints3

## setting archive

ganti user anda menjadi "eprints"
lalu masuk ke direktori /opt/eprints3

lalu jalankan command 
```
bin/epadmin create pub
```

**maka wizard akan menanyakan**

| Pertanyaan | Jawaban |
|------------|---------|
| Archive ID? | `Eprints1` |
| Configure vital settings? | `yes` |
| Hostname? | `eprints.local` |
| Webserver Port? | `80` |
| Alias? | *[Tekan Enter]* |
| Path? | `/` |
| HTTPS Hostname? | *[Tekan Enter]* |
| Administrator Email? | `dimasyogairtanto@gmail.com` *(Gunakan Email Anda)* |
| Archive Name? | `Test Repository` |
| Organisation Name? | `UIN J` |
| Configure database? | `yes` |
| Database Name? | `Eprints1` |
| MySQL Host? | `localhost` |
| MySQL Port? | `#` *(Untuk No Setting)* |
| MySQL Socket? | `#` *(Untuk No Setting)* |
| Database User? | `Eprints1` |
| Database Password? | *[Masukkan Password]* |
| Database Engine? | `InnoDB` |
| Write these database settings? | `yes` |
| Create database 'Eprints1'? | `yes` |
| Database Superuser Username? | `root` |
| Database Superuser Password? | *[Password root MySQL]* |
| Create database tables? | `yes` |
| Create an initial user? | `yes` |
| Enter a username? | `bagas` |
| Select a user type? | `admin` |
| Enter Password? | *[Buat Password Admin]* |
| Email? | `wicaksonob208@gmail.com` *(Gunakan Email Anda)* |
| Build static web pages? | `yes` |
| Import LOC subjects and sample divisions? | `yes` |
| Update Apache config files? | `yes` |


``bash
# Keluar dari user eprints
exit

# Link konfigurasi Apache
# Ganti 'myrepo' dengan Archive ID yang Anda buat
sudo ln -s /opt/eprints3/archives/myrepo/cfg/apache.conf \
  /etc/apache2/sites-available/eprints-myrepo.conf

# Enable site
sudo a2ensite eprints-myrepo

# Reload Apache
```
sudo systemctl reload apache2
```
### Generate Static Pages dan Views
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

### Restart Apache
```bash
sudo systemctl restart apache2
```

### akses eprints

gunakan 
```
eprints.local (sesuaikan dengan hostname di wizard)
```
