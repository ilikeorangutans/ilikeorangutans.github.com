---
layout: post
title: "OSGI, Guice, and Peaberry: first steps"
description: ""
category: 
tags: [OSGI, Guice, Peaberry]
---
{% include JB/setup %}

## Introduction

I've been trying to get [Google Guice](https://code.google.com/p/google-guice/) and [Peaberry](https://code.google.com/p/peaberry/) to work in my OSGI projects for a while. Google Guice is a great dependency injection framework, and Peaberry promises to bridge the gap between OSGI services and dependency injection. 

However, getting Peaberry to work was not trivial, mostly because there aren't many docs and the quality of the docs is somewhat lacking. The best piece is still this pdf [Peaberry - blending services and extensions](https://code.google.com/p/peaberry/downloads/detail?name=peaberry%20-%20blending%20services%20and%20extensions.pdf), but it's a lot of information in very little space. 

These are my development notes while trying to get Peaberry to work, they are incomplete and might even be incorrect. 

## Setup

* Deploy Guice jar file (it's a bundle), Peaberry, and javax.inject
* Peaberry shows up as a separate module, exporting a caching registry service
* In your activator, create a Injector with your own module and ``Peaberry.osgiModule(bundleContext)``.

{% highlight java %}

public class Activator implements BundleActivator {

	public void start(BundleContext context) throws Exception {
		Injector injector = Guice.createInjector(Peaberry.osgiModule(context), new MyModule());
	}

}
{% endhighlight %}

* once you have your injector, you can inject into the activator:

{% highlight java %}
		injector.injectMembers(this);
{% endhighlight %}

* Bundle local objects can be created using regular modules:

{% highlight java %}
public class MyModule extends AbstractModule {
	protected void configure() {
		bind(DemoService.class).to(DemoServiceImpl.class);
	}
}
{% endhighlight %}

## Services

So far we've only used plain Guice, but Peaberry's real strength is that it allows you to bind to OSGI services in your modules. You can: 

### Consume existing OSGI services

Bind them to a special proxy using regular guice syntax:

{% highlight java %}
import static org.ops4j.peaberry.Peaberry.service;
import org.osgi.service.cm.ConfigurationAdmin;
import com.google.inject.AbstractModule;

public class DemoModule extends AbstractModule {
	protected void configure() {
		bind(ConfigurationAdmin.class).toProvider(service(ConfigurationAdmin.class).single());
	}
}
{% endhighlight %}

This example binds the OSGI ConfigurationAdmin service. Notice how we're binding ``ConfigurationAdmin`` to a provider, using the ``Peaberry.service()`` method. Because OSGI services are highly dynamic and Guice injectors are immutable, the creators of Peaberry did something smart and created providers that are proxies for the services. And depending if you want only a single instance of the service, or all the registered services for a type, you can use either ``Peaberry.service().single()`` or ``Peaberry.service().multiple()``:

{% highlight java %}
public class DemoModule extends AbstractModule {
	protected void configure() {
		bind(ConfigurationAdmin.class).toProvider(service(ConfigurationAdmin.class).single());
		bind(SomeService.class).toProvider(service(SomeService.class).multiple());
	}
}

public class Consumer {
	
	/**
	 * Single service
	 */
	@Inject
	private ConfigurationAdmin configurationAdmin; 

	/**
	 * Multiple services are injected as an Iterable.
	 */
	@Inject
	private Iterable<SomeService> services;
}

{% endhighlight %}

You can reference bound OSGI services in your bundle using ``@Inject``, like any other field:

{% highlight java %}
	@Inject
	private ConfigurationAdmin configurationAdmin;
{% endhighlight %}

### Publish objects from Guice as OSGI services
This step I found somewhat counterintuitive. In order to export a Guice managed object as a OSGI service, you have to bind it using ``Peaberry.service().export`` and then inject a member of type ``Export<YourType>`` into some other class. As soon as that field gets injected, the service will be exported. 

You can use peaberry to export a service to OSGI by creating the following binding:

{% highlight java %}
import static org.ops4j.peaberry.Peaberry.service;
import static org.ops4j.peaberry.util.TypeLiterals.export;

public class DemoModule extends AbstractModule {
	protected void configure() {
		bind(export(DemoService.class)).toProvider(service(DemoServiceImpl.class).export());
	}
}
{% endhighlight %}

And injecting a member of type ``Export<YourServiceType>`` for example on your activator:

{% highlight java %}

public class Activator implements BundleActivator {

	/**
	 * As soon as this member gets injected, the service will be exported
	 */
	@Inject
	Export<DemoService> exportedDemoService;

	public void start(BundleContext context) throws Exception {
		Injector injector = Guice.createInjector(Peaberry.osgiModule(context), new DemoModule());
		injector.injectMembers(this);
	}
}

{% endhighlight %}


## Outstanding Questions

* Exposing an object as an OSGI service won't expose the service to your Guice clients. You'll have to add a regular binding for it. Not sure if the regular bound service is the same instance as the OSGI service. But they appear ot be separate instances. 
* Servlets? I've experimented with guice-servlet, but in Apache Sling it seems to not work very well. 

