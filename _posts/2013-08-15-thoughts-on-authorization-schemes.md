---
layout: post
title: "Thoughts on Authorization Schemes"
description: ""
category: 
tags: [ Security, Authorization, OAuth ]
---
{% include JB/setup %}

Today I was experimenting with some new tools ([Prose](http://prose.io/)) and was confronted with a screen that probably everybody that owns a smartphone or uses any kind of connected online service has seen before: an authorization screen where the user is expected to either approve or deny an application based on list of permissions. Here's how Github's screen looks like:

![Github Authorization Screen](/assets/images/github-authorization.png)

Android has a similar screen when you install an app, IOS probably has one as well. Pretty much every connected system, be it Facebook, Github, Stackoverflow, pretty much everything that uses OAuth or similar technologies will have a screen like this. From a technology and development point of view it makes a lot of sense; at least initially. You break down privileges in a system into a few neat categories and if an application wants to use this service, it has to declare what it uses. Ideally the system would enforce that the application can only use the declared permissions and in theory that would make everything safer…

Reality, unfortunately, is different. Most people don't even read the permissions that the apps they're installing on their phones are requesting. It's just another annoying click. Even if people read the list of permissions that apps wanted, what could they do? They can only approve or reject the application. And often enough, the concerns as "What does the Facebook app do with my location and call data?" are dismissed in favour of whatever pleasantness the app offers. This opens an interesting question: what good are permissions if people will give out whatever permissions an application asks for? 

Every system that has this concept of privileges or permissions will warn its users that they are about to let an application read all their text messages or send location data over the wireless data connection. And just to make sure that your system is secure, it will warn you to only install applications that you *trust*. But how could you trust an application without installing it? We've seen malware being distributed by app stores of respectable and *trusted* companies. So what good is trust if it's given out blindly?

So instead of giving an app blank access to everything it asks for, why not let the user decide how much of the permissions he wants to grant? Lets say you are about to install a social media network's app on your phone. You don't know the app, you don't trust it, and you certainly don't want it to read all your text messages, your location data, your emails, and dig around your photos. But using the data connection is ok, after all, you want to see what your friends are up to. Suddenly the permissions screen makes a lot more sense. It actually benefits the user to take time and read it, and give or reject permissions. 

I really hope that user controller permissions will become reality soon. It bugs me that with every app, I have to give away control over my data where it seems that most of that data is not required for running an app. 

And now, I'll install Prose… 