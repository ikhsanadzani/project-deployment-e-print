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




