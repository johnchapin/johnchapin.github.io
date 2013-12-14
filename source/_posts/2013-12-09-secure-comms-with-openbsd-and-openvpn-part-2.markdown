---
layout: post
title: "Secure comms with OpenBSD and OpenVPN, part 2"
date: 2013-12-09
comments: true
categories: openbsd openvpn
published: true
---

This is part 2 in a series of posts detailing how I'm securing my Internet communications using open-source software. In [part 1](http://johnchapin.blogspot.com/2013/12/secure-comms-with-openbsd-and-openvpn.html), I set up an OpenBSD VPS with full-disk encryption and the minimum OS install necessary to run OpenVPN.

It should be noted that even these measures are only securing part of my traffic. Everything that exits my VPN endpoint is protected only by whatever protocol-specific security measures are already in place (e.g. HTTPS for web traffic).

---

### OpenVPN Installation

Installing and configuring the OpenVPN package can seem daunting at first, but given a relatively simple VPN architecture (many clients, one server), the setup is straightforward. Many of the steps below are cribbed from the [OpenVPN section of "Building VPNs on OpenBSD"](http://www.kernel-panic.it/openbsd/vpn/vpn4.html), which is 4 years old but still informative.

<!-- more -->

Install the OpenVPN package from the installation media or an official OpenBSD mirror site. The OpenBSD FAQ has [instructions](http://www.openbsd.org/faq/faq15.html#Easy) for setting up the package system.

``` bash
# Dutch mirror site
$ export PKG_PATH="http://ftp.nluug.nl/pub/OpenBSD/5.3/packages/`machine -a`"
$ pkg_add openvpn
```

Next, make a copy of the easy-rsa directory:

``` bash
$ cp -R /usr/local/share/examples/openvpn/easy-rsa/2.0 easyrsa
$ cd easyrsa
```

### Public Key Infrastructure (PKI) Configuration

The version of easy-rsa that's included with OpenVPN on OpenBSD 5.3 is missing the _whichopenssl_ script, so in the _vars_ file, the _KEY\_CONFIG_ line must be edited in addition to the other _KEY\*_ lines. Here is a diff with my changes:

``` bash
$ diff vars /usr/local/share/examples/openvpn/easy-rsa/2.0/vars
29c29
< export KEY_CONFIG="$EASY_RSA/openssl-1.0.0.cnf"
---
> export KEY_CONFIG=`$EASY_RSA/whichopensslcnf $EASY_RSA`
53c53
< export KEY_SIZE=4096
---
> export KEY_SIZE=1024
64,71c64,72
< export KEY_COUNTRY="ZZcountry"
< export KEY_PROVINCE="ZZprovince"
< export KEY_CITY="ZZcity"
< export KEY_ORG="example"
< export KEY_EMAIL=
< export KEY_CN=
< export KEY_NAME=
< export KEY_OU=
---
> export KEY_COUNTRY="US"
> export KEY_PROVINCE="CA"
> export KEY_CITY="SanFrancisco"
> export KEY_ORG="Fort-Funston"
> export KEY_EMAIL="me@myhost.mydomain"
> export KEY_EMAIL=mail@host.domain
> export KEY_CN=changeme
> export KEY_NAME=changeme
> export KEY_OU=changeme
```

Because they are likely to change for each certificate generated, the _KEY\_EMAIL_, _KEY\_CN_, _KEY\_NAME_, and _KEY\_OU_ values can be removed from the file.

After editing the vars file, source it and run these scripts to setup the PKI system:

``` bash
$ . vars
$ ./clean-all
$ ./build-dh
$ ./pkitool --initca
```

Generate a certificate for the VPN server:

``` bash
$ ./pkitool --server vpn.example.com
```

And one or more client certificates:

``` bash
# Can also supply KEY_CN, KEY_OU, and KEY_EMAIL
$ KEY_NAME=client1 ./pkitool client1.example.com
```

The _keys_ directory should now be full of certificates, keys, and signing requests:

``` bash
$ ls keys/
01.pem
02.pem
ca.crt
ca.key
dh4096.pem
serial
vpn.example.com.crt
vpn.example.com.csr
vpn.example.com.key
client1.example.com.crt
client1.example.com.csr
client1.example.com.key
...
```

### [Part 3 - Running OpenVPN on OpenBSD](http://johnchapin.blogspot.nl/2013/12/secure-comms-with-openbsd-and-openvpn_11.html)
