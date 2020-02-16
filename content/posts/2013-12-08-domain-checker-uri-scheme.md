+++
author = "Alex Bouma"
date = 2013-12-08T16:49:00+01:00
description = "About the Domain Checker for Windows Phone URI scheme."
draft = false
slug = "domain-checker-uri-scheme"
tags = ["app"]
title = "Domain Checker for Windows Phone 8: Custom URI Scheme"
aliases = ["/domain-checker-uri-scheme"]
+++

> This is only supported in version **2.0.0** and higher

- Launch application: `domainchecker:`
- Search availability: `domainchecker:Check?Domain={DOMAIN}`

##### Examples

```raw
# Expected format
# Directly check availabilty
domainchecker:Check?Domain=alexbouma
domainchecker:Check?Domain=alexbouma.me

# "Cluttered" format
# Formats URL and directly check availabilty
domainchecker:Check?Domain=http://alexbouma.me/

# Invalid format
# Waits for user to correct the query
domainchecker:Check?Domain=alexbouma?.me
```

[Change log: Version 2.0.0](/domain-checker-for-windows-phone-8-update-2-0-0/ "Domain Checker for Windows Phone 8: Update 2.0.0")
