---
title: "Reducing Bandwidth Costs by Moving S3 Video to BackBlaze B2 with CloudFlare"
date: "2018-11-02"
categories: 
  - "dev-ops"
  - "public-cloud"
  - "system-administration"
---

I tweeted about the cost savings that I was able to achieve by moving some videos to S3. Unfortunately in my tweet I was off on the number's a bit. Basically by switching costs went from $163.45 in September to $2.91 on Backblaze B2 in October (not quite as nice as I said in my tweet, only a 56x reduction in cost instead of a 100x reduction #oops).

Regardless of my inability to recall the amounts of bills, I figured it would be good to quickly document how I have everything setup. The setup instructions I followed are basically the same as what's documented by Backblaze [here](https://help.backblaze.com/hc/en-us/articles/217666928-Using-Backblaze-B2-with-the-Cloudflare-CDN). For the purposes of this guide, I'll assume you already have a CloudFlare account (they have free plans available if you don't).

1. Create the bucket in B2 ensuring that the bucket is marked as "Public"
2. Upload the video to the B2 bucket
3. Take note of the URL where the video (i.e. f002.backblazeb2.com)
4. In CloudFlare create a CNAME (with the "Orange Cloud") which points to the host name found in the URL of your uploaded video
5. Access the video using your-new-cname.yourdomain.extension/file//

As far as I can tell all files in your bucket will always resolve to the same hostname (the fxxx.backblazeb2.com identifier), so this is only necessary to configure once. Going forward just copy the public link from your Backblaze bucket and replace the backblazeb2.com part with your CNAME, and everything should be happy.

Quick disclaimer: I wasn't compensated by CloudFlare or Backblaze for this, I'm just sharing my experience. I make no guarantees about any results you'll see.
