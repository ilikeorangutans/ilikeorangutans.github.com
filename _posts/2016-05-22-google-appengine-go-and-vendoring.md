---
layout: post
title: "Google Appengine, Go, and Vendoring"
description: ""
category:
tags: [ Golang, Go, Google App Engine ]
---
{% include JB/setup %}

I'm working on a small app running on Google App engine using Go and upgraded to the latest version of the GAE SDK. The latest version uses Go 1.6 instead of 1.4 like the older version I had. Upgrading was mostly straightforward, but once I started using vendoring I got strange build errors like this:

    2016/05/22 13:26:47 go-app-builder: Failed parsing input: parser: bad import "syscall" in vendor/golang.org/x/net/ipv4/dgramopt_posix.go

I got different variations of this, but all came down to the same problem: some code was importing packages that GAE doesn't want you tu use. Sadly these errors don't show up during normal `goapp build` or `goapp test` cycles, but only when you want to deploy or start a local devserver.

The solution turned out to be surprisingly simple: add the `vendor` directory to the `nobuild_files`, like so:

    nobuild_files:
    - vendor

Now both builds and deploys run perfectly.
