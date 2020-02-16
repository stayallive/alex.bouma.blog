+++
author = "Alex Bouma"
date = 2013-12-08T16:49:00+01:00
description = "What is new in version 2.0.0 of the Domain Checker Windows Phone app."
draft = false
slug = "domain-checker-for-windows-phone-8-update-2-0-0"
tags = ["app"]
title = "Domain Checker for Windows Phone 8: Update 2.0.0"
aliases = ["/domain-checker-for-windows-phone-8-update-2-0-0"]
+++

> [View in Windows Phone Store](http://www.windowsphone.com/s?appid=de519403-65f0-4455-9e22-3376493a04f0 "Windows Phone Store - Domain Checker")

What is new in version **2.0.0**?

- Complete rewrite & redesign of the app and it’s back-end
- Now supports 256 top-level domains[^n]
- Implemented: SSL for requests to back-end
- Implemented: Ability to mail WHOIS record
- Implemented: Ability to lookup DNS records (Supported types: A, AAAA, CNAME, MX, NS, SOA)
- Implemented: Custom URI scheme (See: [Domain Checker for Windows Phone 8: Custom URI Scheme](http://alexbouma.me/domain-checker-uri-scheme/ "Domain Checker for Windows Phone 8: Custom URI Scheme"))
- Implemented: Smart input field, you can throw anything at it and it will try to see a domain in it

#### Screenshots

![Domain Checker v2: Screenshots #1](/img/ghost/domain-checker-screens-1.jpg)

![Domain Checker v2: Screenshots #2](/img/ghost/domain-checker-screens-2.jpg)

#### API Status

View the [Statuspage](http://domainchecker.statusapp.cc/).

[^n]: Some TLDs do not support WHOIS lookup