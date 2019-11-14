---
id: 524
title: Logentries + Clojure
date: 2013-10-19T21:43:58+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=524
permalink: /2013/10/19/logentries-clojure/
categories:
  - Uncategorized
---
It goes without saying that if you have an investment in Java, you can easily integrate [Logentries](https://logentries.com/) with your existing codebase. What's maybe not so obvious is that our Java integration works for _any_ language that targets the JVM.

Since it doesn't seem to be such a well-trodden path, let's dive straight in and add the Logentries log4j appender to a toy [Clojure](http://clojure.org) app. This example reads more easily if you have some rudimentary knowledge of the language and its [build tools](http://leiningen.org/), but even if you don't you should see that it's not a hard task.

OK, go: fire up an empty project:

<pre><code class="language-bash">lein new le-example</code></pre>

Add the requisite dependencies; your _project.clj_ should look something like this:

<pre><code class="language-scheme">(defproject le-example "1.0.0-SNAPSHOT"
  :description "Sample clojure app for Logentries"
  :dependencies [[org.clojure/clojure "1.3.0"]
                 [org.clojure/tools.logging "0.2.6"]
                 [log4j/log4j "1.2.17"]
                 [com.logentries/logentries-appender "1.1.20"]])</code></pre>

You'll also want to create a _resources_ folder in the project root and add a _log4j.xml_ configuration; here's one we [made earlier](https://github.com/logentries/le_java#log4j). Swank will pick up anything in this folder and add it to the classpath, similar to maven exec's behaviour.

Run _lein deps_ and we're good to play around in the REPL (you're using emacs, right?). In core.clj, we'll create a function that just spits 10 events down the wire to a given log:

<pre><code class="language-scheme">(ns le-example.core
  (:use [clojure.tools.logging :only (info)])
  (:gen-class))

(defn print-some-logs []
  (dotimes [n 10]
    (info "Hello, logger!")))</code></pre>

Invoke it&#8230;

<pre><code class="language-bash">1 ; SLIME 20100404
2 user> (ns le-example.core)
3 nil                                                                                                                                                                                                             
4 le-example.core> (print-some-logs)
5 nil                                                                                                                                                                                                             
6 le-example.core> </code></pre>

&#8230;and in a couple of seconds you should see some rather uninformative info messages appear in your Logentries log console:

<pre><code class="language-bash">» 19:23:24.790  2013-10-19 19:21:57 +0100 INFO  (logging.clj:272)  Hello, logger!
» 19:23:24.803  2013-10-19 19:21:57 +0100 INFO  (logging.clj:272)  Hello, logger!
» 19:23:24.803  2013-10-19 19:21:57 +0100 INFO  (logging.clj:272)  Hello, logger!
» 19:23:24.803  2013-10-19 19:21:57 +0100 INFO  (logging.clj:272)  Hello, logger!
» 19:23:24.803  2013-10-19 19:21:57 +0100 INFO  (logging.clj:272)  Hello, logger!
» 19:23:24.803  2013-10-19 19:21:57 +0100 INFO  (logging.clj:272)  Hello, logger!
» 19:23:24.803  2013-10-19 19:21:57 +0100 INFO  (logging.clj:272)  Hello, logger!
» 19:23:24.803  2013-10-19 19:21:57 +0100 INFO  (logging.clj:272)  Hello, logger!
» 19:23:24.803  2013-10-19 19:21:57 +0100 INFO  (logging.clj:272)  Hello, logger!
» 19:23:24.803  2013-10-19 19:21:57 +0100 INFO  (logging.clj:272)  Hello, logger!</code></pre>

So that's pretty cool.

If you're using JVM-hosted languages in your app and you're hitting issues with logging, I'd be interested to hear from you.