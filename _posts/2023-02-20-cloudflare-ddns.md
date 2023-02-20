---
title: "Running a Cloudfare DDNS service for your home VPN"
author: "Alex Oswald"
date: 2023-02-20
---


In this post I'm going to show you how to use the Cloudflare DDNS service I wrote to keep my constantly changing public IP
updated with my VPN's DNS record in Cloudflare. I use a NetGear Orbi router at home, and it supports using
[No-IP](https://www.noip.com/) as a free DDNS service. The problem with using this free service, is you have to login and manually
confirm your free hostname monthly. While this may be okay for those that don't own any domain names, it isn't something I wanted to
keep doing manually.


## Why do we need a DDNS service?

The reason for this, is because your ISP typically changes your public IP address every so often, unless you pay for a static IP,
which is only common for businesses. And typically you want your IP address assigned to a hostname, so you can connect your VPN to
something like, `vpn.example.com`, instead of the actual IP address. If you don't, you would need to update your VPN profile on each
device to point to the new IP address each time it changes. So how do we get around this? Dynamic DNS services, like the one my Orbi
router supports, let you configure a free hostname, such as `vpn.mynetgear.com` to point to your public IP, and automatically update
the DNS record each time the router detects a new public IP. But this comes at the cost of doing the manual hostname confirmation
each month to keep it free.

Because I love to automate things, already own some domain names, and run a Docker swarm in my home lab, I wanted to put together
a simples solution to the problem instead of finding another solution online. While I found a few that would work, they didn't do
exactly what I wanted, and I wouldn't of learned as much. So here we are...


## What is CloudfareDDNS

See the CloudflareDDNS repository on github: [https://github.com/alex-oswald/CloudflareDDNS](https://github.com/alex-oswald/CloudflareDDNS)

CloudfareDDNS is a .NET application that runs in a container. It creates a background service that runs a few commands on an interval,
defaulting to every 15 minutes. You can run it locally on any machine. I run mine as a stack on a Raspberry Pi swarm since setup is
simple with [Portainer](https://www.portainer.io/).

The [DDNSBackgroundService](https://github.com/alex-oswald/CloudflareDDNS/blob/main/CloudflareDDNS/DDNSBackgroundService.cs) will do
the following:

1. Use a DNS request to get your public IP address
2. Get a list of Zones from your Cloudflare account
3. Get the id of the Zone you have specified via config from the list of Zones
4. Get the DNS records for the Zone
5. If the DNS record you specified via config does not exist, it will create an A record with your public IP address
6. If it does exist, and the IP addresses do not match, it will update the DNS record with the new IP address
7. If the IP addresses do match, nothing will happen
8. The background service delays for the configures time interval


## Running the container

Make sure you have Docker Desktop installed.

Obtain an API token from Cloudflare with at least Zone Read and DNS Edit permissions.

![api_token_permissions](/assets/images/2023-02-20/api_token_permissions.png)

If you use Portainer as well, you know what to do.

Otherwise, create a `docker-compose.yml` file in a directory of your choosing with the following contents.


```yml
version: '3.8'

services:
  cloudflareddns:
    image: 'ghcr.io/alex-oswald/cloudflareddns:main'
    environment:
      CloudflareApi__ApiToken: api_token
      DDNS__ZoneName: example.com
      DDNS__DnsRecordName: vpn.example.com
```

Add your API token value, and proper Zone and DNS record names.

To change the update interval from the default 15 minutes, add another environment variables named `DDNS__UpdateIntervalSeconds`
with a value of your choosing.

If you've come this far, you know to run the following to see it work.

```bash
docker-compose up
```

You will start seeing logs like the following.

```
[08:29:16 INF] Starting CloudflareDDNS host
[08:29:16 INF] DDNSBackgroundService service started
[08:29:17 INF] Now listening on: http://[::]:80
[08:29:17 INF] Application started. Press Ctrl+C to shut down.
[08:29:17 INF] Hosting environment: Production
[08:29:17 INF] Content root path: /app
[08:29:18 INF] Public IP: *.*.*.*
[08:29:18 DBG] Get zones uri=https://api.cloudflare.com/client/v4/zones
[08:29:18 DBG] Get zone details uri=https://api.cloudflare.com/client/v4/zones/id
[08:29:18 INF] Zone info: id=id, name=example.com
[08:29:18 DBG] Get DNS records uri=https://api.cloudflare.com/client/v4/zones/id/dns_records
[08:29:19 DBG] Found DNS record vpn.example.com
[08:29:19 DBG] Your public IP matches the DNS record content. Nothing to update here. 👍
[08:29:19 INF] Waiting 900 seconds till next update... 😴
```

Enjoy!