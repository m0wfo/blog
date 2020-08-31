---
id: 843
title: "Rails thread safety ain't a thing"
date: 2019-01-14T19:37:22+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=843
permalink: /2019/01/14/rails-thread-safety-aint-a-thing/
categories:
  - Uncategorized
---
So because of reasons I&#8217;ve started writing Ruby again. That mostly makes me happy. Developers prostrate themselves at the foot of the gods of UX and customers&#8217; acceptance criteria but often neglect the whole developer experience and how palatable their own tooling is. Despite some frankly pretty shaky underpinnings, there&#8217;s a lot to like in the Ruby ecosystem if you&#8217;re just trying to get shit done.

Anyway, just pinning a note mainly for my own benefit* about the liberal use of the **_||=_** conditional assignment operator in Rails (or Rack-based apps in general). Go ahead and run this:

<pre>50.times do |i|
  Thread.new { $x ||= i }
end

sleep 1

puts $x</pre>

If you had neither visibility nor atomicity concerns, you&#8217;d always expect 0, right? Go ahead and run it a few more times. If you ran it with JRuby then you probably got a nonzero result straight away, anyway. Ok, so there&#8217;s a bit of bad news:

  1. ||= does not magically compile down to a <a href="https://www.felixcloutier.com/x86/cmpxchg" target="_blank">CMPXCHG</a> instruction or somesuch magic**. It&#8217;s just sugar for _&#8216;if operand is null assign to i else return current value&#8217;. _Therefore multiple concurrently executing instructions can be interleaved and potentially reference the same address before we&#8217;re done
  2.  There&#8217;s no well-defined model describing atomicity of variable reassignment or value visibility in Ruby; it&#8217;s all architecture, interpreter and interpreter-version dependent
  3. The Rack spec gives no indication whether _app.call()_ could be invoked concurrently or not

Hrmph. So are we screwed? In the case of something like a Rails app where you&#8217;re knowingly concurrently mutating something, it&#8217;s good to wrap it in a <a href="https://ruby-doc.org/core-2.5.0/Mutex.html" target="_blank">Mutex</a>; for the above example, that&#8217;d boil down to something like this:

<pre>lock = Mutex.new

50.times do |i|
  Thread.new { lock.synchronize { $x ||= i } }
end

sleep 1

puts $x</pre>

Of course, this sort of programming is horrendously deadlock/livelock prone just as in any other imperative language, regardless of type system, not to mention the fact that you just killed all the parallelism in this case; Ruby&#8217;s out-of-the-box concurrency tools are pretty coarse-grained, unfortunately.

## You haven&#8217;t answered the question

Yeah, I was trying to waffle and do some veiled trolling at the same time. No, you&#8217;re not screwed. Follow a few rules of thumb and you&#8217;ll be fine:

  * Rails controllers & models seem to be reused on a per-thread basis, so if you&#8217;re sharing, e.g. class variables, you&#8217;ll at worst end up with one instance per thread (the number of which is normally capped by the server which preemptively creates them)
  * Anything at the Actionpack level or lower can be expected to be concurrently called and regenerated per-request

Rails was declared &#8216;thread-safe&#8217; around 2.2.x IIRC, but that was achieved by sticking a giant mutex around the app.call() entry point, which is kind of intellectually running away from the problem. Since then the global lock was peeled back but documentation on changes to the threading model changes in the interim seems a bit thin on the ground.

I don&#8217;t want to overtly troll too much, because the challenge of concurrent programming is definitely nontrivial; the original mutex-based cop-out is not an admonishment to the Rails framework- it&#8217;s one of those things where solid foundations matter (i.e. not using C-Ruby), and one reason JRuby continues to be relevant. It took Sun, subsequently Oracle plus many other concurrency luminaries on their payroll billions of dollars and man-hours to come up with the <a href="http://www.cs.umd.edu/users/pugh/java/memoryModel/" target="_blank">JMM</a>, a formidable stab at guaranteeing memory consistency across platforms where not only instruction ordering is not guaranteed (x86, ARM, I&#8217;m looking at you), but Sun, the main vendor readily agreed to mandate this in a portable, runtime-agnostic spec rather than opaquely baking the behaviour into their reference implementation. And even with all that good-will, political clout and money it took them until Java 5 to get it right. Kinda.

So yeah, congrats- you&#8217;ve beaten the odds and the threat of irrelevance and now your app has to scale; it&#8217;s time to go tackle some _hard_ problems. Concurrency is one of them 😉

* * *

&nbsp;

*because c&#8217;mon, nobody else fuckin&#8217; reads this  
**on MRI? What are you smoking?