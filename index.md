---
layout: page
title: Well, hello there!
tagline: Stay awhile and listen...
---
{% include JB/setup %}

Welcome to my little blog. This blog is the successor of my [old blog](http://www.jakusys.de/blog). I write about things that fascinate me, things that took me a long time to figure out or generally just about geeky stuff. 

##Recent Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## To-Do

This theme is still unfinished. If you'd like to be added as a contributor, [please fork](http://github.com/plusjade/jekyll-bootstrap)!
We need to clean up the themes, make theme usage guides with theme-specific markup examples.


