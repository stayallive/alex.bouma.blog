+++
author = "Alex Bouma"
date = 2013-11-11T21:24:00+01:00
description = "I ran in to this problem on my production server and was finally able to find a solution… in a KB from Plesk =]"
draft = false
slug = "bad-gateway-nginx-502-when-exporting-database-from-phpmyadmin-on-plesk-11-5"
tags = ["guide", "nginx", "php"]
title = "Bad Gateway (NGINX 502) when exporting database from phpMyAdmin on Plesk 11.5"
aliases = ["/bad-gateway-nginx-502-when-exporting-database-from-phpmyadmin-on-plesk-11-5"]
+++

I ran in to this problem on my production server and was finally able to find a solution… in a [KB](http://kb.parallels.com/en/117480 "Plesk KB: Could not export database using PHPMyAdmin: \"Nginx 502 Bad Gateway\"") from Plesk =]

#### Solution

1. Set `apc.enabled = 0` (default = `1`) in the `/usr/local/psa/admin/conf/php.ini` configuration file
2. Restart `sw-engine` by running `service sw-engine restart`
