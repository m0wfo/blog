---
id: 649
title: On interviewing
date: 2017-03-09T00:02:29+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=649
permalink: /2017/03/09/on-interviewing/
categories:
  - Uncategorized
---
I drafted this prior to the present _&#8220;Hi, I&#8217;m x but can&#8217;t do y on a whiteboard&#8221;_ thing, but it now seems especially prescient. Go Prescience!

<blockquote class="twitter-tweet" data-lang="en">
  <p dir="ltr" lang="en">
    Hello, my name is David. I would fail to write bubble sort on a whiteboard. I look code up on the internet all the time. I don&#8217;t do riddles.
  </p>
  
  <p>
    — DHH (@dhh) <a href="https://twitter.com/dhh/status/834146806594433025">February 21, 2017</a>
  </p>
</blockquote>



## tl; dr

It&#8217;s a bit more nuanced than that tweet, but this writer thinks you should know, if not have completely instant recall of _some_ stuff. The programming interview is _partially_ broken, but with a bit of discipline and transparency on the part of interviewers, we can make it better. This one won&#8217;t be &#8216;finished&#8217;; it&#8217;s something I&#8217;ll keep polishing, because that&#8217;s the nature of tech interviews.

## ?

I wonder how many hits the Wikipedia bubble sort page got after that. Anyway&#8230; I&#8217;ve worked in a few shops now, and I&#8217;m almost embarrassed to say how many tech interviews I&#8217;ve been in, either as an interlocutor or a witness where the questions posed are worryingly inconsistent or the rush to give everyone a feel for _candidate x_ spooks them. Due to current restrictions on my freedom to write, these technical rants now cost me two laywers&#8217; time in London / NY to make sure I&#8217;m on solid legal ground and as such, I&#8217;m not going to divulge any particular techniques or habits used when testing candidates for technical roles. What I can say, borne out of the frustration of failed interview loops is what it would do you no harm to learn, regardless of the tech role you&#8217;re seeking. It&#8217;s a much less exhaustive list than you think; the difference is you have to want it / give a shit. But if you _really do_ want to jump into an engineering gig, then go for it (and keep trying to go for it if it doesn&#8217;t work out first time round); life&#8217;s too damn short to wonder _what if_. This industry, laden with 20-something WASPish males is crying out for people to stop wondering _what if_. Alright Christophe, what should I know?

## Big- Oh*

I&#8217;m not sadistic enough to ask for proofs on a whiteboard, but one of the things about operating at Web Scale is that you become resource-constrained again. One of the responsibilities that come with this interesting challenge is being cognisant of how much time and/or space something takes up when a million people are doing it, as opposed to when 5 of your best friends are doing it to fake some usage stats for a VC 😉

For example, this takes **O(1)** time:

<pre class="language-javascript"><code>var x = 0;</code>
</pre>

But this takes **O(n)** time:

<pre class="language-javascript"><code>for (var i = 0; i &lt; n; i++) {
// do something with n[i] probably
}
</code>
</pre>

e.t.c. I really like people who have a handle on how much time/space something uses at the back of their heads.

When you operate at web-scale you also have to know when something just ain&#8217;t feasible; the <a href="http://bigocheatsheet.com/" target="_blank">Big-O cheat sheet</a> will enlighten you if you&#8217;re not already sure. Basically, you can absorb most of the knowledge you need to cross the threshold of employability from a decent undergrad DS textbook (<a href="https://www.amazon.co.uk/d/cka/Algorithms-Robert-Sedgewick/032157351X/ref=sr_1_1?s=books&ie=UTF8&qid=1489009361&sr=1-1&keywords=sedgewick+algorithms" target="_blank">Sedgewick</a>, <a href="https://www.amazon.co.uk/Algorithm-Design-Manual-Steven-Skiena/dp/1848000693/ref=sr_1_1?s=books&ie=UTF8&qid=1489009398&sr=1-1&keywords=skiena" target="_blank">Skienna</a>, and of course <a href="https://www.amazon.co.uk/Art-Computer-Programming-Volumes-1-4a/dp/0321751043/ref=sr_1_1?s=books&ie=UTF8&qid=1489009414&sr=1-1&keywords=taocp" target="_blank">Knuth</a> if you&#8217;re feeling flush). It&#8217;s probably helpful to note that none of them really require more than high-school math.

