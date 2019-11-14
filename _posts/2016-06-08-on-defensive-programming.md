---
id: 332
title: On Defensive Programming
date: 2016-06-08T08:04:51+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=332
permalink: /2016/06/08/on-defensive-programming/
categories:
  - Uncategorized
---
tl;dr: unless you work in an environment where the direct consequences of a production bug are death or sanction by the <a href="https://en.wikipedia.org/wiki/U.S._Securities_and_Exchange_Commission" target="_blank">SEC</a>, **please do not do it**.

[<img class="size-full wp-image-469" src="https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2016/06/US_Navy_030328-M-0000X-005_Kuwaiti_firefighters_fight_to_secure_a_burning_oil_well_in_the_Rumaila_oilfields.jpg" alt="030328-M-0000X-005 Southern Iraq (Mar. 27, 2003) -- Kuwaiti firefighters fight to secure a burning oil well in the Rumaila oilfields, set ablaze by Iraqi military forces. Efforts are underway to extinguish fires and protect the region from environmental disaster. Protection of these fields is considered critical to the Iraqi people in support of Operation Iraqi Freedom. Operation Iraqi Freedom is the multi-national coalition effort to liberate the Iraqi people, eliminate Iraqi's weapons of mass destruction, and end the regime of Saddam Hussein. U.S. Marine Corps photo by Pfc. Mary Rose Xenikakis. (RELEASED)" width="900" height="590" />](https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2016/06/US_Navy_030328-M-0000X-005_Kuwaiti_firefighters_fight_to_secure_a_burning_oil_well_in_the_Rumaila_oilfields.jpg)

The longer version is based on something I repeat like a stuck record: when you&#8217;re developing something from scratch, _<a href="http://c2.com/cgi/wiki?MakeItWorkMakeItRightMakeItFast" target="_blank">make it correct, then make it fast</a>_. A corollary to the correct -> fast thing is that a misbehaving program should make its failure mode obvious so it can be remediated.

Yeah, I&#8217;m mostly talking about Java and in particular a couple of things I&#8217;ve seen lately which seem to be informed by either:

  * a misunderstanding of how ExecutorService implementations handle unchecked exceptions
  * an obsession with returning **\*something\*** from a non-void method, even if something is _null_ or _&#8220;ihavenoidea&#8221;_

### 1. Exceptions

O.K, let&#8217;s be clear about this:  <a href="https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ExecutorService.html" target="_blank">ExecutorService</a> implementations (and for that matter, Jetty _queuedthreadpool_s) will swallow up unchecked exceptions and carry on as before. <span style="color: #ff0000;"><strong>You do not have to worry about OS thread death</strong></span>. <span style="color: #3366ff;"><strong>You do not have to worry about suppressing some exception you think <em>may</em> happen</strong></span>. Let it spew a stack trace because that gives somebody who picks up a bug some clue about how a specific routine failed.

If your component relies on a client (HTTP, database, whatever) then it&#8217;s fair-game to use a try/catch block here, because I don&#8217;t care what library your implementation uses; bubbling up _checked_ exceptions here is lazy and also may become redundant if you change your implementation. So do this:

<pre><code class="language-javascript">try {
    client.makeCall(someArgument);
} catch (IOException e) {
    log.error("Couldn't make call!", e);
    throw new RuntimeException(e);
}

// or if you like Guava and javac does too!
try {
    client.makeCall(someArgument);
} catch (IOException e) {
    log.error("Couldn't make call!", e);
    Throwables.propagate(e);
}
</code></pre>

Basically, propagate internal checked exceptions as some kind of runtime exception. This leverages a <a href="https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html" target="_blank">questionable property of Java</a>, but the alternatives are:

  1. force client code to handle null or &#8220;?WTF?&#8221;
  2. force client code to handle IOException or TotallyUnrelatedInternalException

If the underlying failure is unrecoverable anyway, don&#8217;t bubble it up at the level of the compiler. Let the battle-tested logic inside your executor or application server handle shit hitting fans.

### 2. Return values

Let&#8217;s say we have an interface like this:

<pre><code class="language-javascript">public interface IStripper {
    
    String stripThing(String originalThing);
}</code></pre>

What do you expect null or generally bad input to do? Something like this?

<pre><code class="language-java">public GoodStripperImpl implements IStripper {
    
    @Override
    public String stripThing(String originalString) {
        return originalString.substring(0, 8);
    }
}</code></pre>

Or how about this?

<pre><code class="language-java">public PoorStripperImpl implements IStripper {
    
    @Override
    public String stripThing(String originalString) {
        return originalString == null ? "???" : originalString.substring(0, 8);
    }
}</code></pre>

How much fun would it be to root-cause a bug when all you have to go on is fucking _&#8220;???&#8221;_? Again, if the call-site is invoked inside a solid thread-pool implementation, this kind of pessimism is not only unwarranted but suppresses the nub of the issue. And if you&#8217;re confident that a null string is possible input, how about an empty one, or one that&#8217;s too short? Or contains characters in an encoding you didn&#8217;t expect? <a href="https://en.wikipedia.org/wiki/Reductio_ad_absurdum" target="_blank">This</a> springs to mind.

Saving bad input like this is subtle and insidious: it can circumvent the guarantees given by interfaces and if a call chain is written by the same person who wrote the original implementation, you can end up with a pile of code the depends on null-ness or &#8216;???&#8217;-ness, which is a fantastic way of ending up with spaghetti in Java. Please refrain from doing it. Even worse, don&#8217;t make this kind of handiwork wake someone up at 4am only for them to realise a useful stack trace has been replaced with a distant NPE and a garbled string.

### Why are you venting about this now?

Because I felt like it and I hit some arbitrary style-I-don&#8217;t-like threshold. But also because I&#8217;ve worked in Erlang shops where the opportunity cost of even minuscule downtime involved conversations with the CFO; it had a similar gravity to a misreporting fuckup. You often hear people in these circles discussing <a href="http://www.erlang.se/doc/programming_rules.shtml#REF42466" target="_blank">error-kernels</a>* and the <a href="http://c2.com/cgi/wiki?LetItCrash" target="_blank">let-it-crash</a> philosophy, and I know this approach is <a href="http://akka.io/" target="_blank">gaining some traction</a>. It makes sense that microservices take on some of those traits. Either way, it&#8217;s totally at-odds with the notion of defensive programming.

Don&#8217;t try and catch failure based on some poorly informed conjecture, expect every single method call to error and have something solid (e.g. java.util.concurrent, Jetty, actors, whatever) take responsibility for it.

* * *

*Actually that Erlang <a href="http://www.erlang.se/doc/programming_rules.shtml" target="_blank">programming rules page</a> is pretty much universally applicable