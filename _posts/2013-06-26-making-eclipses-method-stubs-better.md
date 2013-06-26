---
layout: post
title: "Making Eclipse's Method Stubs Better and other things"
description: ""
category: 
tags: [ Eclipse ]
---
{% include JB/setup %}

Just downloaded [Eclipse Kepler](http://www.eclipse.org/kepler/) and I'm quite happy with it. It appears fast and stable so far, but that could be just that it's a brand new install. Anyways, I re-added some of my usual code templates and while doing so, I discovered a few useful things. And because I keep doing this on every Eclipse installation, I decided to write this down here. On a related note, an Eclipse plugin to share your Eclipse templates would be pretty rad. But I digress. 


### SLF4J Logger

	${:import(org.slf4j.Logger,org.slf4j.LoggerFactory)}
	private static final Logger LOGGER = LoggerFactory.getLogger(${enclosing_type}.class);

This template will automatically create a new logger for the current type and import the necessary classes. I was unaware of the ```${:import()}``` directive until today, but it is really useful.

### Method Stubs

Per default Eclipse creates method stubs that look like this:

	@Override
	public Object myFunction() {
		// TODO Auto-generated method stub
		// return null;
	}

This is convenient, but can be cause for bugs because you just get null. More than once I've been stung by this. So I decided a better way to deal with this is to replace the template with the following:	

    throw new UnsupportedOperationException("${enclosing_method}()");

Instead of returning a default value, the method stubs will now throw an exception. Forgot to implement it? No problem, you'll find out very quickly. It's debatable whether ```UnsupportedOperationException``` is the appropriate type for this, but in my opinion it works quite well.
