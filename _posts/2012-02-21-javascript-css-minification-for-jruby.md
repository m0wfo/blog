---
id: 496
title: JavaScript / CSS minification for JRuby
date: 2012-02-21T21:31:52+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=496
permalink: /2012/02/21/javascript-css-minification-for-jruby/
categories:
  - Uncategorized
---
I didn&#8217;t think I&#8217;d have to do this, but after seeing that the [ruby-yui-compressor gem](https://github.com/sstephenson/ruby-yui-compressor/) forks a process every time, I thought that, whilst using JRuby, that&#8217;s unequivocally Doing It Wrong&trade;. So I wrapped the YUI Java compressor library in a gem and called it JMinify. If anybody needs to compress their assets and they&#8217;re using JRuby, [knock yourselves out](https://github.com/cmowforth/jminify).