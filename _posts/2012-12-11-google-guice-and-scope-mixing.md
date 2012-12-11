---
layout: post
title: "Google Guice and Scope Mixing"
description: ""
category: 
tags: [ Guice, Dependency Injection, Java]
---
{% include JB/setup %}

I've been working on a small Java application I wrote a few years ago for some bug fixes and in the process of making it better, I introduced [Google Guice](https://code.google.com/p/google-guice/), my favourite dependency injection framework. On of the great features of Guice is that it supports different [scopes for injection](https://code.google.com/p/google-guice/wiki/Scopes). Per default, Guice will return a new object for every request. But sometimes you want to objects to be created a bit less liberally, for example, you want a certain object to be created only once. Guice has a ``@Singleton`` scope for that. Want an object to be created once for a request? Guice and [guice-servlet](https://code.google.com/p/google-guice/wiki/Servlets) offers ``@RequestScoped`` and ``SessionScoped``. But there's more, need JUnit per test scope? [Guiceberry](https://code.google.com/p/guiceberry/) has exactly that: ``@TestScoped`` that will make sure every test gets exactly one object. 

However, with scopes comes the challenge of nesting them. Not every object created in one scope may freely be used in another scope. For example, a request scoped object can not be injected into a singleton scoped object; after all, what should Guice inject? At the time the singleton is created, there are probably no requests. And what if there are multiple concurrent requests? Obviously, the singleton and the request scope are incompatible. In Guice parlance, the request scope is *narrower* than the singleton scope. Any attempt to inject a dependency from a narrower scope into a broader scope will cause Guice to throw an exception.

This opens a sleigh of different problems. For example, Servlets are kind of singletons, yet they need access to request scoped objects. And a session scoped object might need access to an a business service in the singleton scope. Luckily, Guice has an solution for that. Instead of injecting the dependency directly, Guice allows you to inject a [Provider](http://google-guice.googlecode.com/git/javadoc/com/google/inject/Provider.html), that is an object that *provides* other objects. Now, the provider is smart enough to decide when to create a new object or reuse the same one, effectively decoupling the injection target and source and their scopes. Creating a provider is straightforward. For example, to provide instances of ``MyObject`` in request scope, you'd create the following class:

{% highlight java %}
import com.google.inject.Provider;
import com.google.inject.servlet.RequestScoped;

@RequestScoped
public class MyProvider implements Provider<MyObject> {
	public MyObject get() {
		// Create an instance of MyObject...
		return myObject;
	}
}
{% endhighlight %}

To use request scoped instances of ``MyObject`` in a singleton scoped service, you'd implement something like this class:

{% highlight java %}
import com.google.inject.Provider;
import javax.inject.Inject;
import javax.inject.Singleton;

@Singleton
public class MyService {

	@Inject
	private Provider<MyObject> myObjectProvider;

	public void doServiceThings() {
		MyObject myObject = myObjectProvider.get();
		// do things...
	}

}
{% endhighlight %}

Notice how we inject ``Provider<MyObject>`` instead of ``MyProvider`` (this is a mistake I made initially). And in order to get an instance of ``MyObject``, we call ``provider.get()``. The service will always get the appropriate object for the current request (or whatever scope in question). A consequence of this is that the service has to be threadsafe in order to deal with concurrent requests. This applies for all objects that have to deal with objects from a narrower scope.

Once understood, Guice's scopes, and I'm sure other dependency injection frameworks have similar concepts, are a powerful tool that greatly simplify development. Hopefully I was able to show some of their advantages and their use. 

