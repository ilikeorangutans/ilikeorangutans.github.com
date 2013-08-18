---
layout: post
title: Writing and Testing Custom Guice Scopes
description: ""
category: null
tags: 
  - Guice
  - Java
  - Custom Scope
  - Junit
  - Unit Testing
  - Apache Sling
published: false
---

{% include JB/setup %}

I'm a big fan of [Google Guice](https://code.google.com/p/google-guice/) and use it quite a lot. Recently I spent some time writing a new custom scope for Guice. The issue to solve was that our project runs on Adobe CQ and Apache Sling. After adding the Guice injectors to the mix, it became apparent that ``@RequestScoped`` was insufficient for our needs. We needed a scope that contained only a single component, something like ``@ComponentScoped``. Long story short, we needed a custom scope and I thought that would be a great opportunity to dig really deep into Guice.

I studied the [custom component sample](https://code.google.com/p/google-guice/wiki/CustomScopes) in the Guice docs and the fundamental steps are straight forward enough, and they'll help you write your own scope. There were some pitfalls along the way that took me quite a while to figure out. The resulting implementation is not as sophisticated as I've seen in other places, however, I needed to note some specific things that I ran into.

### Create Scope Annotation

The annotation is used to sort objects into specific scopes. Implementation is trivial:

{% highlight java linenos %}
import com.google.inject.ScopeAnnotation;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@ScopeAnnotation
public @interface ComponentScoped { }
{% endhighlight %}

### Implement Custom Scope

The actual scope class represents your scope to Guice. As such, it has to keep track of all the objects that are contained in itself. Fundamentally I followed the Guice guide for custom annotations and used a ``ThreadLocal`` with a ``Map``. 

My scope performs the following operations:

* *Enter* the scope
* Offers methods to *seed* objects in the scope
* Implement ``Scope``'s ``scope()`` method to return objects from the scope
* *Exit* the scope

{% highlight java linenos %}
import com.google.inject.Key;
import com.google.inject.OutOfScopeException;
import com.google.inject.Provider;
import com.google.inject.Scope;
import java.util.HashMap;
import java.util.Map;

public class ComponentScope implements Scope {
	/**
	 * Defines the name of the scope, used to get this particular
	 * scope from the injector.
	 */
	public static final String SCOPE_NAME = "componentScope";

	/**
	 * This ThreadLocal holds a map in which we store all the 
	 * scope specific instances.
	 */
	 private final ThreadLocal<Map<Key<?>, Object>> threadLocal = 
					new ThreadLocal<Map<Key<?>, Object>>();

	public void enter() {
		threadLocal.set(new HashMap<Key<?>, Object>());
	}

	public void exit() {
		threadLocal.remove();
	}

	/**
	 * Returns a Map holding all instances contained in this
	 * scope. Also ensures that we're actually in the scope.
	 */
	private Map<Key<?>, Object> getScopeMap() {
		if (threadLocal.get() == null)
			throw new OutOfScopeException("I'm afraid, old chap, " +
				"that we're not in the appropriate scope.");

		return threadLocal.get();
	}

	/**
	 * Returns a Provider that either finds an object in the current
	 * scope or use the unscoped provider otherwise.
	 */
	@Override
	public <T> Provider<T> scope(final Key<T> key, final Provider<T> unscoped) {
		return new Provider<T>() {
			@Override
			public T get() {

				final Map<Key<?>, Object> map = getScopeMap();

				if (!map.containsKey(key)) {
					map.put(key, unscoped.get());
				}

				return (T) map.get(key);
			}
		};
	}

	/**
	 * Place new objects in the scope.
	 */
	public <T> void seed(Key<T> key, T value) {
		Map<Key<?>, Object> scopedObjects = threadLocal.get();
		checkState(!scopedObjects.containsKey(key), 
				"Key %s was already seeded in this scope.", key);

		scopedObjects.put(key, value);
	}
}	
{% endhighlight %}

There isn't anything surprising about this scope. It essentially wraps a Map and returns that in the ``scope`` method. The only thing I'm still unsure about is the ``unscoped`` provider in the ``scope()`` method. The docs are note clear on what it does or how it behaves, but I imagine it goes back to the injector and asks if there's another provider that might have this object.

### Module to Bind Your Scope

In order for Guice to know about your scope, you'll have to bind it in a module. I created a custom module for my scope:

{% highlight java linenos %}
public class ComponentModule extends AbstractModule {
	@Override
	protected void configure() {
		ComponentScope componentScope = new ComponentScope();
		bindScope(ComponentScoped.class, componentScope);
		bind(ComponentScope.class)        				
        	.annotatedWith(Names.named(ComponentScope.SCOPE_NAME))
        	.toInstance(componentScope);
	}
}
{% endhighlight %}

Nothing surprising here. We create our own instance of the ComponentScope, and use the ``bindScope()`` method to bind the annotation to the scope object. 

In the next line we bind the scope object itself in the module. That way we can obtain the scope instance we just created by querying the injector. 

### Testing the Custom Scope

Testing a custom scope is surprisingly simple, but I made some silly mistakes initially. But let's start with the testing code. A scope is just a plain java object. You enter by calling the ``enter()`` method (or whatever you called it) and you exit it by callling ``exit()``. All you need then is a injector to do the setup.

Here's the unit test I ended up with. The GuineaPig class is just a test class that I use in this unit test. 

{% highlight java linenos %}
public class ComponentScopeTest {

	Injector injector = Guice.createInjector(new ComponentModule(), new AbstractModule() {
		@Override
		protected void configure() {
			bind(GuineaPig.class).in(ComponentScoped.class);
		}
	});

	ComponentScope scope = injector.getInstance(Key.get(ComponentScope.class, Names.named(ComponentScope.SCOPE_NAME)));

	@Test
    public void testScope() throws Exception {
		scope.enter();  
		GuineaPig piggy = injector.getInstance(GuineaPig.class);
        assertThat(piggy, notNullValue());
        scope.exit();
    }
}
{% endhighlight %}



### Integrating With Servlet Filters


### Gotchas with Custom Scopes

**Thread Safety**

If you're planning to use your custom scope in any non-trivial environment, you have to ensure that it is thread safe. Make sure you don't use any instance variables except the ThreadLocal, use final and immutable fields where you can, and in general, be smart about your code. 

**Seeding**

This took me quite a while to properly figure out, longer than I'd like to admit. One of the cool things about a custom scope is that you can fill it with scope related, useful objects. And my initial, naïve thinking was that this is what the ``seed()`` method in the custom scope is for. After reading Andrew Till's post on [Guice Custom Scopes](http://andrewtill.blogspot.ca/2011/09/guice-custom-scopes.html) things started to clear up. In Andrew's words:

| A scope is a condition for reusing injected objects, instead of creating a new one each time.  

With that in mind, it suddenly makes sense what the ``unscoped`` object is, and how to get objects into