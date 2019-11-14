---
id: 176
title: On DRY Spells, Pt. 1
date: 2016-04-25T19:32:34+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=176
permalink: /2016/04/25/on-dry-spells-pt-1/
categories:
  - Uncategorized
---
I was thinking of finishing off an <a href="https://docs.oracle.com/javase/8/docs/technotes/guides/vm/multiple-language-support.html" target="_blank">InvokeDynamic</a> post that&#8217;s been languishing as a draft for a couple of weeks now, but having worked on more than a few dynamically-typed codebases in the past couple of months I decided to change tack and <del>rant</del> make observations about design patterns instead.

To be specific, I&#8217;ve seen a lot of code written in Python which is perfectly well structured but eschews many of the constructs dynamic languages give you to make it more succinct and less repetitive (without compromising readability / maintainability). It&#8217;s surprising to see even experienced developers missing opportunities to aptly DRY shit up, so I&#8217;m going to take a couple of leaves out of Peter Norvig&#8217;s <a href="http://norvig.com/design-patterns/" target="_blank">presentation</a> reconciling typical GoF patterns with the features of a language like Python. Norvig contends that of the 23 original patterns presented in the <a href="https://en.wikipedia.org/wiki/Design_Patterns" target="_blank">Gang of Four book</a>, 16 either become redundant or qualitatively simpler when implemented in a dynamic language. There are a couple of recurring points where refactoring is so obvious it&#8217;s almost silly not to do, so I&#8217;ll focus on them. First up:

### Template Method Pattern

I started with <a href="https://en.wikipedia.org/wiki/Template_method_pattern" target="_blank">templates</a> because you probably use them all the time without realising it. The wikipedia example cites a Java app where each concrete type has a few methods to implement, but in practice it&#8217;s quite common to see something like this:

<pre><code class="language-python">def setup_something():
    # do some kind of setup
    return 1

def finish_doing_something(arg):
    # do something else
    print(str(arg))

def some_method_a():
    a = setup_something()
    # logic unique to this function, e.g.:
    a = 2
    finish_doing_something(a)

def some_method_b():
    b = setup_something()
    # logic unique to this function, e.g.:
    b = 3
    finish_doing_something(b)</code></pre>

The only invariant in those two functions is a 1-liner. Imagine it isn&#8217;t, and the logic surrounding this work is also more involved than returning a number; this happens all the time in web applications that require some initial and final filtering / decorating of values, or to commit something to a database, e.t.c. I&#8217;d like to say I&#8217;ve seen this DRYed up in statically-typed languages like Java with a nice wikipedia-like solution, but too often people just copy and paste the same ritual and change what they need (an IDE can do this kind of thinking for you, right?). Without the nominative typing thrust in your face, it&#8217;s equally tempting to do the same- but first-class functions give you an easy way out: 

<pre><code class="language-python">def setup_something():
    # do some kind of setup
    return 1

def finish_doing_something(arg):
    # do something else
    print(str(arg))

def do_work(work):
  val = setup_something()
  return finish_doing_something(work(val))

def some_method_a():
    return do_work(lambda val: 2)

def some_method_b():
    return do_work(lambda val: 3)</code></pre>

OK, the payback of a few single-line methods is negligible but I bet your average Django app has a lot of easy pickings if you wanted to apply this.

Frankly, if you&#8217;re going to drop all the interesting bits of a language that SICP gave us, why not use C++? One answer may be that to _really_ develop the part of your brain that decomposes code more effectively you have to use a language that&#8217;s so far removed from commonly used imperative languages that you _have to_. But that&#8217;s for another rant. I&#8217;ll just leave you with this motivational video next time you&#8217;re holding off on a refactoring: