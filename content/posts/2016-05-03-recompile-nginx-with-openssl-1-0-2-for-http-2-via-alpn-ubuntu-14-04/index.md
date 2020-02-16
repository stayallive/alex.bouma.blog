+++
author = "Alex Bouma"
date = 2016-05-03T18:28:00+01:00
description = ""
draft = false
cover = "/img/ghost/http2-ok.jpg"
slug = "recompile-nginx-with-openssl-1-0-2-for-http-2-via-alpn-ubuntu-14-04"
tags = ["ubuntu", "server"]
title = "Recompile NGINX with OpenSSL 1.0.2+ for HTTP/2 via ALPN - Ubuntu 14.04"
aliases = ["/recompile-nginx-with-openssl-1-0-2-for-http-2-via-alpn-ubuntu-14-04"]
+++

Since Chrome is soon [dropping](https://ma.ttias.be/chrome-drops-npn-support-for-http2-alpn-only/) HTTP/2 via NPN we need to support HTTP/2 via ALPN, however... on Ubuntu 14.04 NGINX has been compiled with OpenSSL 1.0.1 which does not support ALPN :( So let's fix that!

First we need to upgrade OpenSSL, since that is not that easy to do ourselfs I use the package repository provided by [Ondřej Surý](https://launchpad.net/~ondrej/+archive/ubuntu/php5/) for originally PHP but also includes a up-to-date OpenSSL package.

```bash
# Add the apt repository
sudo add-apt-repository -y ppa:ondrej/php

# Update the package list
sudo apt-get update

# Upgrade OpenSSL
sudo apt-get upgrade openssl
```

> The following steps assume you have [NGINX 1.9.5+](https://by-example.org/install-nginx-with-http2-support-on-ubuntu-14-04-lts/) already running and have OpenSSL 1.0.2 installed on your system (check with the `openssl version` command).

The commands below should get you with your version of NGINX (1.9.5+) compiled with OpenSSL 1.0.2+. But your mileage may vary.

```bash
# Install package building tools
sudo apt-get install -y dpkg-dev

# (optional) cleanup previous work directory
#sudo rm -R /opt/nginx

# Create a work directory
sudo mkdir /opt/nginx

# Switch to our work directory
cd /opt/nginx

# Get NGINX source files
sudo apt-get source nginx

# Install NIGNX build dependencies
sudo apt-get -y build-dep nginx

# Switch to the source files directory
# You might need to change the version number in your case
cd nginx-1.*

# Build the .deb package files
sudo dpkg-buildpackage -b

# Move back to out work directory where the .deb files are placed
cd /opt/nginx

# Stop NGINX
sudo service nginx stop

# Install the newly build .deb file
sudo dpkg --install nginx_1.*~trusty_amd64.deb

# Start NGINX
sudo service nginx start

# Profit ;)
```
