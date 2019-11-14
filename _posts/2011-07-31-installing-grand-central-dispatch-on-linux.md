---
id: 467
title: Installing Grand Central Dispatch on Linux
date: 2011-07-31T08:23:10+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=467
permalink: /2011/07/31/installing-grand-central-dispatch-on-linux/
categories:
  - Uncategorized
---
I&#8217;ve been curious about getting [libdispatch](http://libdispatch.macosforge.org/) and the [blocks runtime](http://compiler-rt.llvm.org/) compiling in Ubuntu for some time but naively guessed that it wouldn&#8217;t be for the faint-hearted since the only useful result yielded by Google was [this thread](http://stackoverflow.com/questions/3147400/using-grand-central-dispatch-in-linux) on SO. How wrong I was!

For this exercise I pulled the [iso for natty](http://releases.ubuntu.com/natty/) server from the ubuntu site and did a fresh install in a VirtualBox VM. The only extra I added during install was an SSH daemon so I could use the terminal on my Mac.

Libdispatch needs llvm/clang, libkqueue and the blocks runtime which are already available through apt-get in natty so let&#8217;s install them.

You&#8217;ll also need libpthread-workqueue0. I had to download the .deb packages from [oneiric here](http://packages.ubuntu.com/oneiric/libpthread-workqueue0) but they installed without any hassle:



I compiled libdispatch itself from source. Don&#8217;t grab it from MacOSforge, [download the tarball](http://archive.ubuntu.com/ubuntu/pool/universe/libd/libdispatch/libdispatch_0~svn197.orig.tar.gz) used to make the .deb package for oneiric. The installation will also need make, autoconf, autogen and libtool. Just to save yourself any hassle trying to solve missing header issues, install the build-essential package and gcc-multilib while you&#8217;re at it:



You should be set to compile libdispatch now. There should already be a configure file in the &#8216;libdispatch&#8217; root folder so let&#8217;s install:



Make sure you force clang as the compiler, not gcc. The end of the install message should tell you where the libs were installed, for me it was /usr/local/lib. Ubuntu rather stupidly&nbsp;<span style="text-decoration: line-through;">ignores</span> [erases](https://bugs.edge.launchpad.net/ubuntu/+source/xorg/+bug/366728) the _LD\_LIBRARY\_PATH_ variable if you set it through the shell, so add a file in /etc/ld.so.conf.d/* pointing to this location if you don&#8217;t have one already.

You should have Grand Central set up by now. Let&#8217;s write a hello world programme and see if it works:



And to compile:



And if all is well you should get the message printed to the screen. I haven&#8217;t tried anything computationally intensive yet and I wouldn&#8217;t be surprised if there&#8217;s a performance discrepancy between FreeBSD / OS X, since libkqueue is just a wrapper around epoll(). But hello world works, and that&#8217;s half the battle, right?