---
date: 2013-02-06T00:00:00Z
description: ""
tags:
- OSGI
- Maven
- SCR
title: Type Incompatibility With Maven SCR Plugin
url: /2013/02/06/type-incompatibility-with-maven-scr-plugin/
---



Yesterday I ran into a most strange error when compiling one of my OSGI bundles:

	[ERROR] Failed to execute goal org.apache.felix:maven-scr-plugin:1.7.4:scr (generate-scr-scrdescriptor) on project XXX: A type incompatibility occured while executing org.apache.felix:maven-scr-plugin:1.7.4:scr: com.thoughtworks.qdox.model.Annotation cannot be cast to java.util.List

I saw several variations of that error and was initially clueless as how to fix it. However, I eventually found the problem:

{{< highlight java >}}
@Service(value = { MyService.class, ManagedService.class })
@Component(immediate = true)
@Properties(@Property(name = "foo", value = "bar"))
public class NissanAutoDataUrlGenerator implements ManagedService { }
{{< / highlight >}}

The mistake is that the ``@Properties`` annotation takes an array as it's parameter. Eclipse does not mark the above as an error, however, the Maven plugin is not happy about it. Here's the correct code:


{{< highlight java >}}
@Service(value = { MyService.class, ManagedService.class })
@Component(immediate = true)
@Properties({ @Property(name = "foo", value = "bar") } )
public class NissanAutoDataUrlGenerator implements ManagedService { }
{{< / highlight >}}
