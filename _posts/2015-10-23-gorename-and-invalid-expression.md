---
layout: post
title: "gorename and invalid expression"
description: ""
category: 
tags: [Golang, Go, zsh, refactoring]
---
{% include JB/setup %}

This took me longer to figure out than I care to admit, so here's the solution.

The issue comes up when trying to use [gorename](https://godoc.org/golang.org/x/tools/cmd/gorename):

{% highlight console %}
$ gorename -from "github.com/ilikeorangutans/foo".MyType -to 'MyBetterType'
gorename: -from "github.com/ilikeorangutans/foo.MyType": invalid expression
{% endhighlight %}

Even though the `from` query looks normal, `gorename` just refuses to work. However the issue is not so much with `gorename` but rahter my shell, zsh. Turns out properly escaping your from query, fixes the issue:

{% highlight console %}
$ gorename -from '"github.com/ilikeorangutans/foo".MyType' -to 'MyBetterType'
Renamed 15 occurrences in 5 files in 1 package.
{% endhighlight %}

Notice the single quotes around the entire `from` parameter. 
