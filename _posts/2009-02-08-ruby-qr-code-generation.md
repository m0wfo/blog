---
id: 578
title: Ruby QR Code Generation
date: 2009-02-08T22:07:20+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=578
permalink: /2009/02/08/ruby-qr-code-generation/
categories:
  - Uncategorized
---
2D barcode standards allow relatively large amounts of information (~7000 digits for a [QR Code](http://en.wikipedia.org/wiki/QR_Code "QR wiki")) in a small footprint. In addition, their unit cost is negligible compared to RFID tags so instead of just gracing the exterior of foodstuffs and UPS deliveries, they&#8217;re finding a host of wider applications in supply-chain management, advertising, e.t.c.

<div class='p_embed p_image_embed'>
  <img alt="Media_httpuploadwikim_qgbel" height="185" src="http://getfile0.posterous.com/getfile/files.posterous.com/import-kppb/hgJBmlCpurhGqalygdjrknFcsAjAJgfyrgEhFiwExwyAsdGljogzdAobqkGF/media_httpuploadwikim_qGbEl.png.scaled500.png" width="185" />
</div>

I recently had to develop an internal letter tracking system for a client, so thinking QR Codes would be a cheap and quick solution to the problem of tracking real-world items, I started investigating Ruby implementations.

To start with, there&#8217;s a [ruby port](http://www.swetake.com/qr/) of [libqrencode](http://megaui.net/fukuchi/works/qrencode/index.en.html) and the [rQRcode](http://rqrcode.rubyforge.org/) gem which generates a text version of the code. The rQRcode gem generated 100 tags in approximately 50 seconds, not really quick enough to use in production.

My stopgap solution was to compile libqrencode:



and then just issue a system call in ruby when needed:



Generating 100 tags took just over 2 seconds this way. The system being developed just required a timely way of being able to pull the correct tag off the file system, so a 50x speedup was sufficient.

I may well post more on this in the near future; From what I&#8217;ve RTFM&#8217;ed it would be fairly trivial to write a ruby extension based on _qrencode.h._