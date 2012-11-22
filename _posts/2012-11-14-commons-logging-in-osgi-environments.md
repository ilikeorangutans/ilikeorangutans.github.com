---
layout: post
title: "commons-logging in OSGI Environments"
description: ""
category: 
tags: [ OSGI, Java, commons-logging, commons-httpclient, Pax Logging ]
---
{% include JB/setup %}

While working on a small toy project using Apache Felix and commons-httpclient, I kept running into the issue that there is no official OSGI bundle for Apache commons-logging out there. While most Apache commons projects either provide simple bundles or full blown OSGI implementations with Activators and Services, commons-logging is an interesting exception. If you scan the [Commons OSGI status page][apache-commons-osgi], you will notice that there is no OSGI version for commons-logging available, and a separate section to explain why. I haven't dug into why and how it conflicts with the OSGI classloading scheme, but I can see that being a problem. Be it as it may, there is no official OSGI bundle for commons logging. 

The problem was that I needed commons-httpclient which requires commons-logging. The solution was rather simple, but took me way too long to figure out. As the Commons OSGI status page indicates, you should go ahead and use the [Pax Logging][pax-logging] bundle. It contains several logging APIs, including log4j, commons-logging, JDK logging, SLF4J, and a few others. 


Once you drop the pax logging bundle in your application, you'll notice that it exports a boatload of logging APIs, which will make using libraries like commons-httpclient easy:


	Export-Package = org.apache.avalon.framework.logger;provider=paxlogging;uses:="org.apache.log";version="4.3",org.apache.commons.logging;provider=paxlogging;uses:="org.ops4j.pax.logging,org.osgi.framework";version="1.1.1", ...

Having the actual APIs exposed in the OSGI classspace is already a huge step forward. However, once you start using the httpclient, it will flood your console with log messages. That might be OK in a GUI app (``> /dev/null``). I'll examine logging configuration in my next post. 


[apache-commons-osgi]: http://wiki.apache.org/commons/CommonsOsgi
[pax-logging]: http://team.ops4j.org/wiki/display/paxlogging/Pax+Logging 

