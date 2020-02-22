+++
author = "Alex Bouma"
date = 2016-09-10T20:41:00+01:00
description = "Guide on how to secure a NGINX vhost with a Let's Encrypt certificate."
draft = false
cover = "/img/ghost/ssl-padlock.jpg"
slug = "setting-up-lets-encrypt-with-nginx-on-ubuntu"
tags = ["server", "ubuntu", "security", "linux"]
title = "Setting up Let's Encrypt with NGINX on Ubuntu"
aliases = ["/setting-up-lets-encrypt-with-nginx-on-ubuntu"]
+++

This guide is more a reference to myself how to setup a fresh auto-renewing certificatie on a Ubuntu (any Linux distro supported by [certbot](https://github.com/certbot/certbot) should work though) box.

_**Note:** All commands should run as the root user so switch to the root user before starting (by running `sudo su` or `su -` depending on your setup)._

#### 0. Setup dependencies

We need `git` to download [certbot](https://github.com/certbot/certbot) from GitHub.

```bash
# Update the package repositories to install the latest
apt-get update

# Install git
apt-get install git
```

#### 1. Setup Let's Encrypt' certbot

```bash
# Switch to a working directory
cd /opt

# Clone the certbot repository into the certbot folder
git clone https://github.com/certbot/certbot

# Create a directory to hold our configuration file(s)
mkdir /etc/certbot

# Create a directory to hold certbot validation files
mkdir -p /var/www/letsencrypt
```

#### 3. Setup a certbot configuration file

```bash
# Start a new file in the configuration directory we just created
nano /etc/certbot/domain.com.conf
```

Add the following contents to this file:

```ini
# Use the webroot authenticator. 
authenticator = webroot

# Use the following path for the webroot authenticator to use
webroot-path = /var/www/letsencrypt

# Generate certificates for the specified domains, add multiple domains by seperating them with a comma
domains = domain.com, www.domain.com

# Register certs with the following email address
email = your@email.com

# Use a 4096 bit RSA key instead of 2048
rsa-key-size = 4096
```

Replace the domain and email value with your own ofcourse :)

#### 4. Edit your domains NGINX config

Open up your nginx vhost configuration file and add the following location block:

```nginx
server {
    listen 80;

    server_name domain.com;

    location ^~ /.well-known {
        allow all;
        alias /var/www/letsencrypt/.well-known;
    }

    # ... snip ... #
}
```

Add the location block in the http server block, or if you already have valid certificate you can also place it in your https server block.

Apply the changes by restarting NGINX: `service nginx restart` (ofcourse after you checked if your configuration is valid by running `service nginx configtest`).

##### 5. Request the certificate

Execute the following command and follow the onscreen instructions to have the certificate being issued.

```bash
/opt/certbot/letsencrypt-auto certonly -c /etc/certbot/domain.com.conf
```

#### 6. Configure NGINX to use the certificate

Change the vhost configuration to something like the following:

```nginx
# Redirect to HTTPS
server {
    listen 80;

    server_name domain.com

    location ^~ /.well-known {
        allow all;
        alias /var/www/letsencrypt/.well-known;
    }

    return 301 https://domain.com$request_uri;
}

# HTTPS server block
server {
    listen 443 ssl;

    server_name domain.com;

    ssl                 on;
    ssl_certificate     /etc/letsencrypt/live/domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain.com/privkey.pem;

    location ^~ /.well-known {
        allow all;
        alias /var/www/letsencrypt/.well-known;
    }

    # ... snip ... #
}
```

Apply the changes by restarting NGINX: `service nginx restart` (ofcourse after you checked if your configuration is valid by running `service nginx configtest`).

#### 7. Check if everything is working

Visit your domain to see if the browser adds the green padlock and run your site through [SSL Labs](https://www.ssllabs.com/ssltest/) to validate all is correct and secure.

**Note:** _I am focussing on the minimal changes to make the certificate work and enable HTTPS. However there are a lot of settings and considerations to make it actually secure and recieve the A+ rating on [SSL Labs](https://www.ssllabs.com/ssltest/). For more info consult:_ [https://raymii.org/strong-ssl-security-on-nginx](https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html)

#### 8. Auto renew the certificate

For this we are going to use a cron script that runs each month and updates our certificate and restarts NGINX.

```bash
# Create a new monthly cron file
nano /etc/cron.monthly/renew-ssl-certificates
```

Add the following contents to that file:

```bash
#!/bin/bash

/opt/certbot/letsencrypt-auto certonly -c /etc/certbot/domain.com.conf --renew-by-default

service nginx restart
```

Don't forget to change `/etc/certbot/domain.com.conf` to whatever your own config file is called.

The last step is to make the renew cron executable, for that run:

```bash
chmod +x /etc/cron.monthly/renew-ssl-certificates
```

#### 9. Profit!

You now have a _free_ Let's Encrypt certificate up and running that auto renews without you having to lift a finger :) Great!
