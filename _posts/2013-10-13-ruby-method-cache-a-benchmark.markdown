---
layout: post
status: publish
published: true
title: Ruby Method Cache - A Benchmark
author:
  display_name: gamache
  login: gamache
  email: pete@gamache.org
  url: ''
author_login: gamache
author_email: pete@gamache.org
wordpress_id: 127
wordpress_url: http://petegamache.com/?p=127
date: '2013-10-13 10:44:33 -0400'
date_gmt: '2013-10-13 14:44:33 -0400'
categories:
- Uncategorized
tags: []
comments: []
---
<p>
I write this on a Sunday morning from the <a href="http:&#47;&#47;wickedgoodruby.com" target="_blank">Wicked Good Ruby<&#47;a> conference in Boston.  I got here in time for coffee today. Good morning!<br />
<&#47;p></p>
<p>Yesterday's first tech talk which really grabbed me was <a href="https:&#47;&#47;twitter.com&#47;charliesome" target="_blank">Charlie Somerville<&#47;a> (GitHub) and his "MRI Magic Tricks" presentation, during which he rides roughshod through Ruby internals, rewrites class hierarchy, and demonstrates segmentation re-violation through forcible hijacking of the crash dump process. He had me laughing more than a tech talk usually would.<br />
<&#47;p></p>
<p>
The other fantastic tech talk I caught was from <a href="https:&#47;&#47;twitter.com&#47;rachelmyers" target="_blank">Rachel Myers<&#47;a> (GitHub) and <a href="https:&#47;&#47;twitter.com&#47;sheenapmccoy" target="_blank">Sheena McCoy<&#47;a> (ModCloth), explaining the MRI Ruby method cache in its various implementations over the years to today, and showing the conditions which cause cache invalidations.<br />
<&#47;p></p>
<p>
I have been working on my self-inflating hypermedia API client <a href="https:&#47;&#47;github.com&#47;gamache&#47;hyperresource" target="_blank">HyperResource<&#47;a> a lot lately &acirc;&euro;&rdquo; watch this space, I am <em>this<&#47;em> close to releasing 0.2 &acirc;&euro;&rdquo; and one of HyperResource's main features is performing runtime method definition when possible, with a fallback to <tt>method_missing<&#47;tt>-based dispatch in other cases.<br />
<&#47;p></p>
<p>
Well, defining methods at runtime busts the entire method resolution cache.<br />
And the talk was doing a good job of convincing me that method cache invalidation was worth avoiding. But compared to <tt>method_missing<&#47;tt>? That's been drilled into Rubyists' heads as a gross inefficiency for years now.<br />
<&#47;p></p>
<p>
I asked the two presenters whether they knew the relative performance hit from method cache busting, as compared to <tt>method_missing<&#47;tt> dispatch. They didn't know offhand. But back to Charlie Somerville, whose illuminating <a href="https:&#47;&#47;charlie.bz&#47;blog&#47;things-that-clear-rubys-method-cache" target="_blank">article listing all operations which invalidate the method cache<&#47;a> which Myers and McCoy reference made it easy to check.<br />
<&#47;p></p>
<p> I pecked out some code, available as a <a href="https:&#47;&#47;gist.github.com&#47;gamache&#47;6962565" target="_blank">gist<&#47;a> and shown here:<br />
(Update 1: reworked to address <a href="https:&#47;&#47;twitter.com&#47;joshjordanwhat" target=_"blank">Josh Jordan<&#47;a>'s rightful concern about <tt>Class.new<&#47;tt>'s overhead. Thanks Josh!  and Update 2: added warm-up code for JVM benchmarking.)<br />
<&#47;p></p>
<pre lang="ruby">
require 'benchmark'</p>
<p>module A; end</p>
<p>class DefinedMethodStyle<br />
  def bust_cache_class(*);  Class.new;            nil end<br />
  def bust_cache_extend(*); Object.new.extend(A); nil end<br />
  def dont_bust_cache(*);   Object.new;           nil end<br />
end</p>
<p>class MethodMissingStyle<br />
  def _bust_cache_class(*);  Class.new;            nil end<br />
  def _bust_cache_extend(*); Object.new.extend(A); nil end<br />
  def _dont_bust_cache(*);   Object.new;           nil end</p>
<p>  def method_missing(method, *args)<br />
    case method<br />
    when :bust_cache_class  then _bust_cache_class(*args)<br />
    when :bust_cache_extend then _bust_cache_extend(*args)<br />
    when :dont_bust_cache   then _dont_bust_cache(*args)<br />
    end<br />
  end<br />
end</p>
<p>dms = DefinedMethodStyle.new<br />
mms = MethodMissingStyle.new<br />
n = 1_000_000</p>
<p>if ENV['WARM_UP'] # this helps with JRuby benchmarking<br />
  puts "Warming up.\n"<br />
  for i in 1..n<br />
    dms.bust_cache_extend(i); dms.bust_cache_class(i); dms.dont_bust_cache(i)<br />
    mms.bust_cache_extend(i); mms.bust_cache_class(i); mms.dont_bust_cache(i)<br />
  end<br />
end</p>
<p>Benchmark.bm do |bm|<br />
  puts "\n defined methods, not busting cache:"<br />
  bm.report { for i in 1..n; dms.dont_bust_cache(i)   end }</p>
<p>  puts "\n method_missing dispatch, not busting cache:"<br />
  bm.report { for i in 1..n; mms.dont_bust_cache(i)   end }</p>
<p>  puts "\n defined methods, busting cache with trivial Object#extend:"<br />
  bm.report { for i in 1..n; dms.bust_cache_extend(i) end }</p>
<p>  puts "\n defined methods, busting cache with Class.new:"<br />
  bm.report { for i in 1..n; dms.bust_cache_class(i)  end }</p>
<p>  puts "\n method_missing dispatch, busting cache with trivial Object#extend:"<br />
  bm.report { for i in 1..n; mms.bust_cache_extend(i) end }</p>
<p>  puts "\n method_missing dispatch, busting cache with Class.new:"<br />
  bm.report { for i in 1..n; mms.bust_cache_class(i)  end }<br />
end<br />
<&#47;pre></p>
<p>
So how bad is <tt>method_missing<&#47;tt>, really, in the grand scheme of things?<br />
<&#47;p></p>
<p>
Not so bad, relatively speaking.<br />
<&#47;p></p>
<pre>
$ ruby --version<br />
ruby 2.0.0p195 (2013-05-14 revision 40734) [x86_64-darwin12.3.0]</p>
<p>$ ruby method-cache-test.rb<br />
       user     system      total        real</p>
<p> defined methods, not busting cache:<br />
   0.270000   0.000000   0.270000 (  0.273828)</p>
<p> method_missing dispatch, not busting cache:<br />
   0.410000   0.000000   0.410000 (  0.408970)</p>
<p> defined methods, busting cache with trivial Object#extend:<br />
   1.390000   0.030000   1.420000 (  1.414536)</p>
<p> defined methods, busting cache with Class.new:<br />
   1.550000   0.090000   1.640000 (  1.638677)</p>
<p> method_missing dispatch, busting cache with trivial Object#extend:<br />
   1.610000   0.020000   1.630000 (  1.640575)</p>
<p> method_missing dispatch, busting cache with Class.new:<br />
   1.840000   0.060000   1.900000 (  1.889407)<br />
<&#47;pre></p>
<p>
When we're not making MRI invalidate its method resolution cache, <tt>method_missing<&#47;tt> comes in not too far behind regular method resolution &mdash;<br />
it's only about twice as time consuming, if your <tt>method_missing<&#47;tt> is relatively lightweight. 100% penalty versus straight dispatch.  Hmm.<br />
<&#47;p></p>
<p>
But when we flush the method cache? Ouch! That's a 600% penalty for normal method resolution, and about 300% for <tt>method_missing<&#47;tt> dispatch.<br />
<&#47;p></p>
<p>
The latest Ruby-HEAD, in the 2.1.0dev series, shows similar results, though there seems to be greater penalty for a trivial <tt>Object#extend<&#47;tt> than for a trivial <tt>Class.new<&#47;tt>:<br />
<&#47;p></p>
<pre>
$ ruby --version<br />
ruby 2.1.0dev (2013-10-13 trunk 43273) [x86_64-darwin13.0.0]</p>
<p>$ ruby method-cache-test.rb<br />
       user     system      total        real</p>
<p> defined methods, not busting cache:<br />
   0.310000   0.010000   0.320000 (  0.311989)</p>
<p> method_missing dispatch, not busting cache:<br />
   0.400000   0.000000   0.400000 (  0.401800)</p>
<p> defined methods, busting cache with trivial Object#extend:<br />
   1.970000   0.010000   1.980000 (  1.983993)</p>
<p> defined methods, busting cache with Class.new:<br />
   1.860000   0.010000   1.870000 (  1.870652)</p>
<p> method_missing dispatch, busting cache with trivial Object#extend:<br />
   2.180000   0.000000   2.180000 (  2.187509)</p>
<p> method_missing dispatch, busting cache with Class.new:<br />
   2.010000   0.010000   2.020000 (  2.009935)<br />
<&#47;pre></p>
<p><b>Update 2:<&#47;b> And seeing as I use JRuby pretty often, I figured I'd test method resolution cache-busting there too.  I got some weird results until I figured out that my code was probably being optimized on the fly at an unpredictable point, so I added a "warm up" phase to the test.  Behold: things are not much better on the other side.<&#47;p></p>
<pre>
$ ruby --version<br />
jruby 1.7.5 (1.9.3p392) 2013-10-07 74e9291 on Java HotSpot(TM) 64-Bit Server VM 1.7.0_13-b20 [darwin-x86_64]</p>
<p>$ WARM_UP=1 ruby method-cache-test.rb<br />
Warming up.<br />
       user     system      total        real</p>
<p> defined methods, not busting cache:<br />
   0.210000   0.010000   0.220000 (  0.119000)</p>
<p> method_missing dispatch, not busting cache:<br />
   0.380000   0.000000   0.380000 (  0.314000)</p>
<p> defined methods, busting cache with trivial Object#extend:<br />
   4.160000   0.140000   4.300000 (  3.372000)</p>
<p> defined methods, busting cache with Class.new:<br />
   8.730000   0.170000   8.900000 (  7.982000)</p>
<p> method_missing dispatch, busting cache with trivial Object#extend:<br />
   3.700000   0.170000   3.870000 (  2.952000)</p>
<p> method_missing dispatch, busting cache with Class.new:<br />
   6.080000   0.130000   6.210000 (  5.306000)<br />
<&#47;pre></p>
<p>
This is admittedly a small and very contrived example, and real-world loads will have wildly different performance, but I think it's fairly convincing of the point that avoiding <tt>method_missing<&#47;tt> is not as important as one might think, in environments which perform cache-busting operations reasonably often.<br />
<&#47;p></p>
<p>
In relation to HyperResource, I am still satisfied with my choice to enable both runtime method definition (which take place generally once per server launch, per API data type), and <tt>method_missing<&#47;tt>-based syntactic sugar.  Runtime method definition increases performance at the cost of requiring system restarts to fully recognize API-side updates, so I'll be adding a way to disable method creation too.<br />
<&#47;p></p>
<p>
(And something brought up during <a href="https:&#47;&#47;twitter.com&#47;manhattanmetric" target="_blank">Joshua Ballanco<&#47;a>'s thorough tour through RubyMotion internals yesterday, is that RubyMotion can't do <tt>define_method<&#47;tt> at runtime.  I fear I would have to refactor a bit of HyperResource code itself in order to support RubyMotion, but it is an interesting thought.)<br />
<&#47;p></p>
<p>
I was very excited to explore this topic in some depth yesterday.  Comments on my benchmark and conclusions are extremely welcome!<br />
<&#47;p></p>
<p>
Aces and thanks to <a target="_blank" href="https:&#47;&#47;twitter.com&#47;rachelmyers">@RachelMyers<&#47;a>, <a target="_blank" href="https:&#47;&#47;twitter.com&#47;sheenapmccoy">@SheenaPMcCoy<&#47;a>, <a target="_blank" href="https:&#47;&#47;twitter.com&#47;charliesome">@charliesome<&#47;a>, and to the <a target="_blank" href="https:&#47;&#47;twitter.com&#47;wickedgoodruby">@WickedGoodRuby<&#47;a> organizers.<br />
<&#47;p></p>
<p>
<a href="http:&#47;&#47;bit.ly&#47;rubymethodcache" target="_blank">Now go read some links that Myers and McCoy put together.<&#47;a><br />
<&#47;p></p>
