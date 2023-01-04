---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2015-02-16 14:51:20'
layout: post
original_url: https://www.gaborsamu.com/blog/workpadz50_netbsd/
slug: ibm-workpad-z50-netbsd-an-interesting-combination
title: IBM Workpad z50 & NetBSD - an interesting combination
---

<p>This week we look at another RISC powered notebook, this time from IBM.<br />
Although IBM did produce a line of PowerPC based Thinkpad systems, this blog
is focused on a little known system called the IBM Workpad z50. This Microsoft
Handheld PC form factor system was launched in March 1999 and ran Windows CE
at the time. As we’ll see below, with some ingenuity it is also able to run
NetBSD, which makes it a much more interesting proposition (at least for me).
Ironically, although this is a high performance computing (HPC) focused blog,
the “HPC” in this case stands for “Handheld PC”.</p>

<p>The Workpad z50 has a form factor smaller than a notebook, but has what I
consider to be an excellent keyboard and of course the trademark Thinkpad
trackpoint!  Looking more closely at the specifications:</p>

<ul>
<li>NEC VR4121 MIPS R4100 CPU @ 131 MHz</li>
<li>16 MB System RAM (expandable)</li>
<li>16 MB System ROM</li>
<li>8.4” LCD Display 640x480 (16-bit)</li>
<li>External Monitor connector (SVGA)</li>
<li>Serial port</li>
<li>Infrared port</li>
<li>CF slot</li>
<li>PCMCIA slot</li>
</ul>
<p>What prevents me from taking my pristine Workpad z50 to the local electronics
recycling facility is NetBSD. With a little effort it is possible to install
recent versions of NetBSD on the Workpad z50 and even run XWindows. There are
a number of sources of information on this topic including some videos on
YouTube which helped me a great deal:</p>


<div style="padding-bottom: 56.25%; height: 0; overflow: hidden;">
  
</div>


<p>I won’t run through the install procedure here as that’s been well covered
already in the above series of videos. Rather, let’s look at the boot-up
sequence and of course in keeping with the high performance computing theme,
run a simple benchmark. Links to the videos follow below:</p>

<p><strong>The requisite system bootup</strong>

<div style="padding-bottom: 56.25%; height: 0; overflow: hidden;">
  
</div>

</p>

<p><strong>Starting XWindows and running Linpack</strong>

<div style="padding-bottom: 56.25%; height: 0; overflow: hidden;">
  
</div>

</p>

<p>Using NetBSD pkgsrc, I have setup NetBSD on a x86 based system and have taken
advantage of distcc to cross compile binaries. This helps greatly to get
packages quickly compiled for the system. Note that I ran into a log of local
compiles failing due to lack of RAM.  So cross compiling is almost a must.</p>

<p>Equipped with PCMCIA, I’m able to easily add to the Workpad z50 such
capabilities as Ethernet, Wireless networking and even SCSI.  Below is my
collection of PCMCIA adaptors.</p>

<figure><img src="https://www.gaborsamu.com/images/pcmcia.jpg" />
</figure>

<p>Next steps?  I&rsquo;ll be looking to move to NetBSD 6.x series and compile a more
compact kernel (with drivers removed that I don&rsquo;t require). And unlike the
system in my previous blog, this one is silent :)</p>