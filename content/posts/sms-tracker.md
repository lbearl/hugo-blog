---
title: "SMS Tracker"
date: "2022-03-16"
categories: 
  - "software-development"
---

My wife and I have a senior dog who is starting to show his age. He has been having a lot of the issues that come with old age, and we needed a way to make sure that we were doing everything we can to keep him healthy. One of the big issues we were running into is remembering all of the stuff that happens daily: did he poop, did he get his medicine, etc.

The initial approach we took was to use a printed paper calendar and just write things down each day. It's an effective, low-tech solution, but obviously has a lot of drawbacks. We went out to Colorado to ski and had to train our pet sitter on how our calendar worked, and also had no visibility into it while we were gone.

I decided to throw together a small web application which allows us to just send an SMS message to track things. This solution is great, since all that is involved is adding phone numbers to an allow list (and anyone we need to have tracking stuff already has shared their number with us), so there is no need to worry about logins, passwords, etc.

## Anatomy of the Application

The application (creatively called SMS Tracker) is a pretty standard .Net 6 web application, using Razor pages for the UI and a couple of more traditional Asp.Net MVC controllers for processing Twilio messages and handling a few AJAX calls. The backend database is just SQLite, using EF Core as an ORM (basically the exact web application template that Microsoft ships with Visual Studio).

You can find the code on my Github [here.](https://github.com/lbearl/SmsTracker/)

### Functionality

The app uses Asp.Net Core Identity to authenticate the two users who actually have access (myself and my wife), and I've disabled the registration page to attempt to prevent others from registering. I've designed the data model in such a way that even if others did register accounts, they wouldn't be able to see anything that isn't part of their account.

One of the more interesting parts of the app is that it allows multiple "profiles", basically you can have one profile for pills, one profile for poops, etc. By using a prefix to disambiguate which profile the SMS is for, multiple things can be tracked.

There is a calendar view which can either display tracked events across all profiles that the user has access to, or it can be filtered to show only a specific type of tracked event (i.e., only check for "pill" events to see if we remembered to give him his medicine today).

### Hosting

Since this is a small app that is consumed a majority of the time from within my home, I decided to just host it on my PC. It literally is just deployed from Visual Studio and running inside of IIS. I have CloudFlare in front of everything (right now it's using a CNAME to my dynamic IP, but eventually I want to get Argo running to hopefully lock things down a bit more). My PC is on a majority of the time, so hosting it there doesn't really impact anything - the app itself is using ~64 MB RAM, while I have 128 GB in the PC.

One very interesting thing with this app is that I actually wrote ~95% of the code on Mac with an Arm chip, and with absolutely zero changes it's now running on Windows with an AMD processor.

## Reception

I built this mostly for an audience of one other (my wife), and she has been pretty enthusiastic about it, so I'd say it's been an effective application. Getting rid of the paper tracking has been nice and being able to see when things have happened (the tracked events record their received timestamp), lets us easily make sure that nothing has been forgotten.

## Conclusion

I would love to hear if anyone else decides to borrow this code, it's a pretty nice basis and hosts pretty easily. I had previously built a docker container for it but decided to just go down the path of hosting it on my PC directly in IIS for now. I am moving in a few weeks to a house and am planning on doing a bit of home automation there, so I may repurpose a Raspberry Pi as a docker host and move it docker at that point.
