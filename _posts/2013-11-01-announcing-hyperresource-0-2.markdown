---
layout: post
status: publish
published: true
title: Announcing HyperResource 0.2
author:
  display_name: gamache
  login: gamache
  email: pete@gamache.org
  url: ''
author_login: gamache
author_email: pete@gamache.org
wordpress_id: 159
wordpress_url: http://petegamache.com/?p=159
date: '2013-11-01 11:45:40 -0400'
date_gmt: '2013-11-01 15:45:40 -0400'
categories:
- Uncategorized
tags: []
comments: []
---
<p>I am pleased to announce the second point release of <a href="https:&#47;&#47;github.com&#47;gamache&#47;hyperresource" target="_blank">HyperResource<&#47;a>, a Ruby client library for hypermedia APIs.<&#47;p></p>
<p>The goal of HyperResource is to make hypermedia the most attractive design for API designers and consumers.  HyperResource provides a concise, elegant, simple interface to link-driven APIs which rivals that of most handwritten client libraries.  It also allows namespacing and extending of API data types, making it not only great for simple customizations, but ideal as a platform for a rich client library as well.<&#47;p></p>
<p>A lot has changed since the initial 0.1 release in April.  A partial list:<&#47;p></p>
<ul>
<li>Support for GET, POST, PUT, PATCH, and DELETE<&#47;li>
<li>Cacheability of HyperResource objects using Marshal<&#47;li>
<li>HyperResource::Adapter interface, allowing easy integration with other hypermedia formats<&#47;li>
<li>Better HyperResource::Exception objects<&#47;li>
<li>incoming_body_filter and outgoing_body_filter methods, allowing arbitrary filter&#47;transform to incoming or outgoing data<&#47;li>
<li>Ruby compatibility from 1.8.7&#47;REE to present HEAD<&#47;li>
<li>JRuby compatibility from 1.8-mode to present HEAD<&#47;li>
<li>YARD documentation at <a href="http:&#47;&#47;hyperresource.com&#47;doc" target="_blank">hyperresource.com&#47;doc<&#47;a><&#47;li>
<li>Behind the scenes: live tests against a reference Sinatra server, major refactoring, naming improvements<&#47;li><br />
<&#47;ul></p>
<p>All in all, HyperResource got a lot nicer over the past two months.  It's stable, it's fast, and it makes hypermedia-driven design a very compelling option for APIs today.  I consider this 0.2 release a candidate for a 1.0 early next year.<&#47;p></p>
<p>If you have a hyperlink-driven API to connect to, or you're developing an API and want your client library to write itself, give HyperResource a look.<&#47;p></p>
<p>
<a href="http:&#47;&#47;hyperresource.com" target="_blank">hyperresource.com<&#47;a><br />
<a href="https:&#47;&#47;github.com&#47;gamache&#47;hyperresource" target="_blank">github.com&#47;gamache&#47;hyperresource<&#47;a><br />
<&#47;p></p>
