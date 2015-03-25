---
layout: post
title: Ruby Method Cache - A Benchmark
date: '2013-10-13 10:44:33 -0400'
tags: code ruby
---

I write this on a Sunday morning from the
[Wicked Good Ruby](http://wickedgoodruby.com)
conference in Boston. I got here in time for coffee today. Good morning!

Yesterday’s first tech talk which really grabbed me was
[Charlie Somerville](https://twitter.com/charliesome) (GitHub) and his
“MRI Magic Tricks” presentation, during which he rides roughshod through
Ruby internals, rewrites class hierarchy, and demonstrates segmentation
re-violation through forcible hijacking of the crash dump process. He had
me laughing more than a tech talk usually would.

The other fantastic tech talk I caught was from
[Rachel Myers](https://twitter.com/rachelmyers) (GitHub) and
[Sheena McCoy](https://sheenapmccoy) (ModCloth), explaining the MRI Ruby
method cache in its various implementations over the years to today, and
showing the conditions which cause cache invalidations.

I have been working on my self-inflating hypermedia API client
[HyperResource](https://github.com/gamache/hyperresource)
a lot lately — watch this space, I am this close to releasing 0.2 —
and one of HyperResource’s main features is performing runtime method
definition when possible, with a fallback to `method_missing`-based
dispatch in other cases.

Well, defining methods at runtime busts the entire method resolution cache.
And the talk was doing a good job of convincing me that method cache
invalidation was worth avoiding. But compared to `method_missing`?
That’s been drilled into Rubyists’ heads as a gross inefficiency for years now.

I asked the two presenters whether they knew the relative performance hit
from method cache busting, as compared to `method_missing` dispatch. They
didn’t know offhand. But back to Charlie Somerville, whose illuminating
article listing all operations which invalidate the method cache which
Myers and McCoy reference made it easy to check.

I pecked out some code, available as a gist and shown here:

(Update 1: reworked to address Josh Jordan‘s rightful concern about
Class.new‘s overhead. Thanks Josh! and Update 2: added warm-up code for
JVM benchmarking.)

{% highlight ruby %}
require 'benchmark'

module A; end

class DefinedMethodStyle
  def bust_cache_class(*);  Class.new;            nil end
  def bust_cache_extend(*); Object.new.extend(A); nil end
  def dont_bust_cache(*);   Object.new;           nil end
end

class MethodMissingStyle
  def _bust_cache_class(*);  Class.new;            nil end
  def _bust_cache_extend(*); Object.new.extend(A); nil end
  def _dont_bust_cache(*);   Object.new;           nil end

  def method_missing(method, *args)
    case method
    when :bust_cache_class  then _bust_cache_class(*args)
    when :bust_cache_extend then _bust_cache_extend(*args)
    when :dont_bust_cache   then _dont_bust_cache(*args)
    end
  end
end

dms = DefinedMethodStyle.new
mms = MethodMissingStyle.new
n = 1_000_000

if ENV['WARM_UP'] # this helps with JRuby benchmarking
  puts "Warming up.\n"
  for i in 1..n
    dms.bust_cache_extend(i); dms.bust_cache_class(i); dms.dont_bust_cache(i)
    mms.bust_cache_extend(i); mms.bust_cache_class(i); mms.dont_bust_cache(i)
  end
end

Benchmark.bm do |bm|
  puts "\n defined methods, not busting cache:"
  bm.report { for i in 1..n; dms.dont_bust_cache(i)   end }

  puts "\n method_missing dispatch, not busting cache:"
  bm.report { for i in 1..n; mms.dont_bust_cache(i)   end }

  puts "\n defined methods, busting cache with trivial Object#extend:"
  bm.report { for i in 1..n; dms.bust_cache_extend(i) end }

  puts "\n defined methods, busting cache with Class.new:"
  bm.report { for i in 1..n; dms.bust_cache_class(i)  end }

  puts "\n method_missing dispatch, busting cache with trivial Object#extend:"
  bm.report { for i in 1..n; mms.bust_cache_extend(i) end }

  puts "\n method_missing dispatch, busting cache with Class.new:"
  bm.report { for i in 1..n; mms.bust_cache_class(i)  end }
end
{% endhighlight %}

So how bad is `method_missing`, really, in the grand scheme of things?

Not so bad, relatively speaking.

    $ ruby --version
    ruby 2.0.0p195 (2013-05-14 revision 40734) [x86_64-darwin12.3.0]

    $ ruby method-cache-test.rb
           user     system      total        real

     defined methods, not busting cache:
       0.270000   0.000000   0.270000 (  0.273828)

     method_missing dispatch, not busting cache:
       0.410000   0.000000   0.410000 (  0.408970)

     defined methods, busting cache with trivial Object#extend:
       1.390000   0.030000   1.420000 (  1.414536)

     defined methods, busting cache with Class.new:
       1.550000   0.090000   1.640000 (  1.638677)

     method_missing dispatch, busting cache with trivial Object#extend:
       1.610000   0.020000   1.630000 (  1.640575)

     method_missing dispatch, busting cache with Class.new:
       1.840000   0.060000   1.900000 (  1.889407)

When we’re not making MRI invalidate its method resolution cache,
`method_missing` comes in not too far behind regular method resolution —
it’s only about twice as time consuming, if your `method_missing`
is relatively lightweight. 100% penalty versus straight dispatch. Hmm.

But when we flush the method cache? Ouch! That’s a 600% penalty for normal
method resolution, and about 300% for `method_missing` dispatch.

The latest Ruby-HEAD, in the 2.1.0dev series, shows similar results, though
there seems to be greater penalty for a trivial Object#extend than for a
trivial Class.new:

    $ ruby --version
    ruby 2.1.0dev (2013-10-13 trunk 43273) [x86_64-darwin13.0.0]

    $ ruby method-cache-test.rb
           user     system      total        real

     defined methods, not busting cache:
       0.310000   0.010000   0.320000 (  0.311989)

     method_missing dispatch, not busting cache:
       0.400000   0.000000   0.400000 (  0.401800)

     defined methods, busting cache with trivial Object#extend:
       1.970000   0.010000   1.980000 (  1.983993)

     defined methods, busting cache with Class.new:
       1.860000   0.010000   1.870000 (  1.870652)

     method_missing dispatch, busting cache with trivial Object#extend:
       2.180000   0.000000   2.180000 (  2.187509)

     method_missing dispatch, busting cache with Class.new:
       2.010000   0.010000   2.020000 (  2.009935)

Update 2: And seeing as I use JRuby pretty often, I figured I’d test method
resolution cache-busting there too. I got some weird results until I figured
out that my code was probably being optimized on the fly at an unpredictable
point, so I added a “warm up” phase to the test. Behold: things are not much
better on the other side.

    $ ruby --version
    jruby 1.7.5 (1.9.3p392) 2013-10-07 74e9291 on Java HotSpot(TM) 64-Bit Server VM 1.7.0_13-b20 [darwin-x86_64]

    $ WARM_UP=1 ruby method-cache-test.rb
    Warming up.
           user     system      total        real

     defined methods, not busting cache:
       0.210000   0.010000   0.220000 (  0.119000)

     method_missing dispatch, not busting cache:
       0.380000   0.000000   0.380000 (  0.314000)

     defined methods, busting cache with trivial Object#extend:
       4.160000   0.140000   4.300000 (  3.372000)

     defined methods, busting cache with Class.new:
       8.730000   0.170000   8.900000 (  7.982000)

     method_missing dispatch, busting cache with trivial Object#extend:
       3.700000   0.170000   3.870000 (  2.952000)

     method_missing dispatch, busting cache with Class.new:
       6.080000   0.130000   6.210000 (  5.306000)

This is admittedly a small and very contrived example, and real-world loads
will have wildly different performance, but I think it’s fairly convincing
of the point that avoiding `method_missing` is not as important as one might
think, in environments which perform cache-busting operations reasonably often.

In relation to HyperResource, I am still satisfied with my choice to enable
both runtime method definition (which take place generally once per server
launch, per API data type), and `method_missing`-based syntactic sugar.
Runtime method definition increases performance at the cost of requiring
system restarts to fully recognize API-side updates, so I’ll be adding
a way to disable method creation too.

(And something brought up during Joshua Ballanco‘s thorough tour through
RubyMotion internals yesterday, is that RubyMotion can’t do `define_method`
at runtime. I fear I would have to refactor a bit of HyperResource code
itself in order to support RubyMotion, but it is an interesting thought.)

I was very excited to explore this topic in some depth yesterday.
Comments on my benchmark and conclusions are extremely welcome!

Aces and thanks to
[@RachelMyers](https://twitter.com/rachelmyers),
[@SheenaPMcCoy](https://twitter.com/sheenapmccoy),
[@charliesome](https://twitter.com/charliesome), and to the
[@WickedGoodRuby](https://twitter.com/wickedgoodruby) organizers.

[Now go read some links that Myers and McCoy put
together.](http://bit.ly/rubymethodcache)


