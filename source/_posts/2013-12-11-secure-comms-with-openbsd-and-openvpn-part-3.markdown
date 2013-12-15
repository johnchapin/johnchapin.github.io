---
layout: post
title: "Secure comms with OpenBSD and OpenVPN, part 3"
date: 2013-12-11
comments: true
categories: openbsd openvpn
published: true
---

This is part 3 in a series of posts detailing how I'm securing my Internet communications using open-source software.

[Part 1][part1] - Set up of an OpenBSD VPS with full-disk encryption and the minimum OS install necessary to run OpenVPN.  
[Part 2][part2] - Installation of OpenVPN and configuring the PKI system.

It should be noted that even these measures are only securing part of my traffic. Everything that exits my VPN endpoint is protected only by whatever protocol-specific security measures are already in place (e.g. HTTPS for web traffic).

---

### Configuring OpenVPN

With the OpenVPN package installed and the PKI components in place, configuring and running the actual server software is straightforward. The OpenVPN package includes a sample server configuration file that makes a good starting point.

<!-- more -->

Make an OpenVPN configuration directory in _/etc_, and add a copy of the sample configuration, the CA certificate, the VPN server certificate and private key, and the Diffie-Hellman parameters:

``` bash
# As root...
$ mkdir /etc/openvpn

# Copy the sample configuration
$ cp /usr/local/share/examples/openvpn/sample-config-files/server.conf /etc/openvpn/server.conf

# Copy PKI materials from the easy-rsa working directory
$ cp easy-rsa/keys/ca.crt /etc/openvpn/ca.crt
$ cp easy-rsa/keys/dh4096.pem /etc/openvpn/dh4096.pem
$ cp easy-rsa/keys/vpn.example.com.crt /etc/openvpn/vpn.example.com.crt

# Make a private directory for the server key
$ mkdir /etc/openvpn/private
$ chmod 500 /etc/openvpn/private

# Move the server key from the easy-rsa working directory
$ mv easy-rsa/keys/vpn.example.com.key /etc/openvpn/private/vpn.example.com.key
$ chmod 400 /etc/openvpn/private/vpn.example.com.key
```

I made the following changes to the default configuration:

- Run the server on port 80, so it can be reached from networks that might restrict or firewall outbound traffic to other ports.
- Use the *tun0* device.
- Enable *redirect-gateway*, to force all client traffic through the VPN.
- Push DNS server information addresses to clients (OpenDNS, in this case).
- Only allow 2 clients at a time (a laptop and a mobile device).

Here's the diff of the _server.conf_ changes:

``` bash
$ diff /etc/openvpn/server.conf  /usr/local/share/examples/openvpn/sample-config-files/server.conf
32c32
< port 80
---
> port 1194
53c53
< dev tun0
---
> dev tun
78,80c78,80
< ca /etc/openvpn/ca.crt
< cert /etc/openvpn/vpn.example.com.crt
< key /etc/openvpn/private/vpn.example.com.key  # This file should be kept secret
---
> ca ca.crt
> cert server.crt
> key server.key  # This file should be kept secret
87c87
< dh /etc/openvpn/dh4096.pem
---
> dh dh1024.pem
187c187
< push "redirect-gateway def1 bypass-dhcp"
---
> ;push "redirect-gateway def1 bypass-dhcp"
195,196c195,196
< push "dhcp-option DNS 208.67.222.222"
< push "dhcp-option DNS 208.67.220.220"
---
> ;push "dhcp-option DNS 208.67.222.222"
> ;push "dhcp-option DNS 208.67.220.220"
255c255
< max-clients 2
---
> ;max-clients 100
```

Note that in this configuration, the server doesn't need to store individual client certificates. The server will only accept clients whose certificates were signed by the master CA certificate (the same one that signed the server certificate).

### Configuring OpenBSD for OpenVPN

The OS requires a few additional tweaks to run OpenVPN.

Turn on packet forwarding:

``` bash
$ sysctl -n net.inet.ip.forwarding=1
```

Add the following line to _pf.conf_ to perform Network Address Translation on VPN connections (the 10.8.0.0/24 block is distributed via DHCP to OpenVPN clients):

```
pass out on em0 from 10.8.0.0/24 to any nat-to (em0)
```

### Running the OpenVPN daemon

``` bash
$ /usr/local/sbin/openvpn --daemon --config /etc/openvpn/server.conf

# Verify that it's actually running:
$ ps -aux | grep openvpn
_openvpn  2900  0.0  0.3  1720  3424 ??  Ss    Mon03PM   21:49.15 /usr/local/sbin/openvpn --daemon --config /etc/openvpn/server.conf
```

Log output will appear in _/var/log/daemon_. At this point, there are no clients, so it's enough that the daemon starts without errors.

### [Part 4 - Client configuration][part4]

[Part 1][part1], [Part 2][part2], [Part 3][part3], [Part 4][part4], [Part 5][part5]

[part1]:/blog/2013/12/07/secure-comms-with-openbsd-and-openvpn-part-1/
[part2]:/blog/2013/12/09/secure-comms-with-openbsd-and-openvpn-part-2/
[part3]:/blog/2013/12/11/secure-comms-with-openbsd-and-openvpn-part-3/
[part4]:/blog/2013/12/14/secure-comms-with-openbsd-and-openvpn-part-4/
[part5]:/blog/2013/12/15/secure-comms-with-openbsd-and-openvpn-part-5/
