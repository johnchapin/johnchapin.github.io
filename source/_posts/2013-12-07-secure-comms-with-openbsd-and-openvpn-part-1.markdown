---
layout: post
title: "Secure comms with OpenBSD and OpenVPN, part 1"
date: 2013-12-07
comments: true
categories: openbsd openvpn
published: true
---

This is part 1 in a series of posts detailing how I secure my Internet communications using OpenBSD, OpenVPN, and other open-source software.

It should be noted that even these measures are only securing part of my traffic. Everything that exits my VPN endpoint is protected only by whatever protocol-specific security measures are already in place (e.g. HTTPS for web traffic).

### Virtual Private Server

I prefer a virtual private server (VPS) over buying, configuring, shipping, upgrading, and disposing of a Real Machine™. Using QEMU/KVM, the hosting company [TransIP](http://transip.eu) supports VPSs running [OpenBSD](http://www.openbsd.org) 5.3. I'm currently using their "Blade VPS X1" product, which costs €10/month and provides 1GB memory, 50GB storage, 1TB transfer, and importantly, a static IPv4 address.

<!-- more -->

### Why OpenBSD?

I trust OpenBSD to be the most secure operating system available, and I have several years of experience installing and using it. The OpenBSD development team is world-renowned for their commitment to security, and I truly believe that there is no better general-purpose operating system to use as a basis for this project.

TransIP supplies installation files for OpenBSD 5.3. Unfortunately, the OpenBSD project does *not* digitally sign their releases, instead choosing only to provide SHA256 hashes of the installation set files to protect against corruption or tampering. In their own words, ["If the men in black suits are out to get you, they're going to get you."](http://www.openbsd.org/faq/faq3.html#Verify)

With that in mind, when using a VPS, we have no choice but to trust that the provider hasn't tampered with the installation process or tapped the VNC console (before SSH can be enabled) to capture our disk encryption or system passwords.

### OpenBSD Installation with Full Disk Encryption

OpenBSD 5.3 supports full disk encryption, so I followed the steps in Ryan Kavanagh's blog post, ["Setting up full disk encryption in OpenBSD 5.3"](http://ryanak.ca/planet-ubuntu/2013/03/26/Setting-up-full-disk-encryption-in-OpenBSD-5.3.html). His instructions are comprehensive, but I found it useful to re-familiarize myself with both the [OpenBSD installation process](http://www.openbsd.org/faq/faq4.html), and the usage of [fdisk](http://www.openbsd.org/faq/faq14.html#fdisk) in particular.

After setting up the encrypted disk, I proceeded with the standard OpenBSD installation, using sd0 as the installation target, and selecting "whole" when prompted, to use the entire device. I chose to use a custom partition layout, forgoing multiple data partitions in favor of a single large "/" partition.

When given the opportunity to select OpenBSD install packages, I chose only:

- **bsd** - The Kernel
- **base53** - Contains the base OpenBSD system
- **etc53** -  Contains all the files in /etc

This represents the bare minimum OpenBSD system necessary to support an OpenVPN server. Any additional software or services represent additional, unnecessary security risk.

### [Part 2 - OpenVPN Installation and PKI Configuration](http://johnchapin.blogspot.com/2013/12/secure-comms-with-openbsd-and-openvpn_5552.html)
