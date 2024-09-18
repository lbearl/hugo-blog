---
title: "VPS Migration to GCP"
date: "2017-04-04"
categories: 
  - "dev-ops"
  - "public-cloud"
  - "system-administration"
---

I've had all of the blogs that I manage on several VPSs hosted through [RamNode](https://clientarea.ramnode.com/aff.php?aff=2989) for the past couple of years. While there is nothing wrong with RamNode, I figured it was about time to try out something new. I originally was going to move to AWS, since their MySQL RDS offering looked pretty compelling, but then I discovered Google pretty much had feature parity with the parts of AWS I was interested in (plus they have a much nicer value proposition than AWS does for hosting costs).

## Out With the Old

In order to keep things reasonably fast, I ran 4 separate VMs: web1, web2, db1, and cache1. The initial idea was that web1 and web2 would be replicas of each other and cache1 (which ran Varnish) would be responsible for balancing load. I never got around to any of that, instead I have web1 (which had about 90% of the traffic) and web2 as completely independent machines. Both talked to cache1, which was running varnish and acted as the public entry point to the blogs. Db1 was just a database server, running some recentish version of MariaDB. Also, in order to help secure the database server, all of the VMs ran OpenVPN so that all internal communication was happening in a private network.

Unfortunately, this was probably a bit over-engineered for the amount of traffic that the blogs actually get in aggregate (under 200k uniques/month), plus it was a maintenance nightmare since I needed to watch over 4 separate servers (but it was a fun learning experience).

## In With the New

Now that I have moved to Google Compute Engine, I just have 1 moderately beefy VM running which hosts all of the blogs. Instead of running a dedicated VM (which I would have to administrate), I'm using Google's Cloud SQL MySQL offering (a dbn1-standard-1 instance). Eventually all of the images will be moved to Google Cloud Storage (this is one area where AWS is light years ahead of Google), in order for that work I need the Google sponsored wordpress plugin to [actually work properly](https://github.com/GoogleCloudPlatform/wordpress-plugins/issues/42).

## Future Plans

The reason that I wanted to migrate to either AWS or Google is in order to support future growth. While volume is moderately low right now, it is to be hoped that eventually one or more of the blogs being hosted will have considerable amounts of traffic. If that happens being able to reconfigure a few things in order to support GCP's load balancing will be critical. The only thing preventing me from setting that up right now is that I will need an SSL certificate which can terminate all of the blogs I manage, which would be a bit expensive (especially since I currently just use LetsEncrypt for all my certificates).

Overall we'll have to see if this ends up being more reliable and at least as fast as what I previously had configured. I have enough CPU and memory budget where I can probably implement a caching strategy again if the performance isn't quite where it needs to be (or I'll just end up setting up a second VM in order to handle all of the caching duties). I'm still running everything through CloudFlare, so that makes sizing everything much more forgiving.
