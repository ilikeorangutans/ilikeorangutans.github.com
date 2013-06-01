---
layout: post
title: "The Case for Continuous Integration"
description: ""
category: 
tags: [ Continuous Integration, Software Development, Unit Testing  ]
---
{% include JB/setup %}

In my career as a software developer, I've come to appreciate the principles of Continuous Integration (CI). It forces you to do the hard things early and often and thus helps you reduce risk during development. It forces you to write tests, and be responsible about what you check in. All in all, good qualities and something that every development team should aspire to. Or so I thought. Reality is different, and so far almost every development team I have interacted with is deadly afraid of doing CI. So much, that there's been near-mutinies because CI and what it means for team, causes so many problems. This is something that puzzles me, and I realize it might be because the teams in question don't fully understand what CI is or only realize a subset of what it means. This is my attempt to demystify and explain CI. 

### Definition: Continuous Integration

According to Wikipedia: 

> [Continuous Integration](http://en.wikipedia.org/wiki/Continuous_integration) is the practice, in software engineering, of merging all developer workspaces with a shared mainline several times a day

Uh-huh. That sounds... trivial... and... wouldn't that cause lots of problems? And... what does that even mean? 

### Understanding the Need for Continuous Integration: a Short Story

Understanding CI based on its definition is possible, but I believe it's easier to understand if it is put in perspective of the problems it addresses. To explain what CI solves and where it helps, here's a short story of a fictive development team working at ACME Inc on an web application. 

The project starts with a meeting where the goals are outlined, and the developers set up a plan. A new branch is created for the project and the developers work with much enthusiasm. Some developers even create branches off the project branch, to keep the code "clean". Meanwhile, some maintenance fixes are done on the original main line branch. 

#### The first milestone

Halfway into the project, the project manager asks for a QA build, but the team cannot get the project branch to compile. Too many in progress changes are on the branch, and a massive refactoring attempt render the code in a broken state. They reassure the project manager that it will be done "tomorrow". 

A week before the launch the project manager gets really nervous and threatens to "not order pizza for the team any more" if a QA build is not up by end of day. In an heroic effort a build is created by the development lead on his computer, that has most of the project code, except for some branches that break the code. The build is on QA, and for the most part it seems good. Some features are missing, some are broken, but many are there. The outstanding issues will be fixed "tomorrow". 

#### Day of the release

It is now the day of the release, 9:00. The updated application is supposed to get deployed at 17:00. After the "successful" QA deploy the team is optimistic and eager to get the release done and celebrate with a cold beer on a patio. The team starts by merging all project branches into the main project branch. Conflicts and broken code ensue and the team works feverishly to get the project branch to work. At 16:00, after a skipped lunch, the development lead manages to get the project branch to build again. A quick check on the QA server, by the patiently waiting QA guys, seems to confirm that it all works. 

Everything looks ok, until the team attempts to merge the project branch into the main branch with the maintenance fixes. There are tons of conflicts and the code is broken again. The developers try to pacify the upset compiler while eating sandwiches at their desk. The project manager is seen pacing in circles behind the development lead's desk. 

#### Release build finally compiles

At 18:30 the code finally builds again and the team rejoices. The development lead creates the release package and it gets deployed to QA. The QA folks, not exactly thrilled about the late deploy, start testing the new application. After the first cursory glance at the app it is dryly noted that some existing features don't work as expected, and some of the new features are missing. The team scrambles to identify what went wrong, and they realize that during the last merge some files got lost. However, they cannot produce a new release package, because the development lead is the only one that has the scripts set up to build the release package. But he is out to finally get some food. After a quick discussion with the system administrators it is decided to just manually upload the missing and changed files. After all, it's only a handful of files, and that's what hotfixes are for, right?

#### "Go" for production

At 19:15 the files have been deployed, and QA commences testing. No major recessions or new bugs are found, and the few minor issues are noted as known issues. The applications is good to go to production. 

#### "Houston, we've had a problem"

It is now 19:45, and the application was just deployed to all production servers. The first quick inspection reveals that nothing works. The developers are in disbelief. Phrases like "*but it worked on QA!*" or "*it never did that before!*" are heard. 

After the project manager has been revived from his sudden triple cardiac arrest, options are triaged. A rollback is not an option, the new features have been promised to the clients. Also, when deploying, the previous package was overwritten with the new one and nobody can find a copy. The team decides to bite the bullet and investigate the problem. 

Two hours later, after skimming logs and extensive debugging the team is not one bit closer to the solution. A stack of fresh and hot pizza temporarily raises the mood and developers gorge themselves on the delicious combination of cheese, tomato sauce and elaborate toppings. Sugary pops complement the feast. Everybody eats two slices more than he should, because the company pays for it. 

Yet another two hours later and additional slices of cold pizza, at midnight, the problem is found. The problem turns out to be a core library, that was updated on all the developer machines and QA, but not on production because it was part of a faulty deploy that never went past QA. Everybody is excited that the ordeal is finally over. The library has been isolated and is being deployed by the administrators to the servers. After a restart, the application comes up. The QA guys, which by now have read *all* of Reddit, get to work and quickly report that the same problems from the first QA launch are back. Oh right, the hotfix! The hotfix gets installed, and everything seems fine. 

At 2:00 in the morning the team finally gets approval from the QA guys and leaves the office. But not before the project manager sends an all-staff email praising the team as "rockstars" for all their hard work. Exhausted but proud to have solved all the hard problems that the project has thrown at them...

*The end.*

Now I ask all the developers that are familiar with situations like the one described above to raise their hands. Actually, don't raise your hand, comment below, I would love to hear your story. ;) My guess is that many of you have been through projects like that. I certainly have been through a few like that…

### What went wrong?

Failed projects are a great way to learn and improve yourself. Whenever I go through a project that fails or end like the one described above, I try to analyze what went wrong and learn from it. So, here's a rough list of mistakes that were made:

* Changes from the main branch are never merged back; the project code base went dark after the first day. 
* Project code is broken most of the time. The team can't even build it. 
* The first real QA test, with both the project and the maintenance changes incorporated, is done... when it's too late. 
* Only the development lead has the ability to create builds. 
* QA deploys were not the real deploys, i.e. they were built with only a subset of the code.
* Deploys are done manually. The deployed application is not atomic and gets "patched". 
* Where are the tests?!?
* Environments are a mess! Why are different libraries on the different servers? 
* People get praised for the amount they work, not for the quality they produce. 
* Late night operations where many manual steps have to be performed in the right order. 

I'm sure you can find more problems, but in my opinion these cover a lot of ground. All these points can be summarized in the following points:

* **Isolation**: development happened in isolation. Branches diverged and caused plenty of conflicts, making it harder for the team to actually compile and build. 
* **Late Feedback**: quality and functionality of the code base is only established hours before the deploy, the team is effectively "flying blind". 
* **Manual Work**: each build and deploy require manual steps, introducing lots of potential for mistakes, especially late at night after 7 slices of pizza.
* **Information Monopoly**: project critical steps like building the application package, can only be done by certain people. Just think about the [bus](http://www.johndcook.com/blog/2008/09/28/programmer-hit-by-a-bus/)...

### Continuous Integration 

With these four categories in mind, lets re-examine what CI is. The fundamental principle is that developers merge their changes back into the main line as often as possible, ideally multiple times a day. Obviously there's more to it, but all other points can be derived from this single principle. 

Let's take a look how CI could have enabled the team to avoid a lot of the trouble it ran into and enjoy its well deserved beer on the patio instead of spending all night in the office. 

#### Isolation

Imagine, that the team would have worked on the main branch instead of an isolated tree of branches that kept diverging from the mainline. If the team would have *continuously* merged their work into this branch, the amount and complexity of the merges would have drastically been reduced. Imagine dealing with conflicts from code that has actively diverged into different directions for months. And let's be honest, conflicts are hard. Conflicts that affect multiple files and/or semantic changes are a nightmare to deal with. 

*By merging code often and early, the chance for conflicts is drastically reduced. The team could have saved itself lots of hours that it spent trying to fix conflicts.*

#### Late Feedback

Remember how the team only got feedback that features were broken at the day of the deploy? Wouldn't it be great if the team got feedback that they broke existing features *the moment they checked into the mainline*? *Impossible* you say? Not quite. With the cunning use of automated tests, you can get almost instant feedback. In addition to the feedback, automated tests are a great safety net for changes to your code, improve the quality of your code (seriously, they're like Vitamin C for source code!) and have [positive impact on the design of your application](http://vimeo.com/15007792). Much has been written on this topic, and I highly recommend [Junit in Action](http://amzn.to/TvNLaH), [Growing Object-Oriented Software, Guided by Tests ](http://amzn.to/198ygh8), and [Working Effectively with Legacy Code](http://amzn.to/198ymW2).

But there's more. Think about how thorough the QA testing in the above story would have been. QA had pretty much 2 hours in the middle of the night to perform testing of the code. I can imagine that more than one issue would have been missed or would have just been added to the known issues list. 

*If the team would have had unit and functional tests in place that checked the existing functionality, it would have found out about the regressions weeks in advance, not the night of the deploy. Would have the team been able to provide full builds to QA at a much earlier stage, the quality of the application could have been improved considerably.*

#### Manual Work

All the builds and the releases the team did, were performed manually. At best it was done during the day, at worst it was done under stress late at night. The deploys were performed manually, with manually created packages. And we've seen how much trouble it caused the team. Manual steps are bad. If you do something more than twice, automate it. It takes a bit time in the beginning, but it pays off fast, in three ways:

 *  Automating a process requires the team to sit down and think about what it is that they do and to codify it in a script. Once that script is in source control, it can be run by everybody, and changes to the process are recorded and visible. 
 * The process will always work the same, no matter the time, how much pizza its operator ate, or the stress level at that current situation. 
 * What better and more accurate documentation is there than the script that actually performs the work. But more on that in the last point.

*If the packaging and deploys would have been automated, a lot of time could have been saved. The team would not need to spend time trying to create a release package and get it deployed.*

#### Information Monopoly

Remember how the team constantly was blocked when somebody that exclusively knew about an important step in the process, was not there? This is a theme I've seen in many places and it can be a real problem in any kind of project. It can range from creating branches to deploying packages, from setting up a new development instance to clearing some obscure problem. The theme is always the same: certain tasks depend on one particular person, and if that person is unavailable, the project is stumped. 

Usually it's the only person that knowns the ``root`` password for the production servers or the only person that knows how to create the release packages. Usually this is called the bus scenario, where the luckless person is run over by a large vehicle employed by the local public transportation company. But it doesn't have to be a bus, it's usually enough if the key person is busy on other important stuff, which is usually the case. Suddenly your team is stumped and unable to make progress.

*If the release preparation and packaging would have been well documented, and even better, automated, the absence of the development lead would have not impacted the progress of the project.*


### A Shift in Thinking

Adopting CI requires your team to change the way it thinks about software development and delivery of its projects. The following is a list of concrete principles that will address a lot of the issues we've outlined above.

* **Avoid long running feature branches**: don't give in to the temptation to just put all the code for the big new project into its own branch. Instead, break up the project into smaller tasks, that all can be done in a few days. Make sure you check in the change
* **Invest in automated tests**: a fundamental pillar of CI is that you get rapid feedback when you commit changes. The only reliable way to do so, is to provide automated tests and have them executed. As I've outlined earlier, tests have many benefits, and the safety net against regressions is what makes them so important in CI. 
* **Automate everything**: if you do something more than twice, consider automating it. Prime candidates are creating a release package, deploying to a server instance, running tests, setting up instances of environments, setting up development environments, etc. 
* **Version everything**: make sure all source code, automation scripts, configuration files, and documentation is versioned. In the same way all code is versioned, you want to be able to easily retrieve the latest version of a deploy script, or review the changes in a server config file. 

No matter at what stage you are in your project, you can always pick one or more of the principles and apply them to your project. 

### Common Challenges

Introducing CI to a development team that is unfamiliar with its principles is a tough sell. Here are some of the common reasons why we can't adopt CI I get and how I counter them. If you have any other common discussion points, email me or drop a comment. :)

#### "*We can't have developers check in code for project XYZ, it's not supposed to go live yet!*"

I've heard many variations of this, but the theme is always the same: "*We can't release it yet because it contains time sensitive changes*" or "*It's for a marketing campaign that only goes up next week*". 

The theme is always that the team is afraid of putting something out there, that shouldn't be out there yet. This leads to all kinds of paradox behaviour and situations:

> We'll need to QA this now [a month before it's live], but we can only test the current branch.
 
But then the code sits on the branch for a month, is unmaintained, and only merged on the day it's supposed to go out. We've seen what last minute merges and deploys look like...

> To get this project out there, we have to deploy a perfect build at exactly that date and time! 

The irony here is that the team has a lot of time that it could use to test and validate the new changes, but instead wastes this opportunity and then tries to perform all changes at the latest moment possible. 

The result is usually the same. A project that is done ahead of time and has a well known deadline, screws up because in the last minute lots of unforeseen problems come up. 

The approach that usually comes to mind are [feature toggles](http://martinfowler.com/bliki/FeatureToggle.html). A feature toggle introduces conditional logic to enable or disable certain parts of an application. Instead of having every new piece of functionality enabled per default, an environment specific configuration file can enable or disable functionality. It is a very elegant solution and allows your QA team to test the feature well in advance in context of all the other ongoing changes. At the same time the feature can be safely disabled in production and then only needs to be activated at the release date.

While this works nicely for new features, it is often challenged on the grounds of changes to existing features. The question I often get is "*How do we deal with changes to existing functionality?*". The answer requires another shift in thinking. Instead of changing the existing code, it's often better to abstract that behaviour into separate classes/functions (think [strategy pattern](http://en.wikipedia.org/wiki/Strategy_pattern)). You can then implement the changed behaviour as a new strategy and switch it using feature toggles. 

Once you get into the habit of hiding features behind toggles, you'll start thinking about the development process differently. Usually in a better way. 

#### "If everybody can build and release, doesn't that mean mayhem?"

The traditional approach to software development is that releasing software to any environment should be strictly limited. Only people with certain privileges are allowed to perform a release and get it onto the QA server. But with the automation that CI encourages, a team usually has automated jobs that perform these tasks. This is often heavily fought by the system administrators and project managers because they fear that the developers will introduce breaking changes and generally make their life more miserable. 

This argument would be valid in a chaotic environment, where releases and builds not well defined. Think about the cobbled together builds in our short story. However, what we're facing here is that administrators and project managers want to ensure that no broken build ever reaches the production servers. And until the team started to adopt CI, it was an adequate mechanism to control the flow of code. The administrators and project managers sought to control risk by postponing it and obviously, by not imposing restrictions on these important steps in the process, you are just asking for trouble.

Unfortunately one important point always gets ignored or is not correctly understood. While PMs and administrates often seek to control risk by postponing it, CI is all about identifying risks as early as possible. Merging early as often, tells you early if you'll encounter conflicts. Running automated tests on every commit tells you if you broke any existing functionality (assuming you have sufficient coverage). Deploying all features to QA as early as possible gives QA a chance to thoroughly test everything in advance. 

To me it's paradox that this requires discussion, but if you think about it, would you want to know about the merge and regressions you're introducing the day you do them or rather two months later at release day? 

At the end of the day, my response to this concern is that CI is all about identifying risks as early as possible. And that's undeniably a good thing.

#### "If we automate everything, what is there to do for administrators?"

This is usually brought up by administrators that fear for their job. In their opinion, the complicated release and deploy processes are where they add value. I tend to disagree; any repetitive steps that can be automated with a 20 line shell script executed manually by a highly paid professional actually reduce value. In my opinion the role of a administrator or release engineer is to *oversee* the release and deploy process, and ensure stability of their environments. This entails a lot of different activities, that are usually neglected: systems monitoring, performance tuning, security audits and hardening, change management, configuration management, just to name a few. 

The way I approach this discussion is to point out that automation of mundane tasks will actually allow administrators to focus on what they are being hired for: system administration. 

#### "We've started merging early, and now we have all these problems!"

This is a challenge I personally found very infuriating. In a project where we started to adopt CI, the developers started to dislike it because they kept "*getting all these problems*" all the time. For example, merge conflicts, challenges in coordinating concurrent projects, or having to split your work into smaller tasks. 

The irony here is that all the problems they were getting were neither new nor unexpected. But because they happened **during** the project, they were more visible. When problems like these are encountered at the release day, the team just works through it and blames it on too tight deadlines or bad project management, and they're quickly forgotten. All that remains is a sense of terror and fear towards release dates.

The best way to deal with this is to look at the success rate of the past releases. In my case, we've already had a track record of high quality releases and with maybe one exception never had to stay overtime at work to fix a release last minute. Once you show your team that you're dealing with problems ahead of time they'll see their woes in a different light.

### Final words

CI requires a shift in thinking, from developers, administrators, project managers and managers. **It requires you to do the hard things early and often** and that scares a lot of people. **It forces you to analyze the tricky part of your project as soon as you can, not as late as possible**, and people don't like doing the hard things. **You'll have to automate a lot of manual steps**, and some people are afraid that this will make their job superfluous.

These are all reasons why it is hard to introduce CI into a development workflow. However, if you succeed in only a few of the steps I've outlined in this document, you will already see a significant improvement in your teams workflow. "Hard" things will suddenly become a lot easier, things that took a lot of time from a developer, suddenly can be done by a script over lunch, and scary things suddenly become less scary, because you keep doing them all the time. 

I hope I was able to present you a good overview of what CI is, what it means and what it *doesn't* mean, and how to convince critics. Good luck. 