## Data Structures

I bet I flunked more DS modules than you. I fucking hated them. If you have the emotional intelligence or cynicism to tell when a lecturer is blindly reciting a {McGraw Hill / Pearson / whatever} slide deck, it can be a handicap, because:

  1. said lecturer is in no way enthused, and you know it
  2. said lecturer may / may not have instant recall of many of the the structures themselves, and you can sense it

I saw through those cowboys instantly. In fact, I had so little time for them that I looked up their pay scales and figured out that my ETF trading on its own was sufficient to negate the benefits of trying to climb any academic ladder. And so I failed and then grudgingly passed&#8230;

With all that said and despite the pervasiveness of horrendous teaching, there are few utterly obvious ones you should have instant recall of, and a couple you should at least be aware of from textbooks. The mandatory, _do-not-waste-my-time-otherwise_ ones:

  * Fixed-size arrays
  * Stacks
  * Linked lists (just learn this parrot-fashion if it comes to it, somebody may ask you how to implement one an interview). **Hint:** if you look at a Lisp/Scheme implementation, they might even look elegant. The best data structures always are.
  * Maps/dictionaries/hash tables/whatever you wanna call them. How do you hash a _thing_ into a bucket and _look it up again_? If your first taste of programming was in a dynamic language like JS, Ruby, Python, e.t.c you&#8217;ve probably taken this for granted. Lots of places spend lots of time, and pay lots of people lots of money to optimise these kinds of things. Sexier now?

There are some data structures / algorithms where I wouldn&#8217;t bother rote-learning the implementation details but would take care to be aware of:

  * Structures with a total-ordering; at least be aware of something like a <a href="https://en.wikipedia.org/wiki/Red%E2%80%93black_tree" target="_blank">red-black</a> or AVL tree, even if you can&#8217;t recall how you add / remove an element from one from first principles. If you know about <a href="https://en.wikipedia.org/wiki/Skip_list" target="_blank">skip lists</a>, I probably want to hire you already.
  * Probably something about sorting but that seems to be a hardball question even for &#8216;senior&#8217; engineers these days, so relax. I would suggest you at least know merge sort, just because it introduces you to the world of theoretically <a href="https://en.wikipedia.org/wiki/Embarrassingly_parallel" target="_blank">embarrassingly parallel</a> algorithms, and it&#8217;s just very elegant. I care about elegance.
  * ANYTHING about persistent data structures (e.g. <a href="https://en.wikipedia.org/wiki/Hash_array_mapped_trie" target="_blank">HAMTs</a>, binomial heaps, any casual reference to <a href="https://www.amazon.com/Purely-Functional-Structures-Chris-Okasaki/dp/0521663504" target="_blank">Okasaki</a>) puts you ahead of 99.9% of your competition.

By the way, you should understand the time / space complexities for all the aforementioned structures; some job specs may say something about asymptotic analysis, which in practice means: _&#8220;can you tell me the <a href="https://en.wikipedia.org/wiki/Big_O_notation" target="_blank">big-O</a> for some operation?&#8221;._ While you can learn them parrot-fashion, dragging yourself through an implementation, or rolling your own will stick it in your head for longer (forever?). If somebody says, &#8220;how does x perform?&#8221; in an interview, that&#8217;s your cue.

## Bits & bytes

There are 8 bits in a byte. Just fucking learn that. Super-softball question, super easy way to stick a nail in your coffin. It&#8217;s like a free &#8216;5 bucks for turning up&#8217; kind of thing.

Bits & bytes questions are somewhat contingent on <a href="https://en.wikipedia.org/wiki/Endianness" target="_blank">byte ordering</a> and platform, e.g. 32 / 64 bit; one of the virtues of hosted languages like Java is that nowadays you can assume 64 bit memory width, everything being signed and big-endian memory addressing. Also remember all primitive Java types have a well-defined size. Whatever language you&#8217;re using, you should know the sizes of these fundamental types. So if somebody asks you how you turn the integer -123 into 123, you can confidently say it&#8217;s _**~i + 1**_. Also if somebody asks how you represent a signed number in Java, make sure you convert if necessary:

