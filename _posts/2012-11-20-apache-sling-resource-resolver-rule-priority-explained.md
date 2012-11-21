---
layout: post
title: "Apache Sling Resource Resolver Rules in a Nutshell"
description: ""
category: 
tags: [ Apache Sling, JCR, OSGI, Regexp ]
---
{% include JB/setup %}

If you work with Apache Sling, you have probably encountered the ResourceResolver and its configuration rules. 
In short, the ResourceResolver is the part of Sling that resolves incoming requests to actual or virtual 
resources. For example, if a request for ``/foo/bar`` is coming in the resolver will resolve that to a 
corresponding node in the JCR. However, sometimes it is not desireable to expose the internal structure of the
repository or the required external structure cannot be represented using the JCR. In that case the resolver
can be customized by installing resolver rules. Resolver rules can be modified via the OSGI configuration 
editor or the OSGI ConfigurationAdmin. 

Each rule consists of three parts. I'll call them the *internal path*, the *operator*, and the *external path*.
The syntax is always like this:

	<internal path>   <operator>   <external path>

In order to understand what *internal path* and *external path* mean, we need to discuss the *operator* first.
However, in order to understand the operators, we'll have to take a quick look at the two fundamental operations
the resolver performs: *resource resolution* and the *resource mapping*.

Resource resolution, as outlined in the introduction, is the process of resolving an incoming request to a 
resource that Sling can provide, usually a node in the JCR. It's important to note that the incoming request's 
path doesn't need to correspond to anything in the JCR.

In order to specify a rule that should be applied to the request URI during resource resolution, you would 
specify a resolver rule using the ``>`` operator. For example, to map all requests for ``/foo`` to 
``/content/bar`` in the JCR, you would write the following:

	/content/bar>/foo

With this example it becomes obvious what *internal path* and *external path* stand for. In case of a resolver
rule, *internal path* is the path the matching external path is replace with. And in order for resolver rules
to be really useful and scare of the faint of heart, they support regular expressions. So, you can utilize
character and matching groups and references:

	/content/$1/$2$3>/([a-zA-Z]{2})/(en|fr|de)(.*)

This resolves requests for ``/us/en/homepage.html`` to ``/content/us/en/homepage.html`` and ``/ca/fr/foobar.html``
to ``/content/ca/fr/foobar.html``. Neat.

Resource mapping can be thought of as the opposite of resource resolution. Let's assume the following resolver 
rule:

	/content/bar>/foo

For incoming requests you are covered, clients will only see ``/foo``. However, in the rendered pages, references
to ``/content/bar`` will stills how up as such. So in order to "hide" this path, we need to *map* it to an 
external path. This is where the second operator come in. You can specify a map rule using the ``<`` operator.
For example, to replace ``/content/bar`` with ``/foo``, you'd specify the following rule:

	/content/bar</foo

Notice how the order of the operands is unchanged, which makes these rules somewhat confusing to work with. 
Of course, you can use regular expressions to make your rules more powerful and more confusing to simpler minds:

	/content/ca/(en|fr)</ohcanada/$1

Again, the order of the operands is important. And in this case, notice how the regular expressions have 
"switched" side. The matchers are on the left hand, the refrences on the right hand side. Once applied to 
a path, for example ``/content/ca/en/homepage``, the mapping rule will transform it into ``/ohcanada/en/homepage``.
Please note that mapping rules are not magically applied. You need to either call [``resourceResolver.map()``](http://sling.apache.org/apidocs/sling5/org/apache/sling/api/resource/ResourceResolver.html#map(java.lang.String)) for
your path, or use an [output rewriting pipeline](http://sling.apache.org/site/output-rewriting-pipelines-orgapacheslingrewriter.html).

The flexibility of specifying resolver and mapper rules separately, is not needed. In most cases you'd want 
the resolver rules to be reflected in the mapper rules and vice versa. This can be acchieved with the ``:``
operator:

	/content/bar:/foo

This results in both a mapper and a resolver rule. However, regular expression usage here is limited. 

Now, let's discuss one last aspect, that I've been silently ignoring so far. How does Sling behave if there
are multiple rules? First, Sling orders all rules by the number of regular expression groups it contains.
(I haven't found any reference to this in the Sling docs, this is knowledge earned through sweat, blood, and
methodical head banging!)
The more groups, the higher the priority of the group. For each incoming request, Sling will try to match 
it against all rules in descending order of their priority. First rule that matches, wins. The following
rules for example, have different priorities:

	/content/foo/$1>/(pie|cake)
	/content/bar/$1/$2>/(apples|oranges)/(.*)

The first rule has a single group whereas the second group has two groups. During the resource resolution
process, the second rule would always be matched *before* the first rule. 


Once understood, resolver and mapper rules offer a versatile and flexible tool for masking content structures 
or offering new external URLs for services. However, the road to understand them, is, at least for me,
a stony if not rocky one. I hope this post will help others to understand and implement them. 
