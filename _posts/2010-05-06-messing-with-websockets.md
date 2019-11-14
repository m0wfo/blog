---
id: 456
title: Messing with WebSockets
date: 2010-05-06T19:34:21+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=456
permalink: /2010/05/06/messing-with-websockets/
categories:
  - Uncategorized
---
Well, here&#8217;s my contribution to the [so far] rather hacky collection of ruby Websocket clients. It&#8217;s MacRuby only as it utilises NSInputStreams as opposed to traditional ruby TCP sockets, and the new Grand Central calls in 0.6. Using it is just a case of creating an instance of Websocket, calling #push on it and implementing a callback to receive whatever gets sent back:



More refinement to come, and maybe a demo. Hack away <a title="WebSocket" href="http://github.com/m0wfo/websocket" target="_blank">here</a>.