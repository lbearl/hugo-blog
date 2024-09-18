---
title: "A Gentle Introduction to Onion Architecture in ASP.NET MVC – Part 1"
date: "2016-09-19"
categories: 
  - "net"
  - "architecture"
  - "software-development"
---

Welcome to part one of a multi part series on Enterprise Application in .Net Core! In this series we'll go over everything that is necessary to build a best in breed enterprise application with the [Onion Architecture](http://jeffreypalermo.com/blog/the-onion-architecture-part-1/) at its core. The word enterprise is integral in describing the software being described as it is going to be software which is capable of being extended year after year in a clean fashion while allowing things to stay [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) and testable. We'll cover the following:

1. System to Design
2. Initial Architecture and Architectural Considerations
3. Proper Abstractions Around Each Layer of the "Onion"
4. Unit Testing

This series of posts isn't going to exhaustively walk you through every decision that may need to be considered (i.e. we'll only very briefly discuss localization). My intention with these posts is to put forth my ideas of what a solid architecture looks like. With that being said, the application we are going to design is called "Block List Checker" which is an application which will allow one or more Email Service Providers (ESPs) such as [Mailgun](https://www.mailgun.com), [SendGrid](https://www.sendgrid.com), etc. to be queried simultaneously to return a list of all email addresses which are on a suppression list. This is a very simple application which is intentionally somewhat over-engineered for the purposes of illustrating the various components of the system.

### Inversion of Control and Dependency Injection

In order to design this software appropriately there are several tools that will need to be incorporated into the solution. The first (and arguably the most important) of these is [StructureMap](http://structuremap.github.io/) a .Net-friendly IoC/DI Container. For those not familiar: IoC (or Inversion of Control) and DI (or Dependency Injection) are design patterns which help make code much more testable. They do this by "inverting" the dependency chain. The way this plays out in MVC applications is that controllers will be injected with all of the dependencies that all of the services which are used by those controllers require. The IoC container will basically maintain a mapping of interfaces to services and, at runtime, will provide the controller with an instance of the service requested. Due to the fact that this effectively requires loose-coupling between components, writing unit tests becomes much easier (note that the description of IoC here is just barely skimming the surface, entire books have been written on this subject).

### Unit Testing

Because this application is so simple, I have opted not to write comprehensive tests around it, however I have written a few tests just to demonstrate using [xunit](https://xunit.github.io/) with this application and how the loose coupling allows the testing to be achieved.

### Known Issues

This application is very basic and the UI could use a bit of love. It currently targets Asp.Net MVC5 instead of .Net Core MVC6 due to the fact that I prefer writing API access code using RestSharp, and that currently doesn't seem compatible with .Net Core. I may release another sample project in the not distant future which targets .Net Core. It also currently doesn't make use of either a proper front end framework, or of most of the functionality that MVC5 exposes on the front end. I haven't written this application to be overly robust, and due to the nature of how it consumes API keys without any authentication I wouldn't expose it on a publicly facing web server.

### Get the code

With all that being said, the code is available here: [https://github.com/lbearl/BlockListChecker](https://github.com/lbearl/BlockListChecker).

Part Two of the series is available [here](/2016/10/gentle-introduction-2)
