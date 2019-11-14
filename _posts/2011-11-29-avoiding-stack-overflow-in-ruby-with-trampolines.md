---
id: 481
title: Avoiding stack overflow in Ruby with trampolines
date: 2011-11-29T21:22:54+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=481
permalink: /2011/11/29/avoiding-stack-overflow-in-ruby-with-trampolines/
categories:
  - Uncategorized
---
Languages that don&#8217;t support tail-call elimination out of the box can make people think twice about using recursion. Ever received a&nbsp;**SystemStackError** when doing something like this?



Ok, the example&#8217;s contrived, but you&#8217;ve no doubt come across cases where you&#8217;d like to avoid using an iterative solution even if it&#8217;s just for the sake of elegance. Luckily in Ruby this can be easily fixed by:

  1. Having your method return a thunk rather than a direct tail-call
  2. Using a [trampoline](http://en.wikipedia.org/wiki/Trampoline_(computers)#High_Level_Programming) to avoid growing the stack

A thunk is essentially just the suspended application of a function. Taking our example, it&#8217;s just a matter of transforming the tail call like so:



The trampoline implementation is equally trivial. It takes a thunk as an argument and iteratively calls it until something other than a continuation is returned:



So now if we start the ball rolling with our trampoline call:



We can run _increment_ as long as we like without blowing the stack. As there&#8217;s no base case it&#8217;ll continue indefinitely- a worst-case example. Using continuation-passing here trades a time-penalty in higher-order function calls for the advantage of not increasing the amount of space needed to perform the computation. Not too hard really, is it?

Alternatively we could use&nbsp;**[Kernel#callcc](http://ruby-doc.org/docs/ProgrammingRuby/html/ref_m_kernel.html#Kernel.callcc)** to replay the computation until we have a return value. Here we get a continuation object to use with callcc { &#8230; }, give @k the correct execution context by reassigning the thunk as before, then call it until we&#8217;re done:



If you want to get a proper grasp of CPS you could do a lot worse than get hold of [Dybvig&#8217;s book](http://books.google.ie/books?id=wftS4tj4XFMC&pg=PA104&lpg=PA104&dq=dybvig+call/cc&source=bl&ots=BgIe-Kt_Sl&sig=Vvs6QSDnfdwdAXw8kgveUeY_NHc&hl=en&ei=8zTVTq75EoPChAef4Plq&sa=X&oi=book_result&ct=result&resnum=1&ved=0CBoQ6AEwAA#v=onepage&q&f=false) and go through all the _call/cc_ examples.