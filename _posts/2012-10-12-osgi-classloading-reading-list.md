---
layout: post
title: "OSGI Classloading Reading List"
description: ""
category: 
tags: [ Java, OSGI ]
---
{% include JB/setup %}

I've been doing a lot of reading on how OSGI handles class loading. I'm starting to fully understand and leverage OSGI and I'm trying to keep my framework [Object Mapper](https://github.com/ilikeorangutans/omf) as compatible with OSGI as I can.

In OSGI classloading is very restricted, similar to what happens in enterprisy Java application servers, and completely unlike traditional Java applications where everything shares a single classloader. In OSGI each bundle has its own classloader and what is visible accross the bundle boundaries is subject to strict export rules. In order to use a class outside of a bundle you'll have to explicitly declare its package as exported. There are many benefits to this strict architecture, clean architectures, reusable components and the ability to update bundles at runtime are probably the most important ones.

The downside of this strict model is that frameworks that rely on byte code generation will have a hard time making classes available. There are many techniques I was completely unaware of, and here are some excellent references I am reading:

* [OSGI Extender Pattern](http://rinswind.blogspot.ca/2009/07/osgi-go-forth-and-extend.html)
* [Code Generation with OSGI](http://www.infoq.com/articles/code-generation-with-osgi)
* [Presentation on OSGI WeavingHook](http://www.slideshare.net/mfrancis/bytecode-weaving) ([WeavingHook](http://www.osgi.org/javadoc/r4v43/core/org/osgi/framework/hooks/weaving/WeavingHook.html))
* [Generating Bytecode Proxies with ASM](https://github.com/GregBowyer/frames/commit/4cfa207a9c16f4e6612dddd5f31fcb8850b3dd8a)
