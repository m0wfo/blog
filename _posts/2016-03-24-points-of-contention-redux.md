---
id: 94
title: Points of @Contention, Redux
date: 2016-03-24T08:15:52+00:00
author: chris
layout: post
guid: https://chris.mowforth.com/?p=94
permalink: /2016/03/24/points-of-contention-redux/
categories:
  - Uncategorized
---
I recently <a href="https://blog.logentries.com/2016/03/a-point-of-contention-cache-coherence-on-the-jvm/" target="_blank">published a post over</a> on the <a href="https://logentries.com/" target="_blank">Logentries</a> blog which outlined the phenomenon of false-sharing, and ways you can work around it on the JVM. I was surprised to see it was so widely read, so I felt I should provide a more rigorous benchmark- this time using <a href="http://openjdk.java.net/projects/code-tools/jmh/" target="_blank">JMH</a>. Head over <a href="https://github.com/m0wfo/false-sharing-demo/blob/master/src/main/java/com/logentries/blog/JMHBenchmark.java" target="_blank">here</a> if you want to check it out; suffice to say the results are equally clear.

Somebody in the office who is far smarter than me subsequently asked the question:

> Doesn&#8217;t volatile always reach out to main memory?

If that&#8217;s case, it seems a bit irrelevant discussing how to avoid on-die cache misses, right? I didn&#8217;t have a ready answer for this, so when confronted with seemingly obvious [but actually quite subtle] things like this, dig out the Java Memory Model spec (link to PDF <a href="http://www.cs.umd.edu/~pugh/java/memoryModel/jsr133.pdf" target="_blank">here</a>. If anybody can find an HTML version, that&#8217;d be cool). One major reason for using volatiles in the example is just straight correctness: chapter 12, _&#8220;Non-atomic Treatment of double and long&#8221;_, makes this clear:

> Java<sup>TM</sup> virtual machines are free to perform writes to long and double values atomically or in two parts

So to guarantee that a reading thread sees an atomic [albeit potentially inconsistent] view of a data type bigger than 32 bits, it&#8217;s gotta be volatile. To be clear, when I say _atomic_, I mean the JMM guarantees that a volatile type will not get into a condition where its value consists of 32 bits of data from one thread and 32 from another. It can be overwritten by multiple other threads, (the inconsistency bit) but as a programmer you can be sure it won&#8217;t be in some crappy half-way state. To be fair, if the working example used an int, byte, e.t.c, this would be a moot point.

The next motivation is visibility- without the volatile modifier, there&#8217;s no way of communicating a value change between threads. As long as _only one_ thread is writing to a given variable, all the readers will see its value correctly. According to the gospel of chapter 3:

> A write to a volatile field happens-before every subsequent read of that volatile.

The JMM refers to the coordination points between threads (like a volatile write) as _happens-before_ relationships. When people talk about memory consistency and instruction ordering in Java, you hear that term a lot 😉

### Answer the question already!

Alright, does a volatile read always reach out to DRAM? This is where things get a bit murky [read: architecture dependent] but for my own edification we can look into the assembly code that HotSpot generates to see what&#8217;s going on. For starters, this means cracking out this little incantation:

<pre>java -XX:+UnlockDiagnosticVMOptions-XX:CompileCommand=print,*YourClass.andMethod YourMainClass</pre>

For that to work, however, you need Java 7 or later and a dynamic library called <a href="https://kenai.com/projects/base-hsdis" target="_blank">hsdis</a>. The Mac _.dylib_ can be found <a href="https://kenai.com/projects/base-hsdis/downloads/directory/gnu-versions" target="_blank">here</a>; drop it into your **$JAVA_HOME/jre/lib** and you&#8217;re all set. There&#8217;s <a href="https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html" target="_blank">detailed documentation</a> of all the CompileCommand variants, if you&#8217;re interested (although it&#8217;s omitted from the Oracle java manpage).

Anyway, let&#8217;s say we have a class like this where for some bizarre reason we want to continuously read a variable and eventually print it without updating:

<pre><code class="language-javascript">public class NonVolatileClass implements Runnable {

    public long a;

    @Override
    public void run() {
        long b;
        for (int i = 0; i &lt; 1E9; i++) {
            b = a;
        }
        System.out.println(a);
    }
}</code></pre>

