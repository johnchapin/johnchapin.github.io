---
layout: post
title: "Secure comms with OpenBSD and OpenVPN, part 5"
date: 2013-12-15 13:21:11 +0800
comments: true
categories: openvpn openbsd ios
published: false
---

This is the fifth and final part in a series of posts detailing how I’m securing my Internet communications using open-source software.

[Part 1][part1] - Set up of an OpenBSD VPS with full-disk encryption and the minimum OS install necessary to run OpenVPN.  
[Part 2][part2] - Installation of OpenVPN and configuring the PKI system.  
[Part 3][part3] - OpenVPN configuration and actually running the OpenVPN daemon.  
[Part 4][part4] - Installation and configuration of the Tunnelblick OpenVPN client on Mac OS X.

---

### OpenBSD 5.4

A few weeks after I set up and configured the VPN system on OpenBSD 5.3, a [new version of OpenBSD](http://www.openbsd.org/54.html) was released. This isn't too surprising of course, as the OpenBSD team releases every 6 months!

<!-- more -->

![](http://www.openbsd.org/images/Puffia.jpg "OpenBSD 5.4 Puffia")

Here are some relevant changes that I've noticed:

**The OpenVPN package has been updated to version 2.3.1p2** - Here's the [changelog](https://community.openvpn.net/openvpn/wiki/ChangesInOpenvpn23#OpenVPN2.3.0) from the previous 2.2.x version. Of particular note, OpenVPN now fully supports IPv6.

**The easy-rsa package is no longer a part of OpenVPN** - This is a Good Thing™, as the version of easy-rsa included in the OpenVPN package on OpenBSD 5.3 is incomplete and difficult to reconcile with the [official easy-rsa package](https://github.com/OpenVPN/easy-rsa). The new easy-rsa package tracks version 2.2.0, and includes the _whichopenssl_ script that was missing from the previous packaged version.

### iOS Client

There is an excellent and free [iOS application for OpenVPN](https://itunes.apple.com/app/openvpn-connect/id590379981). Configuration is simple, just build an OpenVPN client configuration (see [Part 4][part4]), and transfer the files to the OpenVPN application on your iOS device using iTunes.

Here's a screenshot of the OpenVPN client in action (with my actual VPN IP address blanked out):

![](http://i.imgur.com/TsUwTfpl.jpg "OpenVPN iOS Application")

### Parting Thoughts

I've added the following paragraph to the introduction of each part of this series:

> _It should be noted that even these measures are only securing part of my traffic. Everything that exits my VPN endpoint is protected only by whatever protocol-specific security measures are already in place (e.g. HTTPS for web traffic)._

I can't stress enough how important it is to understand the limitations of a VPN and the risk inherent in communicating over the Internet. Because the Internet is a shared, public medium, communications channels that use it have an inherent level of exposure that can never be eliminated.

With that concern in mind, I have to admit that my paranoia has practical limits. To a certain extent this series documents the risks I'm willing to accept, like:

- Using a Virtual Private Server hosted in a third-party country.
- Installing the operating system on that VPS using installation media provided by the hosting company.
- Using closed, commericial hardware (a Macbook Air and an iPhone 4S), running closed-source, commericial operating systems (OS X and iOS).
- Storing passwords and credentials for all the above in a closed-source password management tool.

### The Bottom Line

**All of the technical measures in the world are unlikely to block a determined effort to access your communications.** The solution outlined in this series should be considered a deterrent to casual snooping, and nothing more.

[Part 1][part1], [Part 2][part2], [Part 3][part3], [Part 4][part4], [Part 5][part5]

[part1]:/blog/2013/12/07/secure-comms-with-openbsd-and-openvpn-part-1/
[part2]:/blog/2013/12/09/secure-comms-with-openbsd-and-openvpn-part-2/
[part3]:/blog/2013/12/11/secure-comms-with-openbsd-and-openvpn-part-3/
[part4]:/blog/2013/12/14/secure-comms-with-openbsd-and-openvpn-part-4/
[part5]:/blog/2013/12/15/secure-comms-with-openbsd-and-openvpn-part-5/