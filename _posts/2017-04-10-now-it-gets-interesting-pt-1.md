---
id: 188
title: Now it gets interesting (Pt.1)
date: 2017-04-10T21:16:41+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=188
permalink: /2017/04/10/now-it-gets-interesting-pt-1/
categories:
  - Uncategorized
---
I haven&#8217;t done any full-time web development for more than 5 years, and that&#8217;s been a conscious decision. Engineering <a href="https://hbr.org/2014/03/hire-slow-fire-fast" target="_blank">talent-density</a> seems to vary inversely with proximity to customer-facing code. My extremely jaded view of a web developer is someone who scraped through college courses like compiler design or a variant of <a href="http://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/" target="_blank">SICP</a> and subsequently internalised Steve Yegge&#8217;s tongue-in-cheek modus-operandum:

> You know how to Program. You&#8217;re a Programmer. Now you just need to learn APIs and stuff.

This attitude will land you a comfortable job straight out of college, and if you&#8217;re semi-articulate and like to shout a lot, you&#8217;ll go far. Many workplaces such as consulting firms, ad agencies, e.t.c where customers hit you with predictable, repetitive and discrete workloads need this kind of manpower.  Startups don&#8217;t.

<blockquote class="twitter-tweet" data-lang="en">
  <p dir="ltr" lang="en">
    Just finished rewriting all my ember apps on backbone, and now I get to rewrite them all on angular. Feels good to make progress!
  </p>
  
  <p>
    — Hipster Hacker (@hipsterhacker) <a href="https://twitter.com/hipsterhacker/status/466077431061696512">May 13, 2014</a>
  </p>
</blockquote>

That&#8217;s unfortunate, because today&#8217;s nontrivial web applications are crying out for programmers with a strong CS interest. Things like <a href="https://facebook.github.io/react/" target="_blank">React</a> or <a href="http://elm-lang.org/" target="_blank">Elm</a> wouldn&#8217;t have seen the light of day without people with an interest in FP and persistent data structures. Speaking of React,  you might have noticed the notion of immutability mentioned a lot in the docs. It&#8217;s no accident; shared mutable state used to be something that would only haunt people doing heavy concurrent programming [read: not in a browser]. If you&#8217;ve ever been unable to use a site on an iPad because it was stuttering so much, you&#8217;ll know that modern UI frameworks are doing a hell of a lot of work passing data around the application. You&#8217;ve got to conclude that even with sequentially fast interpreters and a simple [read: nonexistent ;)] threading model, the free ride is over for JS developers. You see, mutating objects like this:

<pre class="language-javascript"><code>var shoppingCart = {
  "apples": 2,
  "oranges": 3
};

shoppingCart.sh_t = "bananas";</code></pre>

&#8230;has a few drawbacks: first up, modern JS interpreters &#8216;de-<a href="https://en.wikipedia.org/wiki/Virtual_method_table" target="_blank">virtualize</a>&#8216; calls to object members when they&#8217;re defined: in essence, creating classes with fast member lookup times in the background. Tacking on properties after you&#8217;ve defined an object means those generated lookups are either dirty and have to be recreated (expensive), or you have to go through some kind of slow v-table-like lookup (also expensive). Have a look at <a href="http://thibaultlaurens.github.io/javascript/2013/04/29/how-the-v8-engine-works/" target="_blank">this</a> for more info on how V8 does it.

Secondly, since you&#8217;re altering the value of the same object _in-situ,_ you&#8217;ve lost the original state; an observer has no idea what changed and how you can deterministically arrive at this place. In a concurrent environment you&#8217;d also need some kind of coordination mechanism so different threads of execution see a correct view of _shoppingCart_ at the right time (consistency + visibility). On top of this, broadcasting changes in mutable state to a potentially HUGE set of observers can be incredibly CPU intensive. Accumulate enough of them, working away, blocking DOM updates and you end up with <a href="http://jankfree.org/" target="_blank">Jank</a>. Ever had a conversation like- _&#8220;oh shit, the number of AngularJS watch functions grows linearly with the number of products on the page! We need to rethink our approach before this launches! And I was gonna hit the gym tonight :/&#8221;_? Not cool. On top of this, passing mutable object references into <a href="https://en.wikipedia.org/wiki/Pure_function" target="_blank">impure functions</a> will turn your nice application into spaghetti pretty quick. How often do you encounter bugs because some function is silently adding/removing/updating an object member or array value? If it&#8217;s like the majority of JS codebases I&#8217;ve seen, this probably happens alarmingly frequently.

React pulls in a fundamental tenet of functional programming languages: **_mutate nothing_**. In concurrent environments, this obviates the need for programmers to deal with locking mechanisms in their own code. In the browser, the notion of immutability totally removes the concept of subscribing to a changing reference: data objects are treated as _values_, and work is only done when a value _changes_.

