---
title: "YASC: Yet Another SSL Checker"
date: "2017-05-07"
categories: 
  - "software-development"
---

I wanted to do a little side project in less than 24 hours, which is fairly simple, but enough to really get my feet wet with ASP.NET Core MVC. I decided to build a little tool which can examine current details of an SSL certificate, and also which you can use to get proactive emails when you are 30 days out from a certificate expiring. This was also a great opportunity to play with bootstrap 4 and C# 7.

## Technologies Used

1. Bootstrap 4
2. ASP.NET Core MVC
3. Entity Framework Core
4. SendGrid + SendGrid Transactional Templates
5. Hangfire.IO

Before I start getting long-winded, if you'd like to just see the code, [head over to my github](https://github.com/lbearl/YASC).

## High Level Approach

This application was kind of fun in that the core logic which actually "does" the important part of checking the SSL certificates is only a couple-dozen lines of code. The way it works is to open up a connection to a server and then take a look at the SSL certificate that was associated with the response. There are a few "limitations" around what can be inspected currently, as anything other than a non-expired, trusted certificate will throw an exception. I figured it would be fun to play around with Hangfire and Sendgrid as well in order to do a nice background batch email process.

## Hosting On Azure

If you want to play around with it, the application is running on a free Azure App Service [here](http://yetanothersslchecker.azurewebsites.net/). Note that these free services will turn themselves off with inactivity, so almost certainly (unless this service becomes incredibly popular), the cron tasks that Hangfire is executing will not happen (as that would require the server to be running).

## Thoughts

This was a really fun 1 day project which let me play around with a few technologies I have played with in a very limited fashion. There are a number of bugs I noticed, so this could really use a bit of polish and TLC, but that probably won't happen anytime soon. Please poke around at the code, let me know if you see anything really "surprising" or anything which would be an easy improvement.

Thanks for reading!
