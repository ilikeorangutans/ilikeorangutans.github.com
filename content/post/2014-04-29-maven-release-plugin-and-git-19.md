---
date: 2014-04-29T00:00:00Z
description: ""
tags:
- Maven
- Maven Release Plugin
- Git
title: Maven Release Plugin and Git 1.9
url: /2014/04/29/maven-release-plugin-and-git-19/
---



Ran into this issue today and wasted a good hour on figuring out what happened. Seems to be an issue with Git 1.9.x. 

### Symptoms

Maven release plugin successfully completes the `release:prepare` goal, creates the tag, but fails to commit the changed `pom.xml`. 

### Solution 

Git's output format changed slightly in one of the recent versions and Maven's SCM provider expects output in a certain way. Luckily you can force git to use the old output style:

		git config --add status.displayCommentPrefix true

[As per SCM-740](http://jira.codehaus.org/browse/SCM-740?focusedCommentId=341325&page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel#comment-341325)

