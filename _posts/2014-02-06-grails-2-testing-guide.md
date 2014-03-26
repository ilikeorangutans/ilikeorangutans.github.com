---
layout: post
title: "Grails 2 Testing Guide"
description: ""
category: 
tags: [ Grails, Grails-2.3.x, Unit Testing, Spock, Groovy, Test Driven Development, Integration Testing ]
---
{% include JB/setup %}

**Note: I'm still working on this post, but I already use it as a reference so there'll be more content over time.**

I've been quite busy at work with updating a Grails 1.3 application to 2.3.4. While writing a test harness it became apparent that lots of things have changed since I've last worked with Grails. Many changes are for the better, especially the integration of Spock framework. However, there were some issues that took me a while to figure out. The Grails docs on [testing](http://grails.org/doc/latest/guide/testing.html) are comprehensive, but long. Here's my cheat sheet.

## General

### Tests & Assertions

Tests without assertions are mostly pointless. Learn to write effective assertions. [Hamcrest matchers](https://code.google.com/p/hamcrest/wiki/Tutorial) are there to help you. If you're writing [Spock](http://spockframework.org) tests, you don't even need to write any explicit asserts. 

### Use Appropriate Test Type

_Unit tests_ are small, self contained, and quick. Whenever you want to test a single method or class, a unit test is usually the way to go. If you find yourself writing a unit test that relies on something else, you'll either want to make it an integration test or mock the collaborators. Using mocks you can precisely control what conditions your code runs under. You can control return values, exceptions, number of interactions, etc.
In Grails, unit tests go under `test/unit`.

_Integration tests_ are more sophisticated. They actually spin up the runtime environment with all its services, dependency injection, etc to provide a full system. Therefore they are slower than unit tests, but give you a better understanding how your components interact. I like to think of integration tests as tests that take an entire slice of the system and test the interactions between the involved classes. Integration tests go under `test/integration`.

### Disabling Tests

Whenever you need to disable a test, use the approriate annotation. Commenting out a test is not an acceptable solution. In GroovyUnit or regular JUnit tests, use ``@org.junit.Ignore`` and in Spock tests ``@spock.lang.Ignore``. Notice that both annotations take a String parameter that communicates why this test was disabled. Use it to make it clear why a given test was disabled. 

{% highlight groovy %}
	@Ignore("do not need to test this right now")
	void "test for a method"() {
		// ...
	}
{% endhighlight %}

### Never Mix Test Constructs

Between unit and integration tests there are huge differences. __Never mix constructs meant for unit tests with integration tests__. 

### GroovyUnit or Spock

Personally I find Spock tests much more expressive than GroovyUnit tests. Quick sample:

{% highlight groovy %}
class FooSpecification extends Specification {
	void "test for a method"() {
		setup: // implicit, no need to declare this:
			
		// Everything after this statement will be treated as an assertion
		expect: 
			
		add(2 + 2) == 4
	}
}
{% endhighlight %}

Check out the [Spock docs](http://docs.spockframework.org/en/latest/), [samples](https://code.google.com/p/spock/wiki/Examples)

## Unit Tests

Unit tests are tests where you want to test only a very specific piece of code, a single unit. That means that all dependencies of that code will have to be mocked or stubbed. The test setup is the same for all unit tests and starts by annotating your unit test with the `@TestFor` annotation. This tells Grails what class you want to test and what variables to create. 

{% highlight groovy %}
@TestFor(MyController)
class MyControllerSpec extends Specification {

	void "test something"() {
		// Grails automatically created a controller field for us:
		controller.doSomething()
	}

}
{% endhighlight %}

### Configuration

Grails' config is usually accessed via an injected instance of ``GrailsApplication`` using the ``config`` property. Grails injects ``GrailsApplication`` into unit tests, so you can access it directly:

{% highlight groovy %}
@TestMixin(GrailsUnitTestMixin)
class MySpec extends Specification {

	void "test something with config"() {
		// grailsApplication gets automatically injected 
		grailsApplication.config.myConfigValue
	}

}
{% endhighlight %}


### Mocks

During a unit test Grails will not create and inject any dependencies for you so it's up to you to do so. There's plenty of things your code interacts with and this section breaks down how to mock these things.

#### Mocking Logging

Almost all loggers in Grails artifacts are injected automatically at runtime. However In unit tests you'll have to tell the framework explicitly what you want logging for. 

TODO: Mock() - what does it do?

#### Mocking Domain Classes

Mocking domain classes is a quick and easy way to provide your test code with input. Mocking domain classes is easy, either you provide a ``@grails.test.mixin.Mock`` annotation with the domain classes you need, or you use the
``mockDomain()`` method which is added to your test by the ``GrailsUnitTestMixin``. One nice thing about the mocked domain objects is that they support most Gorm operations, including criteria and dynamic finders. 

If you need to provide your test with data through mocks you can either just create the objects and save them or you can pass i


Example:

{% highlight groovy %}
@TestFor(MyController)
@Mock([ AddressBook ])
class MyControllerSpec extends Specification {

	void setup() {
		// Explicitly create a new mocked domain object:
		AddressBook addressbook = new AddressBook(name: 'Personal Addressbook')
		addressBook.save()

		// Or provide an entry as a list.
		mockDomain(Entry, [
			[ firstName: 'Hans', lastName: 'Dampf', addressBook: addressBook ], 
			[ firstName: 'Dr', lastName: 'Evil', addressBook: addressBook ]
		)
	}

	void "test something"() {
		// You can now use most Gorm related features, like get:
		AddressBook.get(1)

		// Dynamic finders:
		Entry.findByFirstName('Hans')

		// or Criteria:
		Entry.withCriteria {
			eq('lastName', 'Evil')
		}
	}
}
{% endhighlight %}


**Note**: if you create test data with mocked domain objects, the constraints on the domain class *do* apply, even when using ``mockDomain()``! I've tripped over this a few times. If your mocked domain objects don't show up, make sure
you provide all the necessary fields. 


### Testing Controllers

Testing controllers is straightforward, but requires lots of mocking as they rely on many different things: services, command objects, domain objects, etc. 


#### Testing Content Negotiation

Grails allows automatic content negotiation using the [withFormat](http://grails.org/doc/2.3.4/ref/Controllers/withFormat.html) block. However, setting the format in unit tests is something I keep tripping over. To set a specific response type use `response.format`:

{% highlight groovy %}
@TestFor(MyController)
class MyControllerSpec {
	void "test response"() {
		response.format = 'xml'
		controller.action()
		expect:
		response.contentType == 'application/xml'
	}
}
{% endhighlight %}

Note that the value to use with `response.format` is *not* the mime type, it is the name of what is set in `grails-app/conf/Config.groovy` in the mime type settings. For example: 

{% highlight groovy %}
grails.mime.types = [
		all: '*/*',
		json: ['application/json', 'text/json'],
		xml: ['text/xml', 'application/xml']
]
{% endhighlight %}

If you wanted an XML response, you'd use `response.format = 'xml'` and *not* `response.format = 'application/xml'`.

- - -

## Integration Tests

### Testing Services

Services are really straightforward to test. You can even get Grails to automatically inject the service into your test:

{% highlight groovy %}
class MyServiceSpecification extends Specification {

	MyService myService // Grails will inject the service

	void "test my service method"() {
		String result = myService.doMoreWork()
		
		expect:
		
		result == 'awesome'
	}
}
{% endhighlight %}

Grails will inject everything that is known to it into your integration test, including services or Grail's application context. 

### Integration Testing Services

Writing integration tests for services is very straightforward. You can get the instance injected directly into your test:

{% highlight groovy %}
class MyControllerSpecification extends Specification {

	MyService myService // Injected by grails

	void "test method on service ..."() {
		service.myMethod()
		// ...
	}
}
{% endhighlight %}


### Integration Testing Controllers

Integration testing controllers is just one step away from a full functional test, but has the advantage you don't need to deal with the HTTP details. Unlike a real functional test you don't require actual HTTP clients but you talk to the controller directly. Running the test in the same VM as the application under test also has the added benefit that you can look directly into the application to verify the correct behaviour, e.g. check if necessarey database tables have been created.

#### Dependencies 

Unlike services, controllers cannot be injected into integration tests so you'll have to create them. If your controller relies on a service, you'll have to manually inject it. You can get the service by getting it injected into your test:

{% highlight groovy %}
class MyControllerSpecification extends Specification {

	MyService myService // We'll need this service for the controller

	MyController controller
	
	void setup() {
		// We have to create our own controller instance:
		controller = new MyController() 
		// Manual dependency injection:
		controller.myService = myService 
	}
}
{% endhighlight %}

#### Request and Response

During integration tests Grails will automatically set mocked [request](http://grails.org/doc/latest/api/org/codehaus/groovy/grails/plugins/testing/GrailsMockHttpServletRequest.html) and [response](http://grails.org/doc/latest/api/org/codehaus/groovy/grails/plugins/testing/GrailsMockHttpServletResponse.html) objects on your controllers. To set request parameters, assign values to `controller.params.myParam`. 

If your controllers use content type negotiation, you'll have to set the expected content type on the `controller.response.format` field. Setting `controller.params.format` will *not* work and caused me all kinds of unexpected behaviour. 

{% highlight groovy %}
class MyControllerSpecification extends Specification {

	void setup() { }

	void "test my controller action"() {
		// Set request parameters:
		controller.params.productId = '42'
		
		// Set a content type:
		controller.response.format = 'json'
		
		// Call the controller action under test:
		controller.myAction()
		
		expect:
		
		controller.response.status == 200
		controller.response.contentAsString == '{"productId":42,"name":"cake"}'
	}
}
{% endhighlight %}


## Other References

### Grails 1.3

Use this fantastic [testing cheatsheet for Grails 1.3.x.](http://zanthrash.com/grailstesting/UnitTestingCheatSheet.pdf). 