Before you ask: the loop isn't just there to spice up this otherwise pointless example- it ensures we hit the _b = a_ path often enough for HotSpot to compile it on the fly. Otherwise we don't have any generated native code to observe. The pertinent assembly for the above looks like this:

<pre><code class="language-c-like">  0x0000000104287223: jmpq   0x0000000104287276  ;*iload_3
                                                ; - com.logentries.blog.NonVolatileClass::run@2 (line 13)

  0x0000000104287228: mov    0x10(%rsi),%rdx    ;*getfield a
                                                ; - com.logentries.blog.NonVolatileClass::run@12 (line 14)
                                                ; implicit exception: dispatches to 0x00000001042873fe
  0x000000010428722c: inc    %edi</code></pre>

Ok- no surprises here; it's a straight _mov_ operation into a general-purpose register. Kind of anticlimactic since on a 64-bit platform it can be performed as 1 instruction. Don't count on this, though (cough, ARM, Android). What happens if we make _a_ volatile?

<pre><code class="language-c-like">  0x000000010d6895e3: jmpq   0x000000010d68963c  ;*iload_3
                                                ; - com.logentries.blog.VolatileClass::run@2 (line 13)

  0x000000010d6895e8: vmovsd 0x10(%rsi),%xmm0   ; implicit exception: dispatches to 0x000000010d6897ce
  0x000000010d6895ed: vmovq  %xmm0,%rdx         ;*getfield a
                                                ; - com.logentries.blog.VolatileClass::run@12 (line 14)

  0x000000010d6895f2: inc    %edi</code></pre>

Right- in order to honour the JMM-specified behaviour, the JVM pulls _a_ from memory into an MMX register using the VMOVSD instruction that was introduced with <a href="https://software.intel.com/en-us/articles/introduction-to-intel-advanced-vector-extensions" target="_blank">AVX</a> before it's moved to a general-purpose register. That's not too much worse than a regular read, and if it sits in L1/2/3 cache and doesn't get written, it's just as cheap. Things get a bit more funky if we start updating it though:

<pre><code class="language-c-like">  0x000000010be89ca3: jmpq   0x000000010be89d1e  ;*iload_1
                                                ; - com.logentries.blog.VolatileClass::run@2 (line 12)

  0x000000010be89ca8: vmovsd 0x10(%rsi),%xmm0   ; implicit exception: dispatches to 0x000000010be89eae
  0x000000010be89cad: vmovq  %xmm0,%rdx         ;*getfield a
                                                ; - com.logentries.blog.VolatileClass::run@13 (line 13)

  0x000000010be89cb2: movabs $0x1,%r10
  0x000000010be89cbc: add    %r10,%rdx
  0x000000010be89cbf: mov    %rdx,0x40(%rsp)
  0x000000010be89cc4: vmovsd 0x40(%rsp),%xmm0
  0x000000010be89cca: vmovsd %xmm0,0x10(%rsi)
  0x000000010be89ccf: lock addl $0x0,(%rsp)     ;*putfield a
                                                ; - com.logentries.blog.VolatileClass::run@18 (line 13)

  0x000000010be89cd4: inc    %edi</code></pre>

The first bit should look familiar from the volatile read, but is that a lock at the end? Normally our fancy CPU is free to reorder instructions as it sees fit, but to make our _happens-before_ invariant hold true, the JVM needs to ensure that this can't happen. Adding an x86 <a href="http://x86.renejeschke.de/html/file_module_x86_id_159.html" target="_blank">LOCK</a> operation to the last copy operation ensures that next time a read happens it sees this view of the world. This also has the effect of dirtying any necessary cache regions- did I ever mention false-sharing? If we had another unfortunate volatile variable in the same cache-line, it'd be evicted even if it never got updated. Rough justice, eh?

### Stop showing me generated assembly

Alright, that was a bit intense; to sum up: on x86-64, a volatile read can avoid the penalty of a full reach out to main memory assuming it's not being updated and the cache is not heavily contended. But you might also be surprised by how HotSpot implements the JMM's atomicity guarantees!