---
id: 559
title: 'OS X- Specific Shell Tools'
date: 2009-02-04T22:01:02+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=559
permalink: /2009/02/04/os-x-specific-shell-tools/
categories:
  - Uncategorized
---
One perk of working on a mac over other *nix variants is the tight integration with many of the higher-level os-specific features that make apple machines so nice to use. Here are a few you might not be aware of:

**open**

Pretty much does what it says on the tin. Pass it a file or directory. It&#8217;ll either open with the default application, or open up a finder window for the latter.

**mdfind**

Hit the spotlight index through the command line. I have an alias for _&#8220;mdfind &#8212;onlyin . $1&#8221;_ for finding, say, a method in a big ruby project I don&#8217;t know very well. It&#8217;s a hell of a lot quicker than textmates&#8217; command+shift+f search which freezes with alarming regularity.

**say &#8220;something&#8221;**

Text-to-speech tool. Pass it a string, and your wise words are reiterated. Use this while ssh&#8217;ed into a mac near you to convince the user they have Alzheimer&#8217;s. This is particularly effective when used in conjunction with _&#8220;osascript -e &#8216;set volume 25&#8217;&#8221;_ to turn their speaker volume down beforehand. For even greater impact, run it as a cronjob.

**osascript -e &#8220;script&#8221;**

Inline evaluation of AppleScript. With Apple&#8217;s [OSA](http://developer.apple.com/documentation/Carbon/Reference/Open_Scripti_Architecture/Reference/reference.html "Apple Open Scripting Architecture") you get a pretty expansive command set for controlling system behaviour, the finder, standard apps like iTunes. Try firing up the script editor utility and loading an application dictionary. I set the alias _tmreload=&#8221;osascript -e &#8216;tell app "TextMate" to reload bundles&#8217;&#8221;_ to quickly reload textmate&#8217;s bundle list. For the rubyists, there&#8217;s an Apple-sponsored [RubyOSA](http://rubyosa.rubyforge.org/ "RubyOSA") gem as well.

**pbcopy**

Pipe your output into pbcopy, then paste it into native Cocoa applications.

**opendiff file1 file2**

Fires up FileMerge, diffing the two files you passed as arguments.

**dns-sd -R &#8216;share\_name&#8217; service\_type . port extra_arguments&#8221;**

Announce a service using [mDNS](http://www.apple.com/macosx/technology/bonjour.html "Bonjour"). A list of service types is maintained [here](http://www.dns-sd.org/ServiceTypes.html "mDNS service types").

I&#8217;ve neglected to talk about things like [launchd](http://developer.apple.com/MacOsX/launchd.html "launchd")<sup>†</sup> and the [plethora of command-line tools](http://images.apple.com/server/macosx/docs/Command_Line_Admin_v10.5.pdf "leopard server") adorned on OS X Server Administrators.



_† I&#8217;m an OpenBSD purist; why complicate process management anyway?_