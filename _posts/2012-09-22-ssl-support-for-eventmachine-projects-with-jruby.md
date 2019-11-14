---
id: 502
title: SSL support for eventmachine projects with JRuby
date: 2012-09-22T21:33:19+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=502
permalink: /2012/09/22/ssl-support-for-eventmachine-projects-with-jruby/
categories:
  - Uncategorized
---
I&#8217;ve been hacking on [Foxbat](https://github.com/m0wfo/foxbat) for a while now but haven&#8217;t really said anything about it. If you&#8217;re using JRuby and you want eventmachine to support SSL/TLS, connection timeouts, proper multithreading and more, give it a try. Using it is as simple as _requiring_&nbsp;the gem before any dependencies. For example, here&#8217;s&nbsp;[em-websocket](https://github.com/igrigorik/em-websocket)&#8216;s&nbsp;initial example with SSL support on Java:

<pre><code class="language-ruby">require 'foxbat'
require 'em-websocket'

EventMachine::WebSocket.start(:host => "0.0.0.0", :port => 3333,
    :secure => true, :keystore => '/path/to/keystore') do |ws|
  ws.onopen {
    puts "WebSocket connection open"
     # publish message to the client
    ws.send "Hello Client"
  }

  ws.onclose { puts "Connection closed" }
  ws.onmessage { |msg|
    puts "Recieved message: #{msg}"
    ws.send "Pong: #{msg}"
  }
end</code></pre>

Currently the only subset of EM it supports is TCP clients/servers, SSL, one-shot timers and #defer but it shouldn&#8217;t be hard to extend- it&#8217;s just a wrapper around [netty](http://netty.io/). Not much else to it.