---
layout: post
title: "OSGI Bootstrapping"
description: "How to start an OSGI application"
category: 
tags: [ OSGI, Java]
---
{% include JB/setup %}

I've been working with OSGI based technology for a while now and it's a great piece of technology. However, so far I've only worked with OSGI technology that runs in existing instances of the framework. A running framework is easy to deal with, but how do you get to this point? I spent some time today playing with that, and it took me a while to put it all together. So naturally, I'll have to record my findings here.

Section 4.2.1 of the [OSGI specification (version 4.3.0)][osgi-spec] explains the process in minute detail. Here's a short rundown of how it works:

* Every framework implementation must provide a ``org.osgi.framework.launch.FrameworkFactory`` that produces instances of the framework. There's several different ways to retrieve the actual class to be used. It could be hardcoded, retrieved via command line parameter or via Java service loader
* Once you have your FrameworkFactory instance, it's as easy as calling ``frameworkFactory.newFramework(Map<String, String> configuration)``. The ``configuration`` Map is a bunch of key-value pairs with configuration parameters for the framework (and the implementation). From what I gather, all the OSGI specific keys are defined in ``org.osgi.framework.Constants``. One example is the ``Constants.FRAMEWORK_STORAGE`` key, that defines where the framework is supposed to store its bundles.
* The ``newFramework()`` method will create a instance of ``org.osgi.framework.launch.Framework``, representing the entire OSGI runtime. You'll have to initialize it by calling ``init()`` and then start it with ``start()``. Once ``start()`` returns, your framework should be up and running. 
* In order to avoid the framework being shut down right away, it makes sense to call ``waitForStop(int timeout)``, a synchronuous method that will block until the framework is shut down. 


That should be enough to get your framework started. 


Other useful links:

* [Apache Felix Embedding and Launching][felix-embedding]: how to embed and launch Apache Felix
* [Apache Felix Framework properties][felix-properties]: nice summary of OSGI framework and Felix properties
* [Apache Sling Launchpad][sling-launchpad]: builds bundled Java apps including Jackrabbit, Sling and Felix complete with a launcher.

[osgi-spec]: http://www.osgi.org/Release4/Download
[felix-embedding]: http://felix.apache.org/site/apache-felix-framework-launching-and-embedding.html
[felix-properties]: http://felix.apache.org/site/apache-felix-framework-configuration-properties.html
[sling-launchpad]: http://sling.apache.org/site/maven-launchpad-plugin.html
