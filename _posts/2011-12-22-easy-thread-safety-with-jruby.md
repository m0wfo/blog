---
id: 514
title: Easy thread safety with JRuby
date: 2011-12-22T21:38:39+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=514
permalink: /2011/12/22/easy-thread-safety-with-jruby/
categories:
  - Uncategorized
---
**EDIT:** Beauty, as they say, is pain. In exchange for liberating your code from locks, piling the work onto a queue is approximately [~10x slower](https://gist.github.com/1513847) than a traditional approach. A quick profile suggests that the expense of block creation is non-trivial. It&#8217;s still nicer to look at though, right?

More for my benefit, but it&#8217;s handy that JRuby lets you use asynchronous Java queues. Want to make something like this thread-safe?

<pre><code class="language-ruby">class Something

  def initialize(value)
    @value = value
  end

  # Not thread-safe

  def inc
    @value += 1
  end

  def dec
    @value -= 1
  end

end</code></pre>

Just wrap the assignment in a [Runnable](http://docs.oracle.com/javase/1.5.0/docs/api/java/lang/Runnable.html) (JRuby coerces blocks/Procs/lambdas into Runnables for you) and submit it to a single-threaded Executor. All calls to inc and dec execute one at a time, in-order:

<pre><code class="language-ruby">require 'java'

java_import java.util.concurrent.Executors

class Something

  def initialize(value)
    @queue = Executors.newSingleThreadExecutor
    @value = value
  end

  def inc
    @queue.execute { @value += 1 }
  end

  def dec
    @queue.execute { @value -= 1 }
  end

  def finalize
    @queue.shutdown
  end
end</code></pre>

Tasks are applied FIFO, @value stays consistent without locks- [sound familiar?](https://developer.apple.com/reference/dispatch)