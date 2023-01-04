---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2017-09-15 17:37:03'
layout: post
original_url: https://www.gaborsamu.com/blog/benchmarking_macchiatobin/
slug: cool-and-quiet-benchmarking-on-macchiatobin-armada-8040-
title: Cool and quiet benchmarking on MACCHIATObin (Armada 8040)
---

<p>I&rsquo;ve recently taken delivery of a few new goodies to complement the MACCHIATObin Arm v8 powered board that I&rsquo;ve written about recently on my blog.</p>

<ul>
<li><a href="https://www.gaborsamu.com/blog/spectrumlsf_armv8/">Standing up a IBM Spectrum LSF Community Edition cluster on Arm v8</a></li>
<li><a href="https://www.gaborsamu.com/blog/turning_up_heat_armv8/">Turning up the heat&hellip;on my Armada 8040</a></li>
</ul>
<p>Youi'll recall that my efforts to do some rudimentary testing including running HPL were thwarted by overheating.  So I decided to
address the issue with some parts I&rsquo;ve been meaning to pickup anyway for some other interesting projects I have in the pipeline
(fingers crossed):</p>

<ul>
<li>1 x <a href="https://noctua.at/en/products/product-lines/line-industrial">Noctua NF-A14 cooling fan</a></li>
<li>1 x <a href="http://nofancomputer.com/eng/products/P-500A.php">NOFAN P-500A fanless power supply</a></li>
<li>1 x (red) <a href="https://openbenchtable.com/">Open benchtable</a></li>
</ul>
<p>And this is what is looks like now&hellip;</p>

<figure><img src="https://www.gaborsamu.com/images/red_armada.jpg" />
</figure>

<p>Now, the red workbench and shiny heatsinks scream performance.  So what about my run of HPL (Linpack)?  Well, I decided to start over
from scratch and built my own Linpack against ATLAS, which I also compiled from scratch (let that run overnight).</p>

<p>The result?  I went from hitting the thermal limiter (and a non-result) to a successful Linpack run - with the CPU temperature never
really going much past 50C. As for my Linpack score, you can see that below.</p>

<figure><img src="https://www.gaborsamu.com/images/linpack_17gflops.png" />
</figure>