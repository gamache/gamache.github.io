---
layout: post
status: publish
published: true
title: Tracking a JRuby Memory Leak
author:
  display_name: gamache
  login: gamache
  email: pete@gamache.org
  url: ''
author_login: gamache
author_email: pete@gamache.org
wordpress_id: 145
wordpress_url: http://petegamache.com/?p=145
date: '2014-01-05 08:26:06 -0500'
date_gmt: '2014-01-05 13:26:06 -0500'
categories:
- Uncategorized
tags: []
comments: []
---
<p>I wrote an API in JRuby on Rails, and it hemorrhaged heap memory.  The JVM leaks anonymous classes by default.  First I didn't know, then I fought it, then I ran away.  This is my story.<&#47;p></p>
<p>Over the last year at my employer <a href="http:&#47;&#47;www.localytics.com&#47;" target="_blank">Localytics<&#47;a>, a mobile and web app analytics and marketing platform, I wrote the Localytics API, which is a Rails app deployed on JRuby.  The Localytics API provides a simplified query language for app analytics, and it's the basis for our customer-facing analytics dashboard as well as enterprise customers who'd like to run their own queries.  It's pretty nice, I'll yap about it more sometime.<&#47;p></p>
<p>But first, the not-so-nice part.  It leaked.  The server process would eat heap at a rate roughly proportional to usage, until hitting max JVM heap size and collapsing in a burst of sparks and slag.<&#47;p></p>
<p>Now, in the absence of loopy data structures, memory leaks in Rails apps are not the most common thing.  I checked and re-checked that I wasn't doing anything funky with circular references.  Nope.<&#47;p></p>
<p>Eventually I came to suspect a seemingly harmless piece of code (I am going to call out <a href="https:&#47;&#47;twitter.com&#47;Sastopher" target="_blank">Chris Rollock<&#47;a> for planting the seeds of doubt in my head).  One task the API performs was elegantly expressed by creating an anonymous subclass of a named class, adding some code to it, and sending it on its way.  Not my code, but I like it.  The anonymous class was never stashed in a long-lived data structure, so I thought it would be ok, but when it comes to telling you that your app is being a total RAM pig, the JVM doesn't lie.  And my API was still at the trough.<&#47;p></p>
<p>Then I learned that the JVM does not free anonymous classes by default.<&#47;p></p>
<p>I hadn't seen any reference to this in conjunction with JRuby on the net before, and asking on Freenode #jruby, I came up empty as well.  Until I came upon <a target="_blank" href="http:&#47;&#47;groovy.codehaus.org&#47;Running">Groovy having dealt with the same problem<&#47;a>.  Clear as a bell came the sacred incantation:</p>
<blockquote><p>
Groovy creates classes dynamically, but the default Java VM does not GC the PermGen. If you are using Java 6 or later, add <tt>-XX:+CMSClassUnloadingEnabled -XX:+UseConcMarkSweepGC<&#47;tt>. <tt>UseConcMarkSweepGC<&#47;tt> is needed to enable <tt>CMSClassUnloadingEnabled<&#47;tt>.<br />
<&#47;blockquote></p>
<p>Hey, that's me! And I'm using Java 6 or later too!<&#47;p></p>
<p>I'll let a couple graphs tell the tale.  Here's typical operation without the fix:<&#47;p></p>
<p><img src="http:&#47;&#47;gamache.org&#47;blog-images&#47;JVM-Heap-without-fix.png" alt="" title="JVM Heap without fix" width="616" height="206" class="alignnone size-full"><&#47;p></p>
<p>And with the fix?<&#47;p></p>
<p><img src="http:&#47;&#47;gamache.org&#47;blog-images&#47;JVM-Heap-with-fix.png" alt="" title="JVM Heap with fix" width="616" height="206" class="alignnone size-full"><&#47;p></p>
<p>Nice.  But not quite good enough.  It's still growing!  Just because I can wait 8 hours between server reboots instead of 4 doesn't make me that pleased.<&#47;p></p>
<p>I've condensed this tale into a short story, but this @#$!@$%!# leak was in the back of my mind for weeks.  I fought it on and off, but largely I just cron'ed up server restarts and stewed in my anger.  I tried this, I tried that.  Still the slow leak.<&#47;p></p>
<p>Eventually I got tired of worrying about it, and begrudgingly changed the code.  Now, instead of creating an anonymous subclass with extra ornaments, I just stick UUID-named ornaments onto the main class at runtime and pass the name along instead of the subclass.  Technically this is still a leak, but ornaments weigh a lot less than Christmas trees.<&#47;p></p>
<p><img src="http:&#47;&#47;gamache.org&#47;blog-images&#47;JVM-Heap-with-Avoidance.png" alt="" title="JVM Heap with Avoidance" width="616" height="206" class="alignnone size-full" &#47;><&#47;p></p>
<p>I can live with this.  For now.<&#47;p></p>
