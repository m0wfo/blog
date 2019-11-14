---
id: 518
title: 'Parallel map: more JRuby concurrency mischief'
date: 2011-12-25T21:40:38+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=518
permalink: /2011/12/25/parallel-map-more-jruby-concurrency-mischief/
categories:
  - Uncategorized
---
Like my last post this is more for my future benefit but if anybody else finds it useful then that&#8217;s cool too. Unlike the last one the fruits of my tinkering yielded a nice linear speedup.

Ok, let&#8217;s parallelize Array#map. We&#8217;ll break down the task as follows:

  1. Split the array into chunks
  2. Execute the chunks in asynchronously, in parallel, waiting for them all to complete
  3. Merge the chunks into a new array and return it

How many chunks is optimal? There&#8217;s no definitive answer; In the past I&#8217;ve opted for a very large number of small sub-arrays, e.g. for concurrent divide & conquer reductions where the minimal array length was some low power of the number of processors (I&#8217;ve [played with associative reduction algorithms](http://chris.mowforth.com/parallel-reduction-in-objective-c) in the past). For our #pmap method I&#8217;m just going to split the original array up into as many chunks as there are logical cores on my machine. How do you find that out in JRuby? Java to the rescue again:

<pre><code class="language-ruby">$cores = Runtime.getRuntime.availableProcessors</code></pre>

Now we need a pool of workers to assign tasks to. As parallel mapping is strictly CPU bound, a thread pool with fixed thread count but an unbounded work queue is probably most appropriate:

<pre><code class="language-ruby">queue = Executors.newFixedThreadPool($cores)</code></pre>

That&#8217;s our [ExecutorService](http://docs.oracle.com/javase/1.5.0/docs/api/java/util/concurrent/Executors.html#newFixedThreadPool(int)) up & running, we just need to do a bit of housekeeping before we can write our Array#pmap method. This is where Java&#8217;s baroque-complexity boilerplate rears its ugly head (wouldn&#8217;t it be nice if Executors could take [lambdas](http://www.javac.info/closures-v05.html) as arguments for mass invocation?). Basically we implement the [Callable](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Callable.html) interface- I instantiate my Task implementation with a block which the executor calls when it executes:

<pre><code class="language-ruby">class Task
  include Callable

  def initialize(&#038;block)
    @work = block
  end

  def call
    @work.call
  end
  
end</code></pre>

So now we&#8217;re good to go. Array#pmap here takes an executor as a first argument because I didn&#8217;t want the class to be responsible for starting / shutting down the work queue, but that&#8217;s just an implementation decision.

Once the original array is mapped to a set of Task classes, they can be handed to the executor. I call [ExecutorService#invokeAll](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/ExecutorService.html#invokeAll(java.util.Collection)) because it blocks until all the submitted work is done. It returns an array of [FutureTasks](http://docs.oracle.com/javase/1.5.0/docs/api/java/util/concurrent/FutureTask.html) which can be dereferenced immediately (we want the method to return something!):

<pre><code class="language-ruby">class Array

  def pmap(executor, &#038;block)
    # Parcel out the work into chunks to be executed sequentially
    tasks = self.each_slice(self.size / $cores).map do |slice|
      Task.new { slice.map &#038;block }
    end

    # Execute them all, block until they're done
    results = executor.invokeAll(tasks)

    # Dereference and merge all the FutureTasks
    results.reduce([]) { |memo,obj| memo + obj.get }
  end
  
end</code></pre>

So does all that tomfoolery actually buy you any more performance? Time for a highly unscientific benchmark, incrementing an array of the first million Fixnums:

<pre><code class="language-bash">chris@think-chris:~/Documents/Experimentation$ jruby pmap.rb 
"Splendid new Array#pmap"
"That took 0.287166921s"
"Plain old sequential Array#map"
"That took 0.537166921s"</code></pre>

Not bad- I ran it on a Sandy Bridge i5 with 2 cores and 4 CPU threads.

More important than a cheesy parallel mapping imeplementation, I&#8217;ve learned (or is that re-learned?) two axioms about playing with concurrency in JVM hosted languages:

  1. As in the [previous post](http://chris.mowforth.com/thread-safe-attrwriters-in-jruby), executors expect some kind of anonymous inner class as an argument in the absence of closures. You need to be aware of the cost of converting a ruby closure to a Java Runnable, Callable, Future etc; think of I/O-bound problems where allocating extra objects for each request/event is almost certainly not a good thing. You&#8217;ll definitly save time writing in straight Java as you won&#8217;t have to worry whether or not the closure you just used is going to throw all your performance gains out the window.
  2. Don&#8217;t discount the JVM&#8217;s ability to make a fool of your benchmarks.&nbsp;The one above isn&#8217;t worth the screen real-estate it occupies due to its simplicity and duration relative to startup / shutdown time of the VM.&nbsp;Both JIT and Server modes take time to hit a &#8216;quiescent state&#8217; where performance stabilises. Don&#8217;t just look at wall-time, profile your code properly. Lots has been said about this elsewhere so I won&#8217;t rehash what others have articulated better, but be aware HotSpot has its own performance quirks.

The example in its entirety is [here](https://gist.github.com/1519961).