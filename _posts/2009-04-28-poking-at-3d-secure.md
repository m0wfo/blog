---
id: 563
title: Poking at 3D Secure
date: 2009-04-28T22:03:10+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=563
permalink: /2009/04/28/poking-at-3d-secure/
categories:
  - Uncategorized
---
The list of andrological profanities which would adequately describe the fucktard(s) who devised the [3D Secure protocol](http://en.wikipedia.org/wiki/Verified_by_Visa "3D Secure wiki entry") is vast. But nonetheless, it&#8217;s going to become more prolific [read: mandatory].

I spent a few hours looking throught sample implementations as well as [people who had extended](http://github.com/tekin/active_merchant/tree/3dsecure) activemerchant. But this, and some opaque high-level documentation wasn&#8217;t really giving me much insight into how you&#8217;d work 3D Secure into an existing app, perhaps one using an antiquated version of activemerchant or a hand-rolled solution.

As a preamble to all of the above, I&#8217;ll ignore activemerchant, rails, etc and just start playing about with the constituent parts in ruby. Remember a_m is just a convenient way of abstracting all of those calls to payment gateways that you don&#8217;t really want in your models. **It&#8217;s just a specialised http client**.

With that in mind, let&#8217;s forget about abstracting stuff and just make a call to a gateway (I&#8217;m using Sage Pay*, but the general flow is universal):



Here I passed it the PaRes and MD strings that the gateway gave me when I posted the customer&#8217;s credit card details. You&#8217;d do this after they&#8217;ve successfully completed the 3D Secure validation. The response here should return the status &#8216;OK&#8217;, which completes the transaction (update your ActiveRecord object!).

The [Net::HTTP library](http://www.ruby-doc.org/stdlib/libdoc/net/http/rdoc/classes/Net/HTTP.html) is a great alternative to cURL for making secure http requests and I found it easier to probe gateway behaviour with this than debugging a_m.

Just remember, all you&#8217;ve doing is working with a key pair (**PaReq & MD** or something similar), passing them:

  1. From to issuer to the app
  2. From the app to the card issuer
  3. From the card issuer back to the app
  4. From the app to your gateway
  5. From the gateway back to your app again

If you can grab their values (they&#8217;re immutable AFAIK) and poke the gateway independently with a quick & dirty script, you&#8217;ll probably pick up the principle quicker than by reading 20 pages of documentation interspersed with ASP examples.

_* Sage Pay (what used to be called Protx have changed all their urls. Details [here](http://www.sagepay.com/sagepay_developers_changing_urls.html))._