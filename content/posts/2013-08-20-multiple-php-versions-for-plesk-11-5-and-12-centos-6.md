+++
author = "Alex Bouma"
date = 2013-08-20T18:36:00+01:00
draft = false
slug = "multiple-php-versions-for-plesk-11-5-and-12-centos-6"
tags = ["centos", "guide", "php", "plesk"]
title = "Multiple PHP versions for Plesk 11.5 and 12 (CentOS 6)"
aliases = ["/multiple-php-versions-for-plesk-11-5-and-12-centos-6"]
+++

> **Update 2:** [Plesk 12 supports multiple PHP versions out of the box!](https://alex.bouma.me/finally-plesk-12-supports-multiple-php-versions-out-of-the-box/) 

> **Update 1:** Automated and updated scripts are available for PHP [5.4](http://alex.bouma.me/install-php-5-4-plesk-11-5-12-centos-6/ "Install PHP 5.4 on Plesk 11.5 and 12 (CentOS 6)"), [5.5](http://alex.bouma.me/install-php-5-5-plesk-11-5-12-centos-6/ "Install PHP 5.5 on Plesk 11.5 and 12 (CentOS 6)") and [5.6](http://alex.bouma.me/install-php-5-6-plesk-11-5-12-centos-6/ "Install PHP 5.6 on Plesk 11.5 and 12 (CentOS 6)")

This post will cover the install of multiple PHP versions on Plesk 11.5.

This is tested on a [TransIP](http://www.transip.eu "TransIP") CentOS 6.4 x64 VPS with stock repos + EPEL & REMI. This is written for CentOS but with a bit of imagination this can also work on any other linux OS supported by Plesk.

### Requirements

- Host running Plesk 11.5
- `~100-500 mb` disk space (depending on how many PHP versions you want to add)
- Linux command line experience
- Coffee =]

### Preparation

First off make sure your system in up-to-date with the latest packages and has `wget` installed:

```bash
yum -y update && yum -y install wget
```

Install EPEL repository (dependency of libmcrypt)

```bash
rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
```

Install PHP build-dependencies:

```bash
yum -y install gcc make gcc-c++ cpp kernel-headers.x86_64 libxml2-devel openssl-devel bzip2-devel libjpeg-devel libpng-devel freetype-devel openldap-devel postgresql-devel aspell-devel net-snmp-devel libxslt-devel libc-client-devel libicu-devel gmp-devel curl-devel libmcrypt-devel unixODBC-devel pcre-devel sqlite-devel db4-devel enchant-devel libXpm-devel mysql-devel readline-devel libedit-devel recode-devel libtidy-devel
```

Get the current timezone for later:

```bash
timezone=$(grep -oP '(?<=")w+/w+' /etc/sysconfig/clock)
```

### Install PHP

This applies for basicly any PHP 5.3+ (so also 5.4 & 5.5) but for this example we use PHP 5.3.27.

Download PHP 5.3.27 source:

```bash
wget http://nl1.php.net/get/php-5.3.27.tar.gz/from/this/mirror -O /usr/local/src/php-5.3.27.tar.gz
```

Unpack, configure, build and install PHP:

```bash
# Unpacking
tar xzvf /usr/local/src/php-5.3.27.tar.gz -C /usr/local/src/

# Move to unpacked folder
cd /usr/local/src/php-5.3.27/

# Configure PHP build script
./configure --with-libdir=lib64 --cache-file=./config.cache --prefix=/usr/local/php-5.3.27 --with-config-file-path=/usr/local/php-5.3.27/etc --disable-debug --with-pic --disable-rpath  --with-bz2 --with-curl --with-freetype-dir=/usr/local/php-5.3.27 --with-png-dir=/usr/local/php-5.3.27 --enable-gd-native-ttf --without-gdbm --with-gettext --with-gmp --with-iconv --with-jpeg-dir=/usr/local/php-5.3.27 --with-openssl --with-pspell --with-pcre-regex --with-zlib --enable-exif --enable-ftp --enable-sockets --enable-sysvsem --enable-sysvshm --enable-sysvmsg --enable-wddx --with-kerberos --with-unixODBC=/usr --enable-shmop --enable-calendar --with-libxml-dir=/usr/local/php-5.3.27 --enable-pcntl --with-imap --with-imap-ssl --enable-mbstring --enable-mbregex --with-gd --enable-bcmath --with-xmlrpc --with-ldap --with-ldap-sasl --with-mysql=/usr --with-mysqli --with-snmp --enable-soap --with-xsl --enable-xmlreader --enable-xmlwriter --enable-pdo --with-pdo-mysql --with-pear=/usr/local/php-5.3.27/pear --with-mcrypt --without-pdo-sqlite --with-config-file-scan-dir=/usr/local/php-5.3.27/php.d --without-sqlite3 --enable-intl

# Build
make

# Install
make install

# Create a default php.ini
cp -a /etc/php.ini /usr/local/php-5.3.27/etc/php.ini
# Set the timezone we grabbed earlier
sed -i "s#;date.timezone =#date.timezone = $timezone#" /usr/local/php-5.3.27/etc/php.ini[/cce_bash]
<h3>Register PHP installation with Plesk</h3>
[cce_bash]/usr/local/psa/bin/php_handler --add -displayname "5.3.27" -path /usr/local/php-5.3.27/bin/php-cgi -phpini /usr/local/php-5.3.27/etc/php.ini -type fastcgi -id "fastcgi-5.3.27"
```

And with that you should have a working PHP 5.3.27 visible in your Plesk control panel.

![plesk-php-5-3-27](/img/ghost/plesk-php-5-3-271.jpg)

You can repeat this process for other versions you might want to install by changing 5.3.27 to your disired version (for example 5.4.18 or 5.5.2).
