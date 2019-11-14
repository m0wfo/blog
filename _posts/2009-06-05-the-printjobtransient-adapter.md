---
id: 582
title: The PrintJob::Transient adapter
date: 2009-06-05T22:09:13+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=582
permalink: /2009/06/05/the-printjobtransient-adapter/
categories:
  - Uncategorized
---
One requirement I&#8217;ve had for a couple of projects recently is some kind of adapter allowing me to stream data straight to print without having to touch the file system.

Cups now has a [Transient](http://cups.rubyforge.org/doc/classes/Cups/PrintJob/Transient.html "Transient RDoc") subclass of PrintJob which takes care of this tedium for us. Let&#8217;s use Prawn to illustrate the benefit:



I toyed with the idea of a proper adapter pattern, a C singleton method and this subclass. I&#8217;m not sure this is a perfect implementation, but developmentally it seems like the path of least resistance.