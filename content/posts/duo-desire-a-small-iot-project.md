---
title: "Duo Desire (a small IoT project)"
date: "2020-02-14"
categories: 
  - "iot"
  - "public-cloud"
  - "software-development"
tags: 
  - "azure"
  - "iot"
  - "rebutton"
---

My wife was watching Shark Tank a few weeks ago and we were both chatting about a thing we saw ([which wasn't funded](https://www.businessinsider.com/lovesync-shark-tank-sex-button-kickstarter-mark-cuban-barabara-corcoran-2020-1)). It was kind of a clever idea, and I'll sidestep the entire discussion of whether or not it is a "good" product or not, I just wanted to see what it would take to build something kind of like it (this is loosely inspired by LoveSync, but the LoveSync does have some interesting features around time and also their hardware is _much_ prettier than what I'm using).

<figure>

![](/images/image-4.png)

<figcaption>

The Rebutton (ignore the dirty valet box)

</figcaption>

</figure>

As a quick aside: this method of asynchronous communication in which parties are only notified after reaching consensus seems like an interesting UX pattern. I was very interested to see almost the same pattern pop up (in almost the exact opposite context of LoveSync) in the form of the [Bail App](https://bail-app.netlify.com), which I believe is still in development ([if it's even feasible](https://news.ycombinator.com/item?id=22250962)).

## Quick Overview

So, to the meat of the project: this project is 2 IoT buttons hooked up to an Azure IoT Hub which trigger Azure functions and make use of Azure Table Storage for data persistence. When consensus is reached, a notification is sent via Twilio to all interested parties.

Why Azure you ask? Easy: the buttons that were in stock only had the Azure model available, so that made the decision pretty easy. Amazon's infamous IoT button is apparently no longer for sale to individuals (or if it is it is so well hidden I gave up on looking). Developing this on Azure worked pretty well, although there were some weird issues when I attempted to interact with the IoT Hub service using C#, so I ended up doing everything in Javascript (I know, IoT in JS, I never thought I'd see the day).

## Bill of Materials

- 2 [Rebuttons by SeeedStudio](https://www.seeedstudio.com/ReButton-p-2930.html)
- 1 [Azure](https://azure.microsoft.com/en-us/) subscription
- 1 [Twilio](http://www.twilio.com/referral/L7yzRF) subscription
- WiFi access (obviously)

## Building It

To start, configure the Rebutton to communicate with your IoT Hub. [The docs](https://seeedjp.github.io/ReButton/) do a pretty good job of describing this.

Ensure that the buttons are communicating with the hub. This command will let you see events as they come from the event hub: `az iot hub monitor-events -n <yourapp> --properties anno sys --timeout 0`

In the Azure Function Apps blade, you'll need to create the following functions

### Event Hub Function

The first of the three functions needed is the function which will process event hub messages. At a high level all this is doing is being invoked any time an event occurs and then just writing the event data to the Azure Table Storage table defined in the configuration. You'll need to configure it something like:

<figure>

![](/images/image-1.png)

<figcaption>

The Azure IoT Event Hub function

</figcaption>

</figure>

The code for the function is [here.](https://gist.github.com/lbearl/34dee5c7d971d44c8591cc8c43b73cb9#file-duodesire_iothub-js) As I alluded to above, I was having some weird issues with the C# bindings in Visual Studio, so I ended up just writing everything in Javascript directly in browser. Not the greatest development environment...

### Timer Function

The timer is the next function. It runs on whatever cron schedule is defined (I currently have it running every 10 minutes), and it just looks for consensus in the table for both configured devices. Consensus in this case is defined as one or more single-click events from each device. If any non-single-click events are discovered, then it is assumed that consensus is impossible, and the messages will not go out. It would be an interesting exercise to extend the current logic and use the other button events (double-click, long-press, super-long-press) for other functions.

Upon discovering consensus, the timer function writes to an Azure Queue and writes a "FINISH" record to the table storage to prevent the next run of the function from also triggering a message.

<figure>

![](/images/image-2.png)

<figcaption>

The timer configuration

</figcaption>

</figure>

The code for it is [here](https://gist.github.com/lbearl/34dee5c7d971d44c8591cc8c43b73cb9#file-timer-js).

### Queue Trigger Function

The final function needed is the queue trigger which is responsible for actually sending the messages to Twilio. [The code](https://gist.github.com/lbearl/34dee5c7d971d44c8591cc8c43b73cb9#file-queuetrigger-js) is incredibly simple as Azure has a pre-built integration with Twilio which abstracts away the vast majority of the code that would otherwise need to be written.

<figure>

![](/images/image-3.png)

<figcaption>

The Queue Trigger configuration

</figcaption>

</figure>

At this point, everything should be wired up, so pressing both buttons should trigger the Twilio message.

## Worth It?

Assuming usage remains fairly constant, Azure forecasts that I'll spend ~$0.30 per month, plus ~$50 for the buttons. I think it's worth it for the fun I had building this, plus if the current application of the buttons gets stale, I'm sure I can repurpose them for something else. Interestingly, these buttons are built on the Arduino platform, so conceivably I could do a whole lot of customization by changing the [firmware](https://github.com/SeeedJP/ReButtonApp) around (which might happen eventually).

![](/images/pricing-azure-summary.png.webp)

## In Closing

This was a fun project which didn't take very long to put together and entertained me. It was a timely project for Valentine's Day. If you end up building something like it or have any feedback on this article, please [reach out](https://lukebearl.com/contact-me/), I'd love to hear!
