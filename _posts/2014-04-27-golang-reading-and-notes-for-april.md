---
layout: post
title: "Golang Reading and Notes for April 2014"
description: ""
category: 
tags: [ Go, Golang, Reading List ]
---
{% include JB/setup %}

Last week I attended the [Toronto Golang Usergroup Meetup](http://golang.meetup.com/cities/ca/on/toronto/) and it was plenty of fun. If you're in or near Toronto and like to dabble with Go, come out. Oh, and did I mention free pizza? 

### Notes

* Casting in Go is slightly different than in C related languages. Instead of a cast, you perform [a type conversion](http://golang.org/doc/effective_go.html#interface_conversions):

		var myVariable SomeGenericType = ...
		
		casted, ok := myVariable.(MoreSpecificType)
		// ok is a bool		
		if ok {
			// Type conversion successful 
		} else {
			// myVariable does not implement MoreSpecificType
		}
		
* The `range` keyword when used with two return values does not return references, but rather copies. This had me struggle for a while as my code was not behaving as I thought it would. I had a slice of structs and was happily iterating over it: 


		type Foo struct {
			a string
		}
		
		func main() {
			foos := []Foo{
				Foo{a: "horray"},
				Foo{a: "yeay"},
			}
		
			for i, value := range foos {
				// This works because we directly access the struct stored 
				foos[i].a = "Changing the value" within the slice

				// Won't work because value is a *copy* of the struct found in 
				value.a = "this won't work" the slice
			}
		
			for _, value := range foos {
				println(value.a) 
			}
		}
	
  [Executable snippet here](http://play.golang.org/p/j4tZAKppMn). In retrospect it makes sense, Go always passes copies to functions. But initially I was puzzled as my loops wouldn't properly update my structs. 
  
  If you're using reference types, that issue doesn't arise though. Reference types deserve a post on their own, which I will write soon.
				
### Go Reading List

* [Comparing Go and Erlang](http://blog.erlware.org/2014/04/27/some-thoughts-on-go-and-erlang/): some interesting thoughts on shortcomings of Go, including that Go offers `nil`. This is an argument I've read about quite a few times now and is always along the lines of "_Modern languages should not have a nil/null_". In addition this article 
* [Go Best Practices for Production Environments](http://peter.bourgon.org/go-in-production/): lots of experience from the guys at [Soundcloud](https://soundcloud.com/) about how they're using Go. Covers development environment, logging, dependency management, and builds & deploys. 
* By Peter Bourgon, the author of the previous post, a presentation [Go Do](http://peter.bourgon.org/go-do/#1) on why Go.
* Blog post introducing [Go Learn](http://www.sjwhitworth.com/machine-learning-in-go-using-golearn/), a machine learning library in Go.
* [Go's power is in emergent behavior](http://www.onebigfluke.com/2014/04/gos-power-is-in-emergent-behavior.html)
