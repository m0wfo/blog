---
id: 490
title: A quick preview of the actor API
date: 2012-05-18T21:29:57+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=490
permalink: /2012/05/18/a-quick-preview-of-the-actor-api/
categories:
  - Uncategorized
---
Since the actor / future API is starting to mature, I thought it&#8217;d be nice to show what concurrent programming in JavaScript actually looks like. To begin with, here&#8217;s the canonical actor ping-pong example; two actors collaboratively incrementing a counter without explicitly sharing any state:

<pre><code class="language-javascript">// Shamelessly pulled from the Rubinius example
// at http://rubini.us/doc/en/systems/concurrency/
var ping = new actor(function(msg) {
  if (msg === 1000) {
    console.log(msg);
  } else {
    pong.send(msg++);
  }
});

var pong = new actor(function(msg) {
  if (msg === 1000) {
    console.log(msg);
  } else {
    ping.send(msg++);
  }
});

ping.send(1);</code></pre>

You can also safely hot-swap the message handler inside an actor. When you call _upgrade_, the supplied function gets pushed onto an internal stack and is used as the new handler. Calling _downgrade_ pops it off and uses the previous function (or the original one if you&#8217;re at the bottom of the stack, in which case it&#8217;s idempotent).

<pre><code class="language-javascript">var x = new actor(function(msg) {
  console.log('Foo!');
});

x.send('something'); // => 'Foo!'

x.upgrade(function(msg) {
  console.log('Bar!');
});

x.send('something'); // => 'Bar!'

x.downgrade();

x.send('something'); // => 'Foo!'</code></pre>

Because message processing is mutually exclusive (any given actor will only process one message in its mailbox at any one time) this is generally quite safe to perform on a live system- no restarts! I say generally because messages passed to actors must be immutable, and neither JavaScript nor Java can enforce that sufficiently strongly to stop the ignorant or the strong-willed from throwing caution to the wind. For now, you just have to be disciplined. But it&#8217;s a promising start, anyway.