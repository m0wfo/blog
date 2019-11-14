---
id: 570
title: Quickly sharing the current working directory over HTTP
date: 2009-01-28T22:04:58+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=570
permalink: /2009/01/28/quickly-sharing-the-current-working-directory-over-http/
categories:
  - Uncategorized
---
I was pretty dismayed to find there wasn&#8217;t a quick ruby equivalent of tip #2 in this <a href="http://www.carsonified.com/development/osx-tips-and-tricks" title="os x dev tips" target="_blank">post</a>; webrick/server.rb has the SimpleServer class but as you can see on <a href="https://ella.pragmaticomm.com/symbian-ruby/browser/trunk/lib/webrick/server.rb?rev=1" title="server.rb" target="_blank">l20</a> it&#8217;s just got a singleton method which yields a block.

So I hacked out a gem giving a shell command which boots up WEBrick, mounts a <a href="http://www.ruby-doc.org/stdlib/libdoc/webrick/rdoc/classes/WEBrick/HTTPServlet/FileHandler.html" title="Webrick::FileHandler" target="_blank">FileHander</a> servelet and then broadcasts its presence via <a href="http://developer.apple.com/opensource/internet/bonjour.html" title="mdns" target="_blank">bonjour</a>.

To get started:

**sudo gem install cmowforth-quickshare**

then:



Where **-s** is the mdns share name and **-p** is the port. Neither arguments are required; just running _&#8220;share&#8221;_ is enough. 

I need to write some tests, but these should be done by the time 0.0.1 goes up on rubyforge. I&#8217;m not going to bother suppressing log output. mDNS rules. 15-all, Python.

Check out the source <a href="http://github.com/cmowforth/quickshare" title="quickshare source" target="_blank">here</a>. Due to its use of the dns-sd utility, I&#8217;ve only tested it on BSD machines as I&#8217;m writing this.