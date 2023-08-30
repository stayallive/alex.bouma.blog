---
title: Setup Sentry Relay on Ubuntu
slug: setup-sentry-relay-on-ubuntu-server
author: Alex Bouma
tags:
  - sentry
  - ubuntu
  - guide
date: 2023-08-29T10:00:00+02:00
lastmod: 2023-08-29T10:00:00+02:00
summary: "Learn how to setup Sentry Relay to speed up sending events to Sentry and make it more reliable."
description: "Lean how to setup Sentry Relay to speed up sending events to Sentry and make it more reliable."
keywords:
  - sentry
  - ubuntu
  - guide
draft: false
cover: /img/braden-collum-9HI8UJMSdZA-unsplash.jpg
---

Sentry offers [Relay](https://docs.sentry.io/product/relay/), it's a proxy or relay that sits between your application and Sentry and can be used to filter events, add additional information, and more.
In it's simplest form it can be used to relay events to Sentry (which is why it's called that 😉).

There are a few key benefits to me that make Relay a great addition to your Sentry setup:

- **Speed**: Since you can deploy Relay close to your application (usually on the same host), you can send events to Sentry almost instantly without much networking overhead
- **Reliability**: Relay will queue events when the Sentry instance is unreachable and tries to send them when Sentry is reachable again but while Sentry is down your application can continue to send events to Relay without any issues

