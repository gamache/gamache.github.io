---
layout: post
title: Announcing HyperResource 0.2
date: '2013-11-01 11:45:40 -0400'
tags: api hyperresource code ruby
---
I am pleased to announce the second point release of
[HyperResource](https://github.com/gamache/hyperresource),
a Ruby client library for hypermedia APIs.

The goal of HyperResource is to make hypermedia the most attractive design
for API designers and consumers. HyperResource provides a concise, elegant,
simple interface to link-driven APIs which rivals that of most handwritten
client libraries. It also allows namespacing and extending of API data types,
making it not only great for simple customizations, but ideal as a platform
for a rich client library as well.

A lot has changed since the initial 0.1 release in April. A partial list:

* Support for GET, POST, PUT, PATCH, and DELETE
* Cacheability of HyperResource objects using Marshal
* `HyperResource::Adapter` interface, allowing easy integration with other
  hypermedia formats
* Better `HyperResource::Exception` objects
* `incoming_body_filter` and `outgoing_body_filter` methods, allowing arbitrary 
  filter/transform to incoming or outgoing data
* Ruby compatibility from 1.8.7/REE to present HEAD
* JRuby compatibility from 1.8-mode to present HEAD
* YARD documentation at hyperresource.com/doc
* Behind the scenes: live tests against a reference Sinatra server,
  major refactoring, naming improvements

All in all, HyperResource got a lot nicer over the past two months.
It’s stable, it’s fast, and it makes hypermedia-driven design a very
compelling option for APIs today. I consider this 0.2 release a candidate
for a 1.0 early next year.

If you have a hyperlink-driven API to connect to, or you’re developing an API
and want your client library to write itself, give HyperResource a look.

[hyperresource.com](http://hyperresource.com) <br>
[github.com/gamache/hyperresource](https://github.com/gamache/hyperresource)

