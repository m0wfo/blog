---
id: 897
title: HDR Histogram for Swift
date: 2020-08-31T00:11:00+00:00
author: chris
layout: post
---

I've been writing quite a lot of server-side [Swift](https://swift.org) lately, and overall I'm really liking it- they've hit that sweet spot between the 'blue-collar' mundane, unsurprising nature of Java while being less memory-intensive and more expressive than Go. I think expanding on that bundle of thoughts warrants a few posts on its own, but for let's focus on that superficial similarity with Java: even the biggest server-side Swift champion would probably concede that it's lacking the rich collection of libraries of a more venerable language, especially when it comes to dealing with distributed systems and all the fun that entails- rpc, failure detection, consensus, membership, metrics, etc etc...

While the [SSWG](https://swift.org/server/) is making some huge strides in terms of its appeal to people using Linux in production, one thing I've used many a time to great advantage in JRE-land is Gil Tene's fine [HdrHistogram](http://hdrhistogram.org) library. If you're not familiar with it, watch one of his many 'how not to measure latency' talks on YouTube, but in a nutshell, his hypothesis is that when it comes to latency, 99th or 99.99th percentiles are not such rare occurrences as you think, especially given the combinatorial nature of microservices talking to each other- to that end, measuring many latency samples at the upper end of what you deem to be the threshold of acceptability, at a high resolution, can characterise a system much more accurately than using a strategy of reservoir sampling.

Anyway, coming to Swift, I found no such implementation 😞but taking direct inspiration from the reference Java impl is pretty trivial. Behold the first iteration [here](https://github.com/tuplestream/HdrHistogram). As a general rule you have to bear a few things in mind with any Java -> Swift porting, and especially so for numeric-heavy code:

* All non-null fields have to be assigned directly in an initializer _before_ you call any methods from said initializer. Swift throws a compile error otherwise
* Byte-ordering can now float depending on architecture- on the JVM you have the luxury of assuming big-endianness everywhere. No more.
* Numeric types with unspecified size (e.g. `Int`) are just that, unspecified size. Either use the exact byte width you need, or get used to using `MemoryLayout.size(ofValue: ...)` a lot
* Don't be afraid to make numeric fields optional- you don't pay the same cost of `Optional<Long>`, e.t.c and it's clearer than testing for actual zero vs unspecified/unset
* There's no unsigned right-shift convenience operator- convert or work with a `UInt` variant if necessary

Most of these caveats are compile-time things, too, so byte-shifting/ordering aside, it's pretty obvious how to do the Right Thing™.

I've only implemented the basic non thread-safe version for now but the API is pretty much identical to the Java version:

```
let histogram = try! Histogram(lowestDiscernableValue: 1, highestTrackableValue: Int64.self.max, numberOfSignificantValueDigits: 2)
histogram.recordValue(1337)
print("\(histogram.percentileAtOrBelowValue(value: 1337))")
```

...etc

More Swift-ing to come, fork & enjoy in the meantime.