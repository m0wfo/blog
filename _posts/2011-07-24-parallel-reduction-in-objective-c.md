---
id: 484
title: Parallel reduction in Objective-C
date: 2011-07-24T21:25:42+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=484
permalink: /2011/07/24/parallel-reduction-in-objective-c/
categories:
  - Uncategorized
---
Ruby and Clojure have spoilt me and coming back to Objective-C has starved me of a heap of nice ways to manipulate collections.

Born out of this frustration and a drive to use GCD and blocks for something useful, I&#8217;ve started creating my own framework of categories to add into NSArray, NSDictionary and who knows what else right now (that&#8217;s for another post).

After toiling for an afternoon and realising that you can now create explicitly concurrent dispatch queues in Lion, I managed to work out a divide & conquer reduce algorithm for associative functions:



I mentioned OS X Lion; specifically look at line 18:



Since we assume the user has provided an associative function, we don&#8217;t care about the order in which our recursive calls return. This means we can use the shiny new DISPATCH\_QUEUE\_CONCURRENT constant in 10.7 to send work off in a parallel manner ([thanks for the tip, Kazuki](http://stackoverflow.com/questions/6722995/parallel-reduce-algorithm-implementation)).

For this to work you need a means of splitting the array into sub-arrays of ever decreasing size, until they hit a limit where it&#8217;s practical to reduce them serially:



Right now I&#8217;m&nbsp;na&iuml;vely assuming that the smallest unit of work is when a sub-array satisfies the inequality:

<p style="text-align: center;">
  <span style="font-size: large;"><span style=""> </span>l &le; n ∕ P&sup2; ; {l, n, P}&nbsp;&sub;&nbsp;ℤ&nbsp;∖ {0}</span>
</p>

Where l the base case, n is the length of original NSArray to be conqured and P is the number of processors.

Given the fact that n will typically be vastly greater than P on most personal computers and mobile devices you&#8217;d intuitively think that mandating a relatively large base case is the sensible thing to do- the smaller the job, the more time is spent recursively splitting arrays and dispatching jobs rather than doing any real work.

I&#8217;m purposefully omitting any benchmarking because you know what [Disraeli said](http://en.wikipedia.org/wiki/Lies,_damned_lies,_and_statistics "Lies, damned lies and statistics"), but from my somewhat confined set of &nbsp;experiments I&#8217;m seeing a ≃1.4x increase in execution speed of associative functions. This isn&#8217;t production quality and won&#8217;t be making its way into CMFunctionalAdditions but it illustrates the potential of higher-order functions and Grand Central Dispatch in Objective-C.