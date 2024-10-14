---
title: Setting up a Tor Server
author: mashhood
date: 2024-10-02 17:30:00 +0800
categories: [Anonymity]
tags: [Anonymity, Tor]
media_subpath: '/assets/img/posts/20241002/'
image:
  path: onion-tor.webp
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
---

Tor (The Onion Router) is a network that enables anonymous communication. This guide will walk you through setting up a Tor server with Nginx on a Debian system, including how to customize the default web page.

## Update Your System

First, ensure your system is up to date:

```bash
sudo apt update && sudo apt upgrade -y
```

## Install Tor and Nginx

Install Tor and Nginx using the following command:

```bash
sudo apt install tor nginx -y
```

Then enable and start both Tor and Nginx

```bash
sudo systemctl enable nginx
sudo systemctl enable tor
sudo systemctl start nginx
sudo systemctl start tor
```

## Configure Tor

Now edit the Tor configuration file:

```bash
sudo nano /etc/tor/torrc
```

Uncomment or add the following lines at the end of the file:

```
HiddenServiceDir /var/lib/tor/hidden_service/
HiddenServicePort 80 127.0.0.1:80
```

Save and exit the file (Ctrl+X, then Y, then Enter).

## Restart Tor Service

Restart the Tor service to apply the changes:

```bash
sudo systemctl restart tor
```

## Get Your .onion Address

Your .onion address is now generated. You can find it by running:

```bash
sudo cat /var/lib/tor/hidden_service/hostname
```

You will have a domain name like below

```
qd4axpacwmfx7zg7abdssxrhmikrg66gsgamxd6vr4ms2fmdzvjqq2yd.onion
```

Make note of this address, as it's how users will access your hidden service.
Enter this address to the tor browser and you will get the default webpage of nginx

![Desktop View](nginx-default.webp)

## Customize Your Website

You have two options to quickly customize your website:

### Edit the default index.html

Replace the default index.html with your desired content:

```bash
sudo nano /var/www/html/index.html
```

Add your custom HTML content. For example:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Tor Hidden Service</title>
</head>
<body>
    <h1>Welcome to My Tor Hidden Service</h1>
    <p>This is a custom web page served over the Tor network.</p>
</body>
</html>
```

### Download a static page from GitHub

Alternatively, you can download a pre-made static website from a GitHub repository:

First, remove the existing content in the `/var/www/html` directory:

 ```bash
 sudo rm -rf /var/www/html/*
 ```

Then, clone the desired GitHub repository into the `/var/www/html` directory:

```bash
sudo git clone https://github.com/rzlwbo1/ViolaOnlineStore /var/www/html/
 ```

Now, when you access your Tor hidden service, you'll see the downloaded page instead of the default Nginx page.

![Desktop View](onion-2.webp)
