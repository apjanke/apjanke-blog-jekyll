---
id: 279
title: Setting up ZNC on Ubuntu 20.04
date: 2021-01-19T02:55:00-05:00
author: apjanke
layout: post
categories:
  - Computers
tags:
  - Linux
  - Ubuntu
  - ZNC
  - IRC
---

ZNC is provided as a package in Ubuntu, so you don't have to install it from source. However, it installs it system-wide, while ZNC usually expects to be running from the home directory of a user (sometimes a `znc` user.) So most of the instructions out on the web for configuring ZNC on Linux won't work in this scenario. Here's how to do it.

These instructions should work on Debian, too, since Ubuntu is a Debian derivative.

All the steps here should be done as `root` or with `sudo`.

## Set up the host

I like to create a `znc` DNS alias for the machine I have running ZNC, so I can point to it as `znc.mydomain.com`, keeping the service distinct from the particular machine it's running on.

## Install ZNC

Installing ZNC itself is just this:

```bash
apt install znc
```

But you'll probably want the development tools, too, which are needed to build ZNC modules:

```bash
apt install znc-dev znc-python
```

Instead of installing to someone's `~/.znc` directory, this installs ZNC to `/var/lib/znc`. Its code and data files go directly in that directory, instead of a `.znc` subdirectory under it.

You can start up ZNC with `service znc start` and check its status with `service znc status`.

## Setting up certificates

You'll want to use SSL with your ZNC, for which you'll need certificates.

If you already have an SSL certificate for the machine, you can use that. Otherwise, generate one using LetsEncrypt:

```bash
service znc stop
service nginx stop  # If you have nginx installed
certbot certonly
```

When it prompts you for the domain names to make the certificate for, include both your machine's primary name, and `znc.mydomain.com`.

Then update ZNC's certificate file and fire it back up:

```bash
cat /etc/letsencrypt/live/myhost.mydomain.com/{privkey,cert,chain}.pem > /var/lib/znc/znc.pem
service znc start
```

When you need to renew your certificate later, you can probably just do `certbot renew` instead of `certbot certonly`.

## Configuring ZNC

The configuration file is at `/var/lib/znc/configs/znc.conf`. Edit that to set up ZNC. In particular, you'll probably want to enable `SSL = true` in the `Listener` section, and you'll need to point `SSLCertFile` at the `/var/lib/znc/znc.pem` you generated above.

Add `LoadModule = <modulename>` for any modules you've added.

At this point, the other guides to configuring ZNC out on the web should get you going.

## Installing modules

Some ZNC modules are available as Ubuntu packages. Install them using `apt`.

```bash
apt install znc-backlog
```

But some ZNC modules, like `znc-playback`, are not available in the Ubuntu package repository, so you need to build them yourself from source. See above for how to install the ZNC dev tools.

Download the source distribution for the module to `/var/lib/znc/src-modules/<module-name>`, unpack it, `cd` into the result source directory, and run `make` or `znc-buildmod <module>.cpp`. Then copy the resulting `.so` file to `/var/lib/znc/modules`.
