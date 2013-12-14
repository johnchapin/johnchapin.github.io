---
layout: post
title: "Secure comms with OpenBSD and OpenVPN, part 4"
date: 2013-12-14 11:28:34 +0800
comments: true
categories: openvpn osx
published: false
---

This is part 4 in a series of posts detailing how I'm securing my Internet communications using open-source software. In [part 1](http://johnchapin.blogspot.com/2013/12/secure-comms-with-openbsd-and-openvpn.html), I set up an OpenBSD VPS with full-disk encryption and the minimum OS install necessary to run OpenVPN. [Part 2](http://johnchapin.blogspot.com/2013/12/secure-comms-with-openbsd-and-openvpn-part2.html) covered the installation of OpenVPN and configuring the PKI system. [Part 3](http://johnchapin.blogspot.nl/2013/12/secure-comms-with-openbsd-and-openvpn_11.html) was a walk-through of OpenVPN configuration and actually running the OpenVPN daemon.

It should be noted that even these measures are only securing part of my traffic. Everything that exits my VPN endpoint is protected only by whatever protocol-specific security measures are already in place (e.g. HTTPS for web traffic).

---

### Tunnelblick Installation

On Mac OS X, [Tunnelblick](https://code.google.com/p/tunnelblick/) can be used to connect to the VPN server. It's an open-source application that offers simple, OpenVPN-specific configuration and a convenient graphical interface.

To install, simply download and run the installer.

<!-- more -->

Before configuring the VPN, transfer the CA certificate and the client certificate and key from the VPN server to your client computer:

```
$ scp user@vpn.example.com:~/easy-rsa/keys/ca.crt ~/Desktop/
$ scp user@vpn.example.com:~/easy-rsa/keys/client1.example.com.crt ~/Desktop/
$ scp user@vpn.example.com:~/easy-rsa/keys/client1.example.com.key ~/Desktop/
```

Note that we could have generated the client certificate and key on the client machine, and gone through the certificate signing process without ever transmitting the client's private key over the network.

### Tunnelblick Configuration

Open the Tunnelblick application. On the _Configurations_ tab, click on the + icon in the lower left to add a new configuration. When prompted, select _"I DO NOT have configuration files"_.

<a href="http://imgur.com/7Q69mXG"><img src="http://i.imgur.com/7Q69mXGl.png" title="Hosted by imgur.com"/></a>

Next, select _"Create sample configuration and edit it"_.

<a href="http://imgur.com/1jx0mcs"><img src="http://i.imgur.com/1jx0mcsl.png" title="Hosted by imgur.com"/></a>

This process will create a sample configuration and put it on the Desktop.


<a href="http://imgur.com/NtO3i6R"><img src="http://i.imgur.com/NtO3i6Rl.png" title="Hosted by imgur.com"/></a>


Move the downloaded certificates and key into the configuration folder:

```
$ cd Desktop
$ mv ca.crt Sample\ Tunnelblick\ VPN\ Configuration/
$ mv client1.example.com.* Sample\ Tunnelblick\ VPN\ Configuration/
```

Edit the _config.ovpn_ file. It may already be open in your default text editor. The only changes necessary are:

- Use port 80 instead of 1194
- Replace the dummy server name and PKI file names with valid ones.

Here's a diff of those changes:

``` diff
42c42
< remote vpn.example.com 80
---
> remote my-server-1 1194
89,90c89,90
< cert client1.example.com.crt
< key client1.example.com.key
---
> cert client.crt
> key client.key

```

The Tunnelblick configuration folder can be renamed (in this case, to "Example VPN"). To use this configuration in Tunnelblick, add the _.tblk_ extension to the folder name, and then 
double-click the folder to install the configuration in Tunnelblick.

<a href="http://imgur.com/uKnpOVc"><img src="http://i.imgur.com/uKnpOVcl.png" title="Hosted by imgur.com"/></a>

### Running Tunnelblick

Click on the "Railroad Tunnel" icon in the Mac OS X menu bar, and select the "Connect Example VPN" option.

<a href="http://imgur.com/G0lXEOn"><img src="http://i.imgur.com/G0lXEOnl.png" title="Hosted by imgur.com"/></a>

**That's it** - you're now using Tunnelblick to route your Internet communications through OpenVPN running on a VPS-hosted OpenBSD server.
