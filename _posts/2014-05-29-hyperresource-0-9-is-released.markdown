---
layout: post
status: publish
published: true
title: HyperResource 0.9 Is Released
author:
  display_name: gamache
  login: gamache
  email: pete@gamache.org
  url: ''
author_login: gamache
author_email: pete@gamache.org
wordpress_id: 175
wordpress_url: http://petegamache.com/?p=175
date: '2014-05-29 11:50:33 -0400'
date_gmt: '2014-05-29 15:50:33 -0400'
categories:
- Code
tags: []
comments: []
---
<p>Today I am pleased to announce the release of HyperResource 0.9!</p>
<p><a href="https:&#47;&#47;github.com&#47;gamache&#47;hyperresource" target="_blank">HyperResource<&#47;a> is a Ruby client for hypermedia-driven web APIs.  It's designed to make working with hypermedia web services as simple as working with local data, while still remaining a 100% generic client.</p>
<p>The release of 0.9 represents a significant jump in quality and completeness from 0.2.x.  The interface to HyperResource is largely solidified, and the internal and user-facing code has been significantly improved since earlier versions.</p>
<p>Here are the most important changes from 0.2.x:</p>
<ul>
<li><b>Introducing hostmasked config variables.<&#47;b><br&#47;>HyperResource 0.9 adds the feature that configuration parameters (<tt>namespace<&#47;tt>, <tt>auth<&#47;tt>, <tt>header<&#47;tt>, <tt>faraday_options<&#47;tt>, and <tt>default_attributes<&#47;tt>) can now be scoped by hostmask. This opens the way for seamless integration with an ecosystem of separate APIs which point to each others' resources (well... seamless for the client, anyway). Look at <a href="https:&#47;&#47;github.com&#47;gamache&#47;hyperresource#configuration-for-multiple-hosts" target="_blank">sample code from the HyperResource README<&#47;a> to see an example.
<p>(As a bonus, I spun the URL mask matching code into its own gem:<br />
<a href="https:&#47;&#47;github.com&#47;gamache&#47;fuzzyurl" target="_blank">FuzzyURL, a library for non-strict manipulation and fuzzy matching of URLs<&#47;a>.  Give it a try if using <tt>URI<&#47;tt> hurts.)</p>
<li><b>Automatic method creation doesn't happen anymore.<&#47;b><br><tt>method_missing<&#47;tt> now handles all method calls for link, object, and attribute names. Creating instance methods on the resource's class was error-prone, it could violate the principle of HATEOAS by misleading the end user about links or attributes or objects that don't exist on a particular resource, and most egregiously of all, it completely busts Ruby's method resolution cache, causing absolutely all method calls to incur a ~1ms penalty the first time they're called after the cache is cleared. method_missing has always done a fine job; removing <tt>_hr_create_methods!<&#47;tt> improves real world performance and simplifies the HR code.
<li><b>Resource instantiation is cleaner.<&#47;b><br> Previously, calling an HTTP method on a HyperResource would change its state, then return a different object. Now HTTP is delegated to Link objects, and <tt>HyperResource#to_link<&#47;tt> handles instantiating a link from a resource (in the background, of course). The resulting logic is cleaner, inheritance is easier to track, and object state is easier to comprehend.
<li><b><tt>#create<&#47;tt> and <tt>#update<&#47;tt> are deprecated.<&#47;b><br> Just use <tt>#post<&#47;tt>, <tt>#put<&#47;tt>, or <tt>#patch<&#47;tt>.<br />
<&#47;ul></p>
<p>This release of HyperResource is cleaner, more flexible, and more performant than ever.  Here's hoping it makes life a little easier for API consumers and designers.</p>
