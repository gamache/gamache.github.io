---
layout: post
title: "Elixir Best Practices: When to Use Structs, String-keyed Maps, and Atom-keyed Maps"
date: 2016-02-02 09:11:00 -0500
author: Pete Gamache
excerpt_separator: <!--more-->
tags: elixir
---

Elixir, much like its foundation language Erlang and its syntactic muse
Ruby, makes a distinction between _character strings_, like `"hi_mom"`,
and _atoms_, like `:hi_mom`.

Elixir also offers several ways to pass around key-value data.  There's
[Map](http://elixir-lang.org/docs/v1.1/elixir/Map.html){:target="_blank"}, whose keys can be
either keys or atoms; [Keyword](http://elixir-lang.org/docs/v1.1/elixir/Keyword.html){:target="_blank"}
lists, which are [association lists](https://en.wikipedia.org/wiki/Association_list){:target="_blank"}
(or *alists*) consisting of `{atom, any_value}` tuples; and there are
association lists of `{string, any_value}` tuples, such as the HTTP headers
returned by [Plug.Conn.req_headers](https://hexdocs.pm/plug/Plug.Conn.html){:target="_blank"}.

Elixir [structs](http://elixir-lang.org/getting-started/structs.html){:target="_blank"}
are represented as Maps with atom keys, additionally restricting the user to
a predefined set of keys, and allowing a shorter syntax for field access than
non-struct Maps.

Maps (and structs) and keyword lists implement the Elixir
[Dict](http://elixir-lang.org/docs/v1.1/elixir/Dict.html){:target="_blank"}
behaviour, whereas the string-keyed alist does not.

Given that these different data structures are not compatible, the programmer
must choose which to use at any given place in an application's code.
But how?  This is the source of frustration and confusion in noobs and
greybeards alike.

In this post, we explore the right times to use each.

<!--more-->


## About Atoms and Strings

Atoms (called "symbols" in Ruby and Lisp, among others) are much
like integers; they are [constants where their name is their own
value](http://elixir-lang.org/getting-started/basic-types.html#atoms){:target="_blank"}.
Importantly, <b>atoms are not garbage-collected</b>, so allocating them at
runtime can lead to RAM exhaustion over time.

Atoms are also the "preferred" keys for
maps and dicts; this is to say, the Elixir language provides syntactic
sugar to encourage their use.  Writing `%{a: 1}` is shorter than the
equivalent `%{:a => 1}` and shorter still than the string-keyed version
`%{"a" => 1}`.

Strings are the same character strings you find in most languages: the
same string can be allocated multiple times, and strings are GCed alongside
other data.  There is no penalty to using strings as map or dict keys,
but there is a slight friction to doing so, so many programmers tend to
use symbols as map keys as a default instead.


## Rule #1: Always Use String-Keyed Maps for External Data

Data which comes from outside your application, broadly speaking, can't
be trusted.  Given that atom allocation can lead to memory
exhaustion in a long-running Erlang system, using atoms for external
data opens up your app for a potential denial of service (DoS) attack.

This fact is reflected in many Elixir libraries, for example the popular
JSON parser [Poison](https://github.com/devinus/poison){:target="_blank"}.
Well-behaved libraries will generally use strings as map keys when converting
external data into an internal data structure.  We recommend the
same.


## Rule #2: Convert External Data to Structs ASAP

Elixir structs have some attractive properties.  They enforce a (weak)
schema on your application data, so that code may rely on the structure
that the model struct provides.  Like other atom-keyed maps, they
allow access to the
member values using dot notation; e.g., `some_struct.foo` rather than
the equivalent `some_struct[:foo]`.

Structs enforce that you aren't cramming in non-schematic data,
discouraging you from reverting to the "freeform bags of data" mindset.
And they are easy to specify in pattern matches in function definitions or
case statements, to add a bit of runtime type safety to code.

The basic approach to this procedure is to define a model module with a
struct inside it, then write a function to create and populate a struct
with a map of externally-sourced data.  This is a better idea than
scattering code to populate the structs around your code; it's more DRY
and allows you to make changes in one place and test across the board.

Appcues has released a Hex project called
[ExConstructor](https://github.com/appcues/exconstructor){:target="_blank"}
to aid in this process.  ExConstructor automatically builds constructor
functions for structs, handling map-vs-dict, string-vs-atom, and even
camelCase-vs-under_score key format automatically.
Using ExConstructor means that your code may never have to use string-keyed
maps directly.


## Rule #3: Use Structs in All Other Code

If code in your app uses data which originated from the outside world,
and it's not the one piece of code dedicated to converting that type of
data into a struct, it should be explicitly written against the model
structs.

Write function definitions which make it clear that struct input is
required:

{% highlight elixir %}
def some_func(input) do ...               # no
def some_func(%{}=input) do ...           # no
def some_func(%MyStruct{}=input) do ...   # yes
{% endhighlight %}

Structs provide guarantees. Guarantees strengthen your code by removing
the need for pervasive error/anomaly checking. This leads to better code
in less time.

Note: Phoenix/Ecto models are simply structs with some nice features around
changesets. Using them across your app is encouraged, though it's wise to
decouple the database access from the rest of your app, perhaps using a
data access object, if there is any chance you may move to a different
database backend.


## Rule #4: Use Structs for Output Data

Using structs for output data is a good idea for the same reasons that
using it internally is a good idea: guarantees, DRY, and, well,
structure.

Structs convert to JSON maps quite easily, through either
`Map.from_struct/1`, or custom solutions like `@derive
[Poison.Encoder]`.


## Rule #5: Avoid Using Atom-keyed Maps That Aren't Structs

Using a plain map for passing data around your app may be convenient to
write the first time, but over the lifecycle of an app, it leads to
decreased expressivity and more confusion about the exact shape of the
data at any given point.  Creating structs to represent state, even
state completely internal to a module, reduces this confusion by
plainly advertising (and enforcing) the structure of the underlying data.

There will be times to use plain maps, of course, but if your code has
more bare maps than model structs, this may be an indication that
your app's state objects should be better-defined.

When you do use them, don't fight the language; use atoms as keys.


## Rule #6: Use Keyword Lists Only for Function Arguments

Keyword lists are the idiomatic Elixir way to provide
keyword arguments to a function.  Since only the programmer should be
writing these arguments --
[sanitize your inputs!](https://xkcd.com/327/){:target=>"_blank"}
-- atom-related DOS is not a problem.


## Summary

The choice between maps, structs, and keyword lists,
or between string keys and atom keys,
is not always clear, and improper decisions can lead to headaches.

This article promotes simple guidelines to keep the Elixir programmer
out of trouble:

* Use string-keyed maps for data which has just arrived from an external source.
  * Convert external data to structs as soon as possible.
  * Use [ExConstructor](https://github.com/appcues/exconstructor){:target="_blank"}
    or something similar to abstract away the handling of string-keyed maps.
* Use structs almost everywhere else in your program.
* Use other atom-keyed maps sparingly.
* Use keyword lists only to provide arguments to a function.

These rules of thumb have improved the quality of our code here at Appcues,
and we hope they can help you out too.

<br>

<i>Thanks to [Ian Asaff](https://github.com/montague)
for reading drafts of this post.</i>

