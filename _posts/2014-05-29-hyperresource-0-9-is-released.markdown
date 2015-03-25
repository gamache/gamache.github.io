---
layout: post
title: HyperResource 0.9 Is Released
date: '2014-05-29 11:50:33 -0400'
tags: hyperresource ruby hypermedia api code ruby
---
Today I am pleased to announce the release of HyperResource 0.9!

HyperResource is a Ruby client for hypermedia-driven web APIs. It’s designed
to make working with hypermedia web services as simple as working with local
data, while still remaining a 100% generic client.

The release of 0.9 represents a significant jump in quality and completeness
from 0.2.x. The interface to HyperResource is largely solidified, and the
internal and user-facing code has been significantly improved since earlier
versions.

Here are the most important changes from 0.2.x:

* Introducing hostmasked config variables.
HyperResource 0.9 adds the feature that configuration parameters
(`namespace`, `auth`, `header`, `faraday_options`, and `default_attributes`)
can now be scoped by hostmask. This opens the way for seamless integration
with an ecosystem of separate APIs which point to each others’ resources
(well… seamless for the client, anyway).
Look at [sample code from the HyperResource
README](https://github.com/gamache/hyperresource#configuration-for-multiple-hosts)
to see an example.
(As a bonus, I spun the URL mask matching code into its own gem:
[FuzzyURL, a library for non-strict manipulation and fuzzy matching of
URLs](https://github.com/gamache/fuzzyurl).
Give it a try if using URI hurts.)

* Automatic method creation doesn’t happen anymore.
`method_missing` now handles all method calls for link, object, and attribute
names. Creating instance methods on the resource’s class was error-prone,
it could violate the principle of HATEOAS by misleading the end user about
links or attributes or objects that don’t exist on a particular resource,
and most egregiously of all, it completely busts Ruby’s method resolution
cache, causing absolutely all method calls to incur a ~1ms penalty the
first time they’re called after the cache is cleared. `method_missing`
has always done a fine job; removing `_hr_create_methods!` improves real
world performance and simplifies the HR code.

* Resource instantiation is cleaner.
Previously, calling an HTTP method on a HyperResource would change its
state, then return a different object. Now HTTP is delegated to Link
objects, and `HyperResource#to_link` handles instantiating a link from
a resource (in the background, of course). The resulting logic is
cleaner, inheritance is easier to track, and object state is easier
to comprehend.

* `#create` and `#update` are deprecated.
Just use `#post`, `#put`, or `#patch`.

This release of HyperResource is cleaner, more flexible, and more performant
than ever. Here’s hoping it makes life a little easier for API consumers
and designers.


