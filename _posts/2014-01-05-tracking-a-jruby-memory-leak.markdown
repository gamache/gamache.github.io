---
layout: post
title: Tracking a JRuby Memory Leak
date: '2014-01-05 08:26:06 -0500'
tags: api code jruby
---
I wrote an API in JRuby on Rails, and it hemorrhaged heap memory. The JVM
leaks anonymous classes by default. First I didn’t know, then I fought it,
then I ran away. This is my story.

Over the last year at my employer [Localytics](http://www.localytics.com),
a mobile and web app analytics and marketing platform, I wrote the
[Localytics API](https://api.localytics.com/docs), which is a Rails app
deployed on JRuby. The Localytics API provides a simplified query language
for app analytics, and it’s the basis for our customer-facing analytics
dashboard as well as enterprise customers who’d like to run their own
queries. It’s pretty nice, I’ll yap about it more sometime.

But first, the not-so-nice part. It leaked. The server process would eat
heap at a rate roughly proportional to usage, until hitting max JVM heap
size and collapsing in a burst of sparks and slag.

Now, in the absence of loopy data structures, memory leaks in Rails apps are
not the most common thing. I checked and re-checked that I wasn’t doing
anything funky with circular references. Nope.

Eventually I came to suspect a seemingly harmless piece of code (I am going
to call out [Chris Rollock](https://twitter.com/Sastopher)
for planting the seeds of doubt in my head). One
task the API performs was elegantly expressed by creating an anonymous
subclass of a named class, adding some code to it, and sending it on its
way. Not my code, but I like it. The anonymous class was never stashed in
a long-lived data structure, so I thought it would be ok, but when it
comes to telling you that your app is being a total RAM pig, the JVM
doesn’t lie. And my API was still at the trough.

Then I learned that the JVM does not free anonymous classes by default.

I hadn’t seen any reference to this in conjunction with JRuby on the
net before, and asking on Freenode #jruby, I came up empty as well.
Until I came upon
[Groovy having dealt with the same problem](http://groovy.codehaus.org/Running).
Clear as a bell came the sacred incantation:

> Groovy creates classes dynamically, but the default Java VM does not
> GC the PermGen. If you are using Java 6 or later,
> add `-XX:+CMSClassUnloadingEnabled -XX:+UseConcMarkSweepGC`.
> UseConcMarkSweepGC is needed to enable CMSClassUnloadingEnabled.

Hey, that’s me! And I’m using Java 6 or later too!

I’ll let a couple graphs tell the tale. Here’s typical operation without
the fix:

![Heap memory before fix](/images/jruby-leak/image1.png)

And with the fix?

![Heap memory after fix](/images/jruby-leak/image2.png)

Nice. But not quite good enough. It’s still growing! Just because I can wait
8 hours between server reboots instead of 4 doesn’t make me that pleased.

I’ve condensed this tale into a short story, but this @#$!@$%!# leak was in
the back of my mind for weeks. I fought it on and off, but largely I just
cron’ed up server restarts and stewed in my anger. I tried this, I tried
that. Still the slow leak.

Eventually I got tired of worrying about it, and begrudgingly changed the
code. Now, instead of creating an anonymous subclass with extra ornaments,
I just stick UUID-named ornaments onto the main class at runtime and pass
the name along instead of the subclass. Technically this is still a leak,
but ornaments weigh a lot less than Christmas trees.

![Heap memory after avoidance](/images/jruby-leak/image2.png)

I can live with this. For now.

