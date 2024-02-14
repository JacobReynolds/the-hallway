---
title: a cdn?
layout: default
nav_order: 6
---

# 🛜 CDN

Not exactly a CDN, more of a WAAP, but anytime I say WAAP people starting singing Cardi B lyrics at me. WAAP stands for Web Application and API Protection. The common example would be Cloudflare, but more closely would be Imperva and Akamai. These platforms proxy all of your traffic and perform DDOS protection, bot mitigation, web firewall rule matching, API inventorying, and more. Similar in grandiose size to a bank, but far more in my wheelhouse. At this point I'm about 4 weeks into my start up journey and am getting a bit sick of cold outreach on LinkedIn, so I decided to grace myself with a little technical research for awhile. I knew there was a market for this, I wasn't quite sure what my differentiator would be, but I wanted to see if I could get the basic concepts built first to recharge my batteries a bit.

- Build out an edge network
  - Tried kubernetes and nomad
  - Used nats
- Resolve queries to that edge network
- Provision SSL/TLS certificates on that edge network
- Intercept traffic on that edge network
- Perform WAF and logging
- Dead
