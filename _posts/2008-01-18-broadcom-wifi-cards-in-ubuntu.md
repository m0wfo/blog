---
id: 527
title: Broadcom wifi cards in Ubuntu
date: 2008-01-18T21:49:57+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=527
permalink: /2008/01/18/broadcom-wifi-cards-in-ubuntu/
categories:
  - Uncategorized
---
Quick tip for people who have dist-upgraded their Debian-flavour distro and suddenly found that wpa-supplicant doesn&#8217;t work: you can now remove ndiswrapper, because native drivers are here at last:

<div class="CodeRay">
  <div class="code">
    <pre>sudo rmmod ndiswrapper</pre>
  </div>
</div>

Don&#8217;t forget to remove ndiswrapper from the list of modules that load at startup as well.