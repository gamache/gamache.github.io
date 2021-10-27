---
layout: post
title: "ExConstructor: A Better Struct Initializer for Elixir"
date: 2016-04-06 10:22:00 -0400
author: Pete Gamache
excerpt_separator: <!--more-->
tags: elixir
---

<a href="//engineering.appcues.com/2016/02/02/too-many-dicts.html" target="_blank">Our last blog post</a>
talked about
<a href="https://github.com/appcues/exconstructor" target="_blank">ExConstructor</a>,
a tool we wrote at Appcues to make it
incredibly easy to turn data from the outside world into structs.
We're pleased to announce
<a href="https://github.com/appcues/exconstructor" target="_blank">ExConstructor</a>
version 1.0 to the Elixir community!

<!--more-->

Elixir structs are a great way to represent data in your apps, but there
is often an impedance mismatch between structs and the data used to
initialize them.  Often, a programmer must compensate for key data type
(string or atom), key formatting (under_score or camelCase), and
container data structure (map, keyword list, list of tuples).

<a href="https://github.com/appcues/exconstructor" target="_blank">ExConstructor</a>
gets all this
out of your way.  Add `use ExConstructor` after a `defstruct` expression
to generate a constructor function that handles atoms vs strings,
under_score vs camelCase, and map vs list issues automatically.


## Why ExConstructor?

As discussed in
<a href="//engineering.appcues.com/2016/02/02/too-many-dicts.html" target="_blank">our last post</a>,
it's good practice to write functions that use structs instead of generic
maps or keyword lists, because your functions
become clearer in their intent (what kind of data do they deal with?
what's it look like?) and usage (precisely what arguments am I allowed
to give?).

Elixir doesn't make it easy to create structs from data which doesn't
already perfectly match the struct, but `ExConstructor` does.  Here is
a small example:

{% highlight elixir %}
defmodule TestStruct do
  defstruct field_one: nil,
            field_two: nil,
            field_three: nil,
            field_four: nil
  use ExConstructor
end

# Unruly data. There's atoms, there's strings, there's camels, there's snakes!
input_data = %{"field_one" => "a", "fieldTwo" => "b", :field_three => "c", :fieldFour => "d"}

TestStruct.new(input_data)
# => %TestStruct{field_one: "a", field_two: "b", field_three: "c", field_four: "d"}
# Oh my phear ***IT JUST WORKS***
{% endhighlight %}

## Why Not Just Use Native Elixir?

It sounds almost too simple. The task of creating structs may seem to be
so trivial that the gains from doing so are barely worth adding a dependency
to your app.

This is almost true.
<a href="https://github.com/appcues/exconstructor/blob/master/lib/exconstructor.ex" target="_blank">There's only about 100 lines of code</a>
in ExConstructor.

So, really, what can ExConstructor do that e.g.,
`struct(TestStruct, your_input_here)` can't?  A table will hammer it home.

| Data Structure || Key Type || Key Format || Native || ExConstructor |
| --- |---| --- |---| --- | --- | :---: | :---: | :---: | :---: |
| Map            || Atom   || under_score || ✔ || ✔ |
| Map            || Atom   || camelCase   ||   || ✔ |
| Map            || String || under_score ||   || ✔ |
| Map            || String || camelCase   ||   || ✔ |
| Keyword list   || Atom   || under_score || ✔ || ✔ |
| Keyword list   || Atom   || camelCase   ||   || ✔ |
| 2-Tuple list   || String || under_score ||   || ✔ |
| 2-Tuple list   || String || camelCase   ||   || ✔ |

<br>
If you handle any of the data formats without a checkbox in the Native
column, you should look at ExConstructor.  It's not that you couldn't do it
yourself -- it's that you are going to do it yourself hundreds of times.


## Try It

<a href="http://hexdocs.pm/exconstructor/ExConstructor.html" target="_blank">Full documentation for ExConstructor</a>
is available on <a href="http://hexdocs.pm" target="_blank">Hexdocs.pm</a>,
including installation and configuration instructions.  Give it a try,
DRY up your code, and spend more time on interesting work!

