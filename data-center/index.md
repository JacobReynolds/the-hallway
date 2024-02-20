---
title: data center
layout: default
description: how many licks does it take to get to the center of a data center?
order: -11
icon: server
---

# 🏙️ Data Center

This is a brief aside because I found some cool technology and wanted to remember it. This wasn't really as much about building a physical data center as it was about managing a virtual data center. In relation to my research around [CDNs](../cdn/) I was thinking about how I would deal with managing physical servers, and maybe even letting people run compute on them. While looking at solutions, I realized there's a whole market of data center management software. It's pretty obvious when you think about it, but whatever. Some of those offerings are:

- [https://www.openstack.org/](https://www.openstack.org/)
- [https://harvesterhci.io/](https://harvesterhci.io/)
- [https://maas.io/](https://maas.io/)
- [https://www.proxmox.com/en/](https://www.proxmox.com/en/)

All of those solutions solve slightly different problems and have different licensing/pricing models. Openstack is what got me really excited, but after reading reviews about it the [general consensus](https://news.ycombinator.com/item?id=33274744) seems to be it's a nightmare to maintain and the documentation they have available is miserable.

Harvester looks really cool, but it actually runs on top of kubernetes, which is a nightmare I'm not ready to repeat. Proxmox is pretty popular and many people use it for managing homelabs, but it's closed source and pretty expensive.

MAAS was the last provider I looked at and probably the most likely one I would use if I decided to pursue this path. I wasn't able to find too much online discourse about it, but canonical is reputable, their docs seemed good and they have a shit load of functionality. I played around with the idea of grabbing some bare metal and trying it out. I was already familiar with some bare metal hosting providers and [Equinix](https://equinix.com) was my favorite. However when looking at pricing, their bare metal hosting is more expensive (?!) than comparable Droplets at Digital Ocean. I always thought we were paying extra for using a VPS, but unless you physical own the bare metal you're running on it seems to even out in this case. So I'll put a pin in this unless I can find some far cheaper bare metal.