Especially for PHP applications Relay is a must-have since PHP is single threaded so every ms spent waiting for Sentry to respond is a ms your application can't do anything else and when Sentry is down or slow your application could be slow as well. 
The PHP SDK (of which I am a maintainer) tries to do as much as it can to minimize the impact of sending events to Sentry but it can only do so much and Relay solves the problem. See also [Improve Response Time](https://docs.sentry.io/platforms/php/performance/#improve-response-time) from the PHP/Laravel/Symfony docs.

There are 3 operating [modes for Relay](https://docs.sentry.io/product/relay/modes/):

- `managed` (only available on Business and Enterprise plans or self-hosted): Relay will request project settings from Sentry, so it can be configured from the UI.
- `static`: Relay is configured with specific [DSN](https://docs.sentry.io/product/sentry-basics/dsn-explainer/)s it will allow, and allow accepts events for those projects.
- `proxy`: Relay will proxy requests to Sentry, and will forward any event you sent to it.

I will focus on the `proxy` mode in this guide since it's the easiest to setup but apart from the specific configuration the setup is the same for all modes.

> _Note: I am assuming a few things in this guide. That the reader knows how to use SSH and can perform basic command line actions. It is also assumed you are installing Relay on a Ubuntu server with `systemd` (the guide was tested on Ubuntu 18.04/20.04/22.04)._

If you'd rather follow the official guide, you can find it [here](https://docs.sentry.io/product/relay/getting-started/), I will focus on running a binary with `systemd` but there is also a Docker option in the official [Getting Started](https://docs.sentry.io/product/relay/getting-started/) guide.

_If you are using [Ploi](https://ploi.io/?referrer=BwZowvI55rM5y9ZVqjdB) to manage your servers you can use [this marketplace script](https://ploi.io/panel/marketplace/305-sentry-relay?referrer=BwZowvI55rM5y9ZVqjdB) that performs the steps outlined below.
However I highly recommend you read through this guide so you know what is happening and how to update Relay in the future._

## Installing Relay

Start by SSH'ing into your server and switching to the root user using `sudo su`.

Now the fun begins, starting with downloading the latest Relay binary from the [Sentry Relay releases page](https://github.com/getsentry/relay/releases).

> _Note: At the time of writing the latest release is `23.8.0`, but you should check the [release page](https://github.com/getsentry/relay/releases) to see if there is a newer version._

You want to download the `relay-Linux-x86_64` release asset since that is the binary for Linux, at the time of writing there is no ARM binary.

```bash
wget -O sentry-relay https://github.com/getsentry/relay/releases/download/23.8.0/relay-Linux-x86_64
```

Next we want to make the binary executable by setting the appropriate permissions:

```bash
chmod +x sentry-relay
```

Now we can move the binary to `/usr/local/bin` so it's available globally:

```bash
mv sentry-relay /usr/local/bin
```

## Configuring Relay

We downloaded the binary and made it available globally, now we need to configure Relay so it knows how to connect to Sentry and what to do with the events it receives.

Start by creating a folder the configuration file, and use `sentry-relay` to generate the `config.yml`. This YAML file contains the configuration for Relay and if you need to edit it later you can find it at: `/etc/sentry-relay/config.yml`.

```bash
mkdir -p /etc/sentry-relay
sentry-relay config init -c /etc/sentry-relay
````

This will ask a few questions on how to setup Relay, you can find the answers below:

```text
Q: Do you want to create a new config?
A: Yes, create custom config

Q: How should this relay operate?
A: Proxy for all events

Q: upstream
A: https://sentry.io/

Q: listen interface
A: 127.0.0.1

Q: listen port
A: 3000

Q: do you want listen to TLS
A: no
```

> _Note: If you run a self-hosted instance replace the upstream answer with the hostname of your self-hosted instance._

## Setup `systemd` service

We downloaded the binary, made it executable, and created a configuration file. We are almost done!

Now we need to setup `systemd` so Relay will start on boot and can be managed via `systemd`.

We first need to create a user for Relay to run as, this is a security measure so Relay can't access any files it shouldn't be able to access.

```bash
useradd -r -s /bin/false sentry-relay
```

This new user needs to be the owner of the configuration file we created earlier:

```bash
chown -Rf sentry-relay:sentry-relay /etc/sentry-relay
```

We also need to create a `systemd` service file for Relay, this will tell `systemd` how to start Relay and what user to run it as.

```bash
cat << EOF > /etc/systemd/system/sentry-relay.service
[Unit]
Description=Sentry Relay
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=3
User=sentry-relay
ExecStart=/usr/local/bin/sentry-relay run --config /etc/sentry-relay

[Install]
WantedBy=multi-user.target
EOF
```

After creating the `systemd` service file we need to reload `systemd` so it knows about the new service file and then enable the service so it will start on boot.

```bash
systemctl daemon-reload
systemctl enable sentry-relay
```

Now we can start the service:

```bash
systemctl start sentry-relay
```

And check the status:

```bash
systemctl status sentry-relay
```

## Sending events to Relay

The last part is updating the DSN in your application to send events to Relay instead of Sentry directly.

Your DSN looks something like this:

**https**://12345abcdef10111213141516171819@**o1.ingest.sentry.io**/2345

To send events to Relay we need to update the DSN to look like this:

**http**://12345abcdef10111213141516171819@**127.0.0.1:3000**/2345

> _Note: We changed the protocol from `https` to `http` and the hostname from `o1.ingest.sentry.io` to `localhost:3000`. If you used different listen interface or port when creating the configuration make sure to re-use those here!_

Now that Relay is running and we have our updated DSN we can send events to it, let's use `sentry-cli` to do that ([installation instructions](https://docs.sentry.io/product/cli/installation/)):

```bash
export SENTRY_DSN='http://12345abcdef10111213141516171819@127.0.0.1:3000/2345'
sentry-cli send-event -m 'A test event using Relay'
```

If everything went well you should see a new event in your Sentry project!

## Updating Relay

Updating Relay is as simple as downloading the latest binary and replacing the old one, you can do this with the following commands:

```bash
# Download the latest binary (replace the version with the latest one)
wget -O sentry-relay https://github.com/getsentry/relay/releases/download/23.8.0/relay-Linux-x86_64

# Make the binary executable
chmod +x sentry-relay

# Move the binary to /usr/local/bin
mv sentry-relay /usr/local/bin

# Restart the Sentry Relay service
systemctl restart sentry-relay
```
