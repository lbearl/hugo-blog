+++ 
date = 2024-10-14T19:48:05-05:00
title = "Reminder Lights"
description = "Reminder Lights are lights in a home automation system which aid in remembering recurring tasks"
slug = ""
authors = []
tags = ["home-automation"]
categories = []
+++

## What is it?

A reminder light is a light that isn't on all the time. When the light is on (or is a specific color), it is reminding you to do something. I'm using reminder lights in order to remember to take the trash out (every Monday night) and to give one of my dogs his medicine every morning. The concept is pretty simple: I have a multi-color light in the light socket over my kitchen sink (we rarely use that light). Every morning shortly before I get up, it turns on and turns blue in color. When I make my way downstairs and see the light on over the kitchen sink, I'm always reminded to prep my dogs medicine so that he can have it with his breakfast. Since the solution was working so well, I set up a second automation to turn on the light (green in this case) on Monday evenings to remind me to take out the trash.

## Implementation

If you already have a home automation system of some type, implementation is fairly straightforward: setup the light, create a couple of automations to turn it on (and to the correct color) at the correct times, and then implement something to turn the light off (and potentially do other things at the same time). My house leverages Apple HomeKit pretty extensively, so that is the ecosystem I've developed everything in. One of the great strengths of having everything in Homekit is that there is native support for Apple Shortcuts allowing me to do more than simply turn on/off the light (I currently have shortcuts that either send a text message or mark a task complete in the Apple Reminders app).

![Screenshot of Shortcuts implementation](/images/garbage-out.jpeg)

In the shortcut above, a search is done of all of the reminders that are in my iCloud account, where the title is "Take out trash" which have not been completed and which are due today, and if any are found they are marked complete. When I first moved into my house I setup a reminder called "Take out trash" which recurs every Monday. I have the shortcut saved on my iPhone's Home screen so after the bins are taken out I can simply press a button on my phone and the light will turn off and the task will be marked complete.

## Future Direction

In the future it would be nice to automate things even more so that even the light turning off and other actions associated with that are handled automatically. I've seen some interesting concepts online around having motion sensors in the area where trash bins are stored, however I haven't seen any options that seem reliable enough to me that are also straightforward enough to implement. I enjoy home automation, but enjoy it significantly less when things don't just work the way they should.


