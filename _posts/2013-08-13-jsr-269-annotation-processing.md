---
layout: post
title: "JSR-269 Annotation Processing"
description: ""
category: 
tags: [ Java, Annotation, JSR-269 ]
---
{% include JB/setup %}

I've been contemplating compile time bytecode manipulation for the [Object Mapper Framework](http://www.objectmapper.org/) for a while now. Compile time instrumentation of classes seems to be a better approach and does away a whole lot of class loading issues, especially in OSGI environments. In any case, I remembered [Project Lombok](http://projectlombok.org/) and reading about [JSR-269](http://jcp.org/en/jsr/detail?id=269), which was introduced with Java 1.6. It's an API that allows you to plug custom annotation processors into javac.

After experimenting and reading a bit, here is my reading list:

* [ZDNet](http://www.zdnet.com/writing-and-processing-custom-annotations-part-1-2039352825/) has a good tutorial ([Part 2](http://www.zdnet.com/writing-and-processing-custom-annotations-part-2-2039357034/), [Part 3](http://www.zdnet.com/writing-and-processing-custom-annotations-part-3-2039362483/))
* [JavaBeat](http://www.javabeat.net/2007/06/java-6-0-features-part-2-pluggable-annotation-processing-api/) tutorial
* [Stackoverflow discussion](http://stackoverflow.com/questions/6967514/maven-example-of-annotation-preprocessing-and-generation-of-classes-in-same-comp) outlines the high level steps very well
* [cdivilly](http://cdivilly.wordpress.com/2010/03/16/maven-and-jsr-269-annotation-processors/) helps getting around the paradox situation when you try to build a JSR-269 processor with Maven (long story short: in order for the compiler to pickup the processor, you need a [service](http://docs.oracle.com/javase/7/docs/api/java/util/ServiceLoader.html) descriptor in ```META-INF/services```; however, if you have that file on the Maven classpath, which you usually do in ```src/main/resources```, the compilerwill try to use the processor it's supposed to compile... obviously, that won't work too well)
* [dr. macphail](http://deors.wordpress.com/2011/09/26/annotation-types/) has another great tutorial ([Part 2](http://deors.wordpress.com/2011/10/08/annotation-processors/), [Part 3](http://deors.wordpress.com/2011/10/31/annotation-generators/))

It's an interesting concept and I got a simple processor to work. Next steps will be to experiment with bytecode weaving/generation and see if I can pregenerate proxies with this.
