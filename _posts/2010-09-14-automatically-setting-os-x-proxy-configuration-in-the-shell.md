---
id: 459
title: Automatically setting OS X proxy configuration in the shell
date: 2010-09-14T19:45:49+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=459
permalink: /2010/09/14/automatically-setting-os-x-proxy-configuration-in-the-shell/
categories:
  - Uncategorized
---
I&#8217;ll make it quick. It&#8217;d be nice to have the proxy settings from the OS X network preference pane reflected in an _http_proxy_ variable when they&#8217;re needed- it&#8217;s useful if your laptop regularly finds itself in more than one location. Unfortunately most of the solutions google yields are complex exercises in technical masturbation, so here&#8217;s mine: 9 lines of code, no settings stored in a file somewhere in /etc, no symlinks, no messing around:

Save .proxy_export.rb in your home folder:



Then add this to the end of your .bash_profile



Job done. As [Albert](http://en.wikipedia.org/wiki/Albert_Einstein) said, &#8220;Make everything as simple as possible, but not simpler.&#8221;