### Why should I care? It&#8217;s just one more API to learn, right?

Yeah, I guess you&#8217;re right: you can hit up the todo-list example on the React.js site and bullshit your way through another framework. Meanwhile I can attain catharsis by looking for javascript-tagged questions on Stackoverflow and inform the OP that web developers are all morons. Both are fun endeavours but neither are terribly good paths for self-improvement. Instead I was hoping to pique your interest in some more abstract but enduring concepts by dipping into <a href="https://facebook.github.io/immutable-js/" target="_blank">immutable.js</a> and one of its core structures: the _Hash Array Mapped Trie_.

### HAMT?

<a href="https://youtu.be/Kgjkth6BRRY" target="_blank">This highly relevant video</a> explains everything.

You see, if all your functions are deterministic and all your values are immutable, how do you effect change in a program? By creating _new_ values based on your existing ones. People in Java-land may have come across things such as:

<pre class="language-javascript"><code>ImmutableMap&lt;String, String> foo = ...
// take foo, add 'new' => 'kvp'
ImmutableMap&lt;String, String> bar = ImmutableMap
    .builder()
    .addAll(foo)
    .put("new", "kvp")
    .build();</code></pre>

That keeps foo unchanged and is totally nondestructive but incurs a full copy of foo into the new bar, all for the sake of adding a single new key-value pair :/ Do this a few million times in a loop and you&#8217;ll get something much slower than the traditional imperative version. So immutability gives us safety + correctness at the expense of performance; can we do anything about that?

This is where immutable, **persistent** data structures* come in. While plain immutable structures prevent explicit modification outright, persistent ones outwardly allow alteration but internally generate new values, leaving the original untouched, all with good complexity guarantees (i.e. close to a mutable equivalent).  Put another way, imagine a variant of

<pre>shoppingCart.sh_t = "bananas";</pre>

above which:

  1. Returns a new object containing all of shoppingCart plus the new member
  2. Leaves the old reference of _shoppingCart_ untouched
  3. Has roughly the same of performance of adding a KVP to a mutable object (no **_O(n)_** deep copies)

Got it? Good.

When the <a href="http://clojure.org/" target="_blank">Clojure</a> programming language had to come up with an immutable but performant alternative to the classic Java <a href="https://docs.oracle.com/javase/7/docs/api/java/util/HashMap.html" target="_blank">HashMap</a>, Rich Hickey based the implementation on a <a href="http://lampwww.epfl.ch/papers/idealhashtrees.pdf" target="_blank">2001 paper</a> by Phil Bagwell describing a hash structure based on a <a href="https://en.wikipedia.org/wiki/Trie" target="_blank">trie</a>. What&#8217;s more relevant in the context of this rant is that Facebook has decided that a JavaScript implementation is feasible to integrate with React and Flux.

### So how does a HAMT work?

I&#8217;ll get the surprise out of the way: it&#8217;s based on <a href="https://en.wikipedia.org/wiki/Hash_function" target="_blank">hashing</a> like a regular map/dictionary/whatever-you-call-it. The difference is what a HAMT _does_ with that hash. A typical map implementation would look something like this:

<img class="alignnone size-medium wp-image-274" src="https://chris.mowforth.com/wp-content/uploads/2016/05/regularhash1.svg" alt="regularhash" /> 

Essentially we have a group of buckets and a function which the key is passed through; the output of the function dictates which bucket it lands in. If we have a crappy hash function or not enough buckets, multiple keys end up in the same bucket. We then have to resolve those collisions, commonly by some kind of &#8216;probing&#8217; strategy where you stick your value in the next consecutive free bucket or have some kind of chain structure inside each bucket. Either way it degrades to an _O(n)_ lookup in the worst case.

HAMTs split the hashed value over a <a href="https://en.wikipedia.org/wiki/Trie" target="_blank">trie</a> (sorry, more data structures). How? If the hash function yields an _n_-bit number then the bits in it can be partitioned into various _t_-size chunks, each of which is a node in the trie. Let&#8217;s see an example- ECMAScript only specifies one 64-bit wide number type, so we&#8217;ll go with _n_=64 and _t_=8**:

<img class="alignnone size-medium wp-image-289" src="https://chris.mowforth.com/wp-content/uploads/2016/05/bitpartition.svg" alt="bitpartition" /> 

In part 2 we look at the significance of tries, space-efficient ways of representing them and how to do an HAMT lookup.

* * *

* if you&#8217;re in an interview situation, asking a candidate to make the distinction between immutable and persistent data structures is one way of seeing if they&#8217;ve been doing more than hacking out Spring templates in Eclipse for the last 5 years.

** if you&#8217;ve been writing in a language, including JavaScript for more than a couple of years, you should know the size and layout of the fundamental types.