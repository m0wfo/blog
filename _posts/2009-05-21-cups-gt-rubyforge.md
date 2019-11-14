---
id: 535
title: 'Cups => Rubyforge'
date: 2009-05-21T21:53:44+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=535
permalink: /2009/05/21/cups-gt-rubyforge/
categories:
  - Uncategorized
---
All future cups releases will be disseminated through Rubyforge from now on, as opposed to my Github account. I released v0.0.5 of the gem last night, so now you just run:

**_sudo gem install cups_**

if you want in on this piece of craftsmanship. I&#8217;ll also be releasing debian packages in parallel, starting with 0.0.6 if I get any time.

My thanks to Brian Butz for his contributions which have solved some portability issues and fixed several bugs in addition to adding a [Cups#options_for](http://cups.rubyforge.org/doc/classes/Cups.html#M000004) method, which is really cool.