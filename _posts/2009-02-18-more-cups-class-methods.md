---
id: 553
title: More CUPS class methods
date: 2009-02-18T21:59:09+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=553
permalink: /2009/02/18/more-cups-class-methods/
categories:
  - Uncategorized
---
I&#8217;m separating the print job methods into the PrintJob class, and environment-specific stuff into a class called Cups. I added a first version of the job querying method _all\_jobs\_on(printer)_ this evening. It returns a hash of _{:job_id => [job, info]}_ which I shall of course elaborate on.

More info here:

[<http://github.com/cmowforth/cups/>](http://github.com/cmowforth/cups/commit/1046d17951fe51d3839ef689ff2329560c8e7cec)  
[commit/1046d17951fe51d3839ef689ff2329560c8e7cec](http://github.com/cmowforth/cups/commit/1046d17951fe51d3839ef689ff2329560c8e7cec)