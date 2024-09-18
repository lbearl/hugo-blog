---
title: "Does it even exist if it isn't in source control?"
date: "2017-04-12"
categories: 
  - "software-101"
  - "software-development"
  - "source-control"
---

I was working on a little side project at work to make my employer's data backup processes a bit more robust which involved writing a small console utility. I was asked if the code for the utility was already in source control (this code represented about 10 hours of effort on my part), and I almost replied "Does it even exist if it isn't in source control?" Instead, of course, I simply sent the link to the repo where the code lives.

This made me start thinking about what really goes into a project, small or large. How many small little side projects have I started, and abandoned within a few hours simply because whatever I was working on wasn't interesting enough to hold my interest? Would it be logical to start every side project by setting up source control?

At the end of the day I think there is a real judgement call that has to be made about what is appropriate to set up a repository for and what can just live (and likely die) on your local disks. When learning new technologies I generally work through a few different types of "Hello World" type applications that are very simple and then I'll work through a tutorial or two that is a bit more advanced. Generally the bulk of the code for those things is heavily derived from whatever resource I'm using, so there is minimal to no benefit to putting it in source control. As soon as you move beyond the tutorials: enter source control, stage left.

Modern day software professionals have no excuse not to use source control. [Github](https://www.github.com) has free public repositories (and at $7/mo, most folks in software can afford the expense), [Bitbucket](https://www.bitbucket.org) has free public and private repositories, and those are only the two options that I have used extensively. If you don't trust anyone else with your code, [standing up your own git instance](https://www.linux.com/learn/how-run-your-own-git-server) on a VPS is possible as well.

If you really don't like git for some reason (I'd highly encourage you to learn it though), there are plenty of other options: 1. Bitbucket offers mercurial hosting 2. Fossil SCM (with hosting available [here](http://chiselapp.com/)) - note I've never used Fossil, but I've heard many good things about it 3. If you want to kick it old school, there are plenty of SVN hosting options available as well

In the end, the adage "Does it even exist if it isn't in source control" rings true. There are so many different options suitable for just about any skill level and personal preference and cost bracket that there are no excuses to no simply use source control.
