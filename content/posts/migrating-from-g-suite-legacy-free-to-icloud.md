---
title: "Migrating from G Suite Legacy Free to iCloud"
date: "2022-01-20"
categories: 
  - "software-development"
tags: 
  - "email"
---

For those of us who are on Google's Legacy Free G Suite plan, there was an announcement the other day that all good things must come to an end. Sometime in the July timeframe everyone will need to either start paying or stop using the service. Fortuitously, Apple recently (in October) announced custom domain support for iCloud. For people who were using G Suite only as an email service (i.e. no Google Docs, or other Google Apps), and are only using it for personal accounts, iCloud can serve as a perfectly acceptable email host.

The migration path is pretty straight forward:

1. Go to [iCloud Settings](https://www.icloud.com/settings/customdomain) and click on the Manage button under Custom Email Domain
    1. Click Add a domain you own
    2. Select if this is for a single person or for a family account (I believe this controls if the domain is made eligible for family sharing)
    3. Enter the domain
    4. When entering existing emails, only enter the ones that already exist
    5. Enter the DNS records (MX, SPF, DKIM, TXT, etc) that Apple provides into your DNS configuration
    6. Test sending an email
    7. If you have the legacy Gmail account and the new/updated iCloud account both in the Apple Mail app on a Mac, you can just copy and paste to migrate all old emails (I moved thousands of messages going all the way back to October 2010 in about 5 minutes with this process)

At this point my plan is to just let my old Google account exist, but I expect Google will make it inoperable on their announced timeline.

## Update March 8, 2022

It looks like one thing that doesn't work all that well is sharing the account to children accounts. This reddit thread goes into detail: https://www.reddit.com/r/iCloud/comments/pttz4q/custom\_domain\_not\_working\_for\_child\_account/. Not a big deal for me right now, but hopefully something that will be addressed in the next couple of years.
