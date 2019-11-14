---
id: 786
title: Bloom filters
date: 2018-11-22T23:19:06+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=786
permalink: /2018/11/22/bloom-filters/
categories:
  - Uncategorized
---
For some reason the subject of Bloom Filters has come up a few times recently in conversations and if you&#8217;re not solid on them, they&#8217;re definitely worth exploring both for their simplicity and because they are foundational elements in many PB-sized data stores.

## Alright, what are they?

The really really short answer is that they&#8217;re a fast, space-efficient way of asking whether a thing can be found within a huge set of other things. A Bloom Filter can reply with two answers:

  1. Maybe
  2. No, definitely not

If you need to test for set membership where a set may consist of millions/billions/trillions+ of entries, they&#8217;re the go-to data structure. If you use some kind of NoSQL database, you&#8217;re probably leveraging them already.

## How do they do that?

A Bloom filter, named after Burton Howard Bloom, a seemingly publicity-shy MIT grad came up with the idea that hashing could be harnessed as a means not just of testing identity, but also set membership [i.e. can I find object _x_ in collection _y_?] at arbitrary scale.

Ok, so let&#8217;s agree that a hash function takes a bunch of data and produces some number. Let&#8217;s also agree, for the purposes of this discussion that the function is <a href="https://en.wikipedia.org/wiki/Deterministic_algorithm" target="_blank">deterministic</a>, otherwise we&#8217;ll be here all day. Most languages provide a facility for identifying primitives based on a hash, for example in Java:

<pre>"Hello".hashCode(); // will return 6960965</pre>

A Bloom Filter builds on this by fitting the output to a slot in a fixed-size bitmap, initially all zeroed out. For example, let&#8217;s say it has a size of 8*:

[<img class="alignnone size-full wp-image-807" src="https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2018/11/empty-bitmask.png" alt="empty-bitmask" width="770" height="146" />](https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2018/11/empty-bitmask.png)

modd&#8217;ing the hash value (690965 % 8) says we should flip the bit at index 5:

[<img class="alignnone size-full wp-image-809" src="https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2018/11/1-bit-flipped.png" alt="1-bit-flipped" width="794" height="148" />](https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2018/11/1-bit-flipped.png)

What we have so far in essence is a crappy hash table- a Bloom Filter builds on this by using multiple hash functions. Without this our bitmap would quickly fill up regardless of size and the collisions would render membership tests useless.

Imagine then that we have 2 more hash functions that we modded and, for our _&#8220;Hello&#8221;_ string, flipped bits in indices 2 and 7:

[<img class="alignnone size-full wp-image-811" src="https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2018/11/fully-hashed.png" alt="fully-hashed" width="728" height="382" />](https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2018/11/fully-hashed.png)

Devising good quality hash functions is a black art and also potentially expensive depending on what kind of objects you&#8217;re hashing. Commonly used implementations such as Google&#8217;s <a href="https://github.com/google/guava/wiki/HashingExplained#bloomfilter" target="_blank">Guava</a> take an idea from a <a href="https://www.eecs.harvard.edu/~michaelm/postscripts/rsa2008.pdf" target="_blank">paper by Kirsch and Mitzenmacher</a> showing that taking a pair of positive function outputs and repeatedly multiplying them by a loop variable introduces enough entropy to do away with a bunch of unique implementations. There&#8217;s a little more to it but the core code is <a href="https://github.com/google/guava/blob/0e596412953f7f9f07070b64b63f0d38bb263bc8/android/guava/src/com/google/common/hash/BloomFilterStrategies.java#L54" target="_blank">here</a>.

## The Test

We&#8217;re pretty much done- I was trying to build up to a great climactic dénouement but frankly, my writing style is too shoddy and Bloom Filters are too simple for that<img src="https://chris.mowforth.com/wp-includes/images/smilies/frownie.png" alt=":(" class="wp-smiley" style="height: 1em; max-height: 1em;" /> 

There&#8217;s one last thing- how do we test for membership? We just put our object through the hash functions again. If all the bits at the corresponding indices are 1s, it&#8217;s a _**maybe**_. If any of them are still zeroed, it&#8217;s a _**no**_.

So if we see _&#8220;Hello&#8221;_ again:

[<img class="alignnone size-full wp-image-817" src="https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2018/11/present.png" alt="present" width="734" height="546" />](https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2018/11/present.png)

&#8230;we&#8217;re in business. Maybe. If we see &#8220;Goodbye&#8221;, and we don&#8217;t hit all ones:

[<img class="alignnone size-full wp-image-818" src="https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2018/11/not-present.png" alt="not-present" width="732" height="552" />](https://d2ub5d3l6xuomp.cloudfront.net/wp-content/uploads/2018/11/not-present.png)

&#8230;no dice.

And that really is it; the most elegant data structures are always simple. Wikipedia has some useful material on <a href="https://en.wikipedia.org/wiki/Bloom_filter#Probability_of_false_positives" target="_blank">false positive probabilities</a> if you are in the position of sizing one of these things. If you were just curious, hopefully you can see how Bloom Filters are a cheap and effective way of reducing costly lookups on main data stores- or even a probabilistic replacement for them in the case where data is infinite or can&#8217;t be replayed.

* * *

*I chose a power of 2 because many commercial implementations do this. **_x % y_** is the same as **_x & (y &#8211; 1)_** where **_y_** is a power of 2, and a single logical AND is much cheaper. The modulo operation is on a hot path, being called for every hash function in the filter; filters themselves are often called many 000&#8242; of times a second in place of brute-forcing the backing data store, so small constants like this add up.