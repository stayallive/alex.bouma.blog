+++
author = "Alex Bouma"
date = 2014-07-28T23:51:00+01:00
description = ""
draft = false
slug = "enable-passive-ftp-plesk-11-and-12"
tags = ["firewall", "ftp", "plesk"]
title = "Enable Passive FTP on Plesk 11.x & 12.x"
aliases = ["/enable-passive-ftp-plesk-11-and-12"]
+++

FTP passive mode require some or all unprivileged (`1024-65534`) ports to be accessible by the world.

By default Plesk does not add a rule for these ports, so passive mode does not work.

To enable passive mode we need to edit the ProFTPD (FTP server used by Plesk) configuration.

Edit the config file (located here: `/etc/proftpd.conf`) with your favorite shell editor (like nano or vim). Find the `<global>` section and add the following rule between the `<global>` and `</global>` tags:

```raw
PassivePorts 60000 65534
```

This changes will tell ProFTPD to use ports `60000-65534` for passive connections. To apply on CentOS run the following to restart the FTP server:

```bash
/etc/init.d/xinetd restart
```

#### Plesk firewall configuration

Now we will have to add a firewall rule in plesk.

![Adding a custom rule to the Plesk firewall.](/img/ghost/Screen-Shot-2014-07-29-at-02-31-40-284x3001.png)

- Login to your plesk panel
- Go to “Server Management” > “Tools & Settings” > “Security” > “Firewall”
- Click on “Modify Plesk Firewall Rules”
- Go to “Add cutom rule”
- Give it a sensible name, something like “FTP Passive ports”
- Set direction to “Incoming”, action to “Allow”
- Add a port range `60000-65534` for TCP
- Hit “Ok”
- Click “Apply Changes” > “Activate”
- Check if FTP works with passive mode enabled
- Profit!
