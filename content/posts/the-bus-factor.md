---
title: "The Bus Factor"
date: "2017-05-07"
categories: 
  - "personal-development"
  - "software-development"
---

I've been meaning to write this for quite a while, as the bus factor is something I've (literally) run into in my career. For those of you not familiar with it, the "Bus Factor" is basically an informal measure of resiliency of a project to the loss of one or more key members. It's basically the programming version of the old adage "Don't put all your eggs in one basket".

## Story Time

Some years ago I was a software development intern at a large company in Milwaukee, Wisconsin. The team I was on was broken into a U.S. development team, an offshore dev team in India and an offshore QA team in China. We had daily scrum meetings at 8 AM every morning so that the US and Indian teams could participate all in one. One day we got word that one of the senior-most developers had literally been hit by a bus while crossing the street (thankfully he made a full recovery, but it certainly slowed down that part of the team as he was out for 8 weeks or so).

## How to Reduce the Bus Factor

I'm sure people can (and probably have) written entire books on the subject of reducing the bus factor, and spreading knowledge around through the entire team. Spreading knowledge is really the key element in bus factor reduction.

How many people currently work on a team where one or two people are basically the wizards who secret spells make critical things happen (like deployments or provisioning infrastructure assets, or SSL certificates, or any of the other million things that need to be done in order to make software work)? I know I've worked on several teams where that happened. I've also worked with people who wanted to increase the bus factor as they thought it gave them better job security (a notion I strongly disagree with).

In my experience one of the best ways to reduce the bus factor is to maintain an internal wiki where developers and administrators can document processes for anything which they are going to do more than once (and sometimes it's good to document things that are being done once as well). Another great idea is to regularly schedule cross training (n+1 isn't only a good idea for infrastructure, developers and admins should have a bit of redundancy as well).

## Ethics

I personally feel that there is an ethical responsibility for all engineers to be transparent in what they do. I never want to be the only person capable of doing something, instead I do my best to make sure that anything I do, which may ever need to be done again, is documented at least well enough that someone can probably piece it together. Doing this ensures that if I am ever hit by a bus the rest of my team won't have to try to figure out the magical incantations I have developed in order to do a number of things.

## Conclusion

At the end of the day reducing the bus factor is good for your team. You never know when you or one of your colleagues are going to end up no longer being available to work (they might be hit by a bus, or it might be something more mundane like taking a new job, or leaving for a few months for a sabbatical or maternity/paternity leave). As an engineer and a member of a team you have an ethical obligation to ensure that you are both sharing processes and techniques you've developed with your colleagues, and also trying to learn those processes and techniques from your colleagues.
