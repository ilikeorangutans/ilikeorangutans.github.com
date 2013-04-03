---
layout: post
title: "Date Arithmetics in XSLT 2"
description: ""
category: 
tags: [ XSLT, XML, XPATH ]
---
{% include JB/setup %}

Now here's something I didn't know: XSLT 2 and XPath actually support date arithmetic! Took me a while to figure it out, but here's how it works.

First, all your dates will have to be in [ISO-8601](http://en.wikipedia.org/wiki/ISO_8601) format. For dates only it looks like this: ``YYYY-MM-DD`` and for dates and times, like this: ``YYYY-MM-DDTHH:mm:SS.sssZ``. There's a few other formats, but these are the ones that probably cover all use cases. 

In order to make use of all the functions regarding date and time, the values will have to be converted into the appropriate types. They data types are defined in the XMLSchema namespace http://www.w3.org/2001/XMLSchema and are:

* ``xs:date`` a single date
* ``xs:time`` a timestamp
* ``xs:dateTime`` a combination thereof

In order to use them, declare the ``xs`` namespace in your stylesheet:

{% highlight xml %}
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform" version="2.0" xmlns:xs="http://www.w3.org/2001/XMLSchema">
{% endhighlight %}

Then you have several ways of creating instances of the above variables:

{% highlight xml %}
<!-- current date, time, date and time: -->
<xsl:value-of select="current-date()" />     <!-- 2013-04-02-04:00 -->
<xsl:value-of select="current-time()" />     <!-- 22:18:47.48-04:00-->
<xsl:value-of select="current-dateTime()" /> <!-- 2013-04-02T22:18:47.48-04:00 -->

<!-- create date, time, dateTime from strings: --> 
<xsl:value-of select="xs:date('1983-06-27-04:00')" />
<xsl:value-of select="xs:time('16:00:00-04:00')" />
<xsl:value-of select="xs:dateTime('1983-06-27T16:00:00+04:00')" />

<!-- of course that works with properties and nodes as well: -->
<xsl:value-of select="xs:dateTime(some/element/birthday)" />

<!-- and you can create variables: -->
<xsl:variable name="var" select="some/element/birthday" as="xs:dateTime" />
{% endhighlight %}

But there's more. In addition of the data types that represent points in time, a data type to represent durations exists: ``xs:duration``. In its string representation it looks a bit funny. The "P" indicates "period", and "T" stands for "time": ``P1Y2M3DT4H5M13S`` (meaning: 1 year, 2 months, 3 days, 4 hours, 5 minutes, and 13 seconds). Other than that, I found the ``xs:duration`` data type fidgety and hard to use. Part of that has to do with certain [ambiguities](http://www.stylusstudio.com/xsllist/200409/post50020.html) that make it almost impossible to use arithmetic operations on them. However, *extracting* information from duration instances is easy:

{% highlight xml %}
<!-- Put the duration in a variable for better readability: -->
<xsl:variable name="duration" select="xs:duration('P1Y2M3DT4H5M13S')" />
<xsl:value-of select="hours-from-duration($duration)"/>   <!-- 4 -->
<xsl:value-of select="days-from-duration($duration)"/>    <!-- 3 -->
<xsl:value-of select="months-from-duration($duration)"/>  <!-- 2 -->
<xsl:value-of select="years-from-duration($duration)"/>   <!-- 1 -->
{% endhighlight %}

The return values of the ``*-from-duration()`` methods can be used to perform calculations. 

Once you have your dates and times in the correct datatype, you can perform arithmetic using the familiar operands ``+ - * /``. Almost all operations return instances of ``xs:duration`` that can 

{% highlight xml %}
<!-- Let's put the dates in variables for better readability: -->
<xsl:variable name="now" select="current-dateTime" />
<xsl:variable name="past" select="xs:date('1983-06-27+02:00')" />
<xsl:value-of select="$now - $past"/>    <!-- P10872DT15H19M40.125S -->
{% endhighlight %}

I hope these notes are useful to anybody who has to deal with dates and times in XSLT. Unfortunately, I found most of the documentation on the web incomplete or more confusing than helpful. [XSLT, 2nd Edition](http://amzn.to/ZyYJfg) turned out to be very helpful and I recommend it to every developer working with XSLT and XML.

