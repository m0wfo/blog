---
id: 511
title: 'CMFunctionalAdditions- multicore ruby-like utilities for Objective-C'
date: 2011-07-29T21:36:15+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=511
permalink: /2011/07/29/cmfunctionaladditions-multicore-ruby-like-utilities-for-objective-c/
categories:
  - Uncategorized
---
Hot on the heels of the last post, I&#8217;ve decided say a bit more about [CMFunctionalAdditions](https://github.com/cmowforth/CMFunctionalAdditions). As I mentioned last time the project is the result of functional and syntatic-sugar widthdrawl symptoms. In a nutshell, CMFunctionalAdditions is my take on being able to call the following in Objective-C without altering the receiver:

  * Map
  * Map with index (each\_with\_index.map&#8230;)
  * Reduce (inject, fold, whatever)
  * Filter (select)
  * Remove (reject, delete, whatever)
  * Partition (the [ruby flavour](http://www.ruby-doc.org/core/classes/Enumerable.html#M001496), not the [clojure flavour](http://clojuredocs.org/clojure_core/clojure.core/partition))
  * Unique
  * Flatten
  * Take (take_while)
  * tbc

Additionally many of the linear-time methods above run in O(n ∕ P) time or better with Grand Central Dispatch. I want concurrency to be transparent to the end user wherever possible. A picture speaks a thousand words so to rip the examples from the CLI demo&#8230;



The framework currently requires Snow Leopard or better but Lion might become a prerequisite (see last post). iOS 4 should work but I haven&#8217;t tried it. I&#8217;m also attempting compilation on Linux as this is being written. As the only dependencies are Foundation and libdispatch (compiled with llvm) it&#8217;d be a nice bonus.

Proper documentation is in the pipeline. Any feedback would be appreciated.