<pre class="lang-ruby"><code>TYPE x = blah;
UNSIGNED TYPE z = y & x.SIZE;</code></pre>

Also make sure you _get_ bit-shifting, because if you do, again you put yourself ahead of a bunch of people who passed this by in college. This kinda thing is crucial when you need to hash something, Let&#8217;s say you want to take the upper and lower 32 bits of a Java long:

<pre class="language-javascript"><code>
long x = ...;
int y = (int) x;
int y = (int) x &gt;&gt;&gt; 32;
</code>
</pre>

Make sense? No? First, go learn the sizes of these fundamental types (<a href="https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html" target="_blank">here they are in Java</a>), then go understand bit-shifting. Ask on SO or something- it really isn&#8217;t as terrifying as you think.

## Sexy things

Of course, they all have lovely structures. But regardless of whether you&#8217;re a seasoned pro or a grad, there are certain things that the market, given the current hardware/business/political constraints likes right now. They&#8217;re not necessarily significantly more complex than the stuff you&#8217;ll learn in a rigorous CS course, but the market just places more value on them right now:

### Probabilistic data structures

That&#8217;s a frightening way of describing something like:

  * Skip lists
  * Bloom filters
  * Count-min-sketch

Scary names, but they&#8217;re a) beautiful and b) likely to scare the shit out of the bottom 50% of interviewers- in which case you saved yourself a lot of time and worked out you&#8217;re too good for that place anywho.

### Async <ANYTHING!>

Yeah, really. But the problem with async something is, unless you&#8217;re working for a startup or agency which harbours perpetual greenfield projects, it typically has to fit in with a big investment in language / framework X.

One common question I ask when I see a node-heavy CV is, &#8220;when **wouldn&#8217;t** you use node?&#8221;, trying to imply a differentiation between certain workloads (you see how, apart from gut-feeling, this may pass you by unless you&#8217;re comfortable with Big-O?). I don&#8217;t think I&#8217;ve really had a decent solid answer from any candidates. The glib answer is to <a href="https://blog.logentries.com/2016/07/what-exactly-is-an-event-loop/" target="_blank">go here</a> and read another of my diatribes. The more sane suggestion is to get your head around an event notification system like <a href="https://en.wikipedia.org/wiki/Epoll" target="_blank">epoll</a> and understand how it differs from its predecessors. The even less instructive but more reliable suggestion is to develop a large-scale system using a nonblocking framework like node from the ground-up and see what problems you run into. If you&#8217;ve done that, expect some questions about the above from a decent place. If they just quiz you on the latest trends on npm then you are not working with top-tier developers.

## More?

For the most part, the ideal tech interview should be reassuringly boring but reflect the fact that while constraints and technologies change, the underlying maths and CS don&#8217;t (much). If you&#8217;re being hit with a tonne of riddles or things which were open computer-science problems for long periods of time (i.e. with non-obvious ways of decomposing the problem), don&#8217;t feel bad- the interviewer probably just wanted to make themselves feel smart. Things like detecting cycles in linked lists fall into this category.

Please also understand a huge caveat with this shopping list of CS topics- I believe improving your grasp of these things will multiply your chances of acing a tech interview but algorithmic ability and problem decomposition are _necessary but not sufficient_ prerequisites for becoming a great programmer; it&#8217;s unlikely, for example, that you&#8217;ll go through an interview loop where your solid whiteboard implementation of a linked list is undone by an admission that you&#8217;ve never used something like LISP or ML in anger, even though in my experience such programmers tend to produce qualitatively better code in more mundane but commonplace languages like Java or Python. I still think that an understanding of things like parsers, compilers, consensus algorithms <a href="https://www.joelonsoftware.com/2005/12/29/the-perils-of-javaschools-2/" target="_blank">plus FP and pointers</a> underpin what mostly qualifies as _great_ software, open or otherwise. But that&#8217;s for another post or ten&#8230;

This is hopefully something of a living document, so I&#8217;ll add/amend bits as necessary.

* * *

&nbsp;

*not that one. Grow up 😉