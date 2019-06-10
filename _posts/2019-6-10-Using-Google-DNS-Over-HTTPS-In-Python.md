---
layout: post
title: Using Google DNS Over HTTPS In Python
---

I'm a big fan of [DNS Over HTTPS](https://en.wikipedia.org/wiki/DNS_over_HTTPS). Although DNS is a beautiful system, privacy and security was never a consideration when it was built. 

Google DNS are providing a HTTP endpoint to retrieve DNS records over an HTTPS encrypted link. You can send a `GET` request and get back A records, AAAA records, or whatever else you want in a friendly JSON format.

I thought it might be fun to create a Python implementation of this to make resolving hostnames to IP addresses easy in a Python program. I've turned it into a module which you can import and make a really quick request to get an A record:

```
import dnsOverHttps
aRecord = dnsOverHttps.GoogleDNSQuery.getARecordFor("google.com")
print(aRecord) # 216.58.198.174 (or whatever!)
```

Or if you want more records or control then you can create a custom request too.

I use it in a [Python script](https://gist.github.com/lovattj/1f57a38514162e8867564f8db5ff0c84) that generates OpenVPN configuration files that bypass DNS-filtering networks. For example, Network A uses DNS filtering to block `myvpnprovider.com`. So trying to use a configuration profile that connects to, for example `uk1.myvpnprovider` will just fail to connect. But by using Google DNS over HTTPS to get the IP address, the configuration file can be rewritten with the IP address instead of the hostname, and this will then connect.

It can be used in the same way with IKEv2 VPN networks, and I've got another Python script that generates iOS and Mac `.mobileconfig` files that configure an IKEv2 network with the IP address rather than the hostname.

If you want to try the library, check it out [on GitHub](https://github.com/lovattj/python-Google-DNSOverHTTPS-Resolver). There are some examples of how to use it.

Feel free to send in any feedback and have fun!