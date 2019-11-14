---
id: 619
title: On DRY Spells, Pt. 2
date: 2017-01-16T23:34:44+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=619
permalink: /2017/01/16/on-dry-spells-pt-2/
categories:
  - Uncategorized
---
In this sporadic series whose continuation is contingent on how I&#8217;m feeling on any given day/week/month, I pick some well known OO design patterns and see if they&#8217;re any use in a dynamic and/or functional language. [Last time](https://chris.mowforth.com/2016/04/25/on-dry-spells-pt-1/) we saw how the template method pattern almost evaporates when you use a language with lexical closures. This time let&#8217;s explore the phenomenon of <a href="https://en.wikipedia.org/wiki/Multiple_dispatch" target="_blank">multiple dispatch</a>.

## Alright, what is it?

Cool, I&#8217;ve lured you in, this is pure linkbait isn&#8217;t it?!<img src="https://chris.mowforth.com/wp-includes/images/smilies/simple-smile.png" alt=":)" class="wp-smiley" style="height: 1em; max-height: 1em;" /> Let&#8217;s say you&#8217;re coming from Java land, specifically a place where you have an interface like this:

<pre><code class="language-javascript">public interface Widget {
    void doSomething();
}</code></pre>

Let&#8217;s also say you&#8217;ve got two concrete implementations:

<pre><code class="language-javascript">public class WidgetA implements Widget {
    @Override
    public void doSomething() {
        System.out.println("WidgetA");
    }
}

</code></pre>

<pre><code class="language-javascript">public class WidgetB implements Widget {
    @Override
    public void doSomething() {
        System.out.println("WidgetB");
    }
}</code></pre>

All very taxing stuff, it must be said. What if you want to make a call to doSomething() without caring about the receiver type? That&#8217;s fine; it&#8217;s a normal use of type polymorphism in Java and the compiler will happily figure out the correct type to dispatch the method call onto. For example:

<pre><code class="language-javascript">public class WidgetConsumer {
    
    public void iDunno(Widget widget) {
        widget.doSomething();
    }
}</code></pre>

Whatever Widget implementation you pass in, the call site can be resolved at compile time. Let&#8217;s step it up a notch: what if we want to hit a different version of iDunno() depending on Widget subtype?

<pre><code class="language-javascript">public class WidgetConsumer {
    
    public void iDunno(WidgetA widget) {
        System.out.println("WidgetA logic");
    }

    public void iDunno(WidgetB widget) {
        System.out.println("WidgetB logic");
    }
}</code></pre>

Trying to call iDunno() now will make the compiler explode- it doesn&#8217;t know which overloaded version to use. The normal workaround for this secondary dispatch is to use the visitor pattern. Here, the two lumps of iDunno logic in WidgetConsumer would suffice as a visitor, while the concrete Widget implementations are the things that get visited. To top the example off we need some kind of client to kick off the visitation, and we need a new method on widget to allow the WidgetConsumer to visit each implementation:

<pre><code class="language-javascript">
public interface Widget {
    void doSomething();
    void accept(WidgetConsumer consumer);
}

public class WidgetA implements Widget {
    @Override
    public void doSomething() {
        System.out.println("WidgetA");
    }

    @Override
    public void accept(WidgetConsumer consumer) {
        consumer.iDunno(this);
    }
}

public class WidgetB implements Widget {
    @Override
    public void doSomething() {
        System.out.println("WidgetB");
    }

    @Override
    public void accept(WidgetConsumer consumer) {
        consumer.iDunno(this);
    }
}

public class WidgetClient {

    public static void main(String[] args) {
        WidgetConsumer consumer = new WidgetConsumer();
        WidgetA widget = new WidgetA();

        widget.accept(consumer); // should print "WidgetA logic"
    }

}</code></pre>

That&#8217;s a common <a href="https://en.wikipedia.org/wiki/Design_Patterns" target="_blank">GoF workaround</a>, and for my money I think it&#8217;s pretty clever (although hampered by the lack of expressiveness afforded by a statically typed language). What if we had a language which relaxed the need to resolve the concrete type ahead of time? They&#8217;re actually not ten-a-penny. We have the same problem in JavaScript:

<pre><code class="language-javascript">function iDunno(hopefullyAString) {
    // do something with a string
}

function iDunno(hopefullyANumber) {
    // do something with a number
}</code></pre>

That code doesn&#8217;t even make sense to read; clearly, dynamic and/or weak typing on its own is not sufficient. Are we stuck with visitors forever then?

## Just put up with Java already

Nooooo! There are a few of languages which can resolve this issue and spring to mind, but I&#8217;m going to revert to character and bang the drum for Clojure. Go figure. You&#8217;ve committed to reading this waffling post already so you get what you pay for. And Clojure is great anyway. This one&#8217;s actually a bit of a misnomer- despite targeting the JVM, the JVM actually only supports single polymorphic dispatch through a classic <a href="https://en.wikipedia.org/wiki/Virtual_method_table" target="_blank">vtable</a> or similar lookup (although most modern JVMs can elide the lookup if they know a call site is monomorphic- that&#8217;s a speculative optimisation).

_Anyway_, WTF does a Clojure multimethod look like? Well, it looks like this good Sir / Madam:

<pre><code class="language-scheme">; define the signature w/ dispatch function
(defmulti iDunno class)

; handler for WidgetA
(defmethod com.example.WidgetA [_] (println "WidgetA"))

; handler for WidgetB
(defmethod com.example.WidgetB [_] (println "WidgetB"))
</code></pre>

&#8230;where class is a function that gets called before the actual method + args are invoked. Awkwardly, that single example pretty much wraps up the solution; if you have a language with constructs for runtime method dispatch, what we have is another pattern which almost evaporates.

This isn&#8217;t hating on Java or any other statically typed language- but it&#8217;s a cautionary tale that getting stuck in a particular programming paradigm puts you at risk of falling prey to the <a href="https://en.wikipedia.org/wiki/Law_of_the_instrument" target="_blank">Law of the instrument</a>, and at the risk of much trolling, that&#8217;s something that is particularly prevalent in industrial OOP circles: Java/C#/C++, I&#8217;m looking at you.