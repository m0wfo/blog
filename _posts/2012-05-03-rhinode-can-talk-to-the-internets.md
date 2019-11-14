---
id: 499
title: Rhinode can talk to the internets!
date: 2012-05-03T21:32:35+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=499
permalink: /2012/05/03/rhinode-can-talk-to-the-internets/
categories:
  - Uncategorized
---
I&#8217;ve held off talking about Rhinode up until now, primarily because going into detail about the actor system, STM and concurrent I/O in JavaScript is all a bit meaningless without a concrete example to flesh it out. Well that&#8217;s no longer an impediment, because Rhinode now has a structured and extensible (albeit incomplete) event system and a TCP library emulating the core functions from the&nbsp;_net_&nbsp;module in node.js. What does this mean in practice? For starters it means running this example from the node.js documentation in no longer throws a stack trace:

<pre><code class="language-javascript">var net = require('net');
var server = net.createServer(function(c) { //'connection' listener
  console.log('server connected');
  c.on('end', function() {
    console.log('server disconnected');
  });
  c.write('hello\r\n');
  c.pipe(c);
});
server.listen(8124, function() { //'listening' listener
  console.log('server bound');
});</code></pre>

More profoundly, each &#8216;connection&#8217; event gets executed inside an actor whose work can be distributed across Rhinode instances, all transparently to the developer. Put another way, I&#8217;ve hit one of my short-term goals for this little toy project: to provide an almost seamless way for developers to transplant their JS to the JVM, scaling to multiple cores in the process, without having to resort to the somewhat less refined tools currently available to node.js. Happy hacking. Lots more to come&#8230;

GitHub repo is [here](https://github.com/m0wfo/rhinode).