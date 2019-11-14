---
id: 533
title: Avoiding looking for a host in a haystack
date: 2009-03-10T21:52:54+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=533
permalink: /2009/03/10/avoiding-looking-for-a-host-in-a-haystack/
categories:
  - Uncategorized
---
This kind of stuff shouldn&#8217;t happen. Last week at [work](http://rawnet.com "rawnet") I got a Linux server set up & configured for a specific project. All was well so I forgot about it and shut it down last thing on Friday. Some days later I found that as the machine had been assigned a different IP, it proved to be somewhat elusive.

I know OS X starts an mDNS broadcast of your local SSH service when you enable remote login, which is great for finding local machines you seldom log into. As it happens, it&#8217;s pretty easy to do with [Avahi](http://avahi.org/ "Avahi"), too.

  1. Install Avahi. This is pretty well covered in the Avahi wiki and in endless forum threads. The likelihood is that your distro has a package.
  2. Create a file called _ssh.service_ in _/etc/avahi/services_. It should contain something like this:

<http://gist.github.com/76964>

Restart the daemon.

If you&#8217;re on a mac, pressing shift + cmd + K should pop up a list of available hosts with the service name you gave showing:

<div class='p_embed p_image_embed'>
  <img alt="Media_httpfarm4static_glfld" height="431" src="http://getfile3.posterous.com/getfile/files.posterous.com/import-kppb/bJvnkFHxHzwdxyFCgdtczjEtAuyxDFmDDiwDqywqGzEDexiEIImenBtoBjkv/media_httpfarm4static_GlflD.jpg.scaled500.jpg" width="431" />
</div>

More info [here](http://avahi.org/download/avahi.service.5.xml "avahi-publish").