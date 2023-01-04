---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2021-06-16 00:57:31'
layout: post
original_url: https://www.gaborsamu.com/blog/risque_computing/
slug: very-risqué-computing
title: Very risqué computing
---

<p>This spring, we’ve been blessed with fantastic and almost tropical weather here
in Southern Ontario, Canada. Normally at this time, after a long winter the last
thing on my mind are indoor activities. However on June 3rd, I was greeted one
morning by an email about an incoming delivery. It turns out it was on of the
items I&rsquo;ve been waiting patiently for from [Crowd Supply](<a href="https://www.crowdsuppl">https://www.crowdsuppl</a>
y.com/) in the hopes of keeping me busy during what I thought would be a cold
spring season.</p>

<p><strong>Christmas in June</strong></p>

<p>As I have multiple things from Crowd Supply on order (don&rsquo;t ask!) I didn&rsquo;t quite
know which item was arriving. It turns out it was the long awaited <a href="https://www.sifive.com/boards/hifive-unmatched">SiFive
HiFive Unmatched</a> RISCV powered
board. Those who know me (and I&rsquo;ve said this many times) understand that I don&rsquo;t
like mainstream anything. And that also applies to computers. My interest in Arm
based systems dates from the 1990&rsquo;s with the venerable Acorn Archimedes
computers. However, all of the news around the RISCV community has really piqued
my interest. I passed on the SiFive Unleashed primarily because it didn&rsquo;t have a
PCIe slot - although this was remedied with an optional, but costly add-on
board.</p>

<p>So when the SiFive Unmatched was announced with a competitive price and a bump
to 16GB I jumped at the opportunity to purchase one. And it turned out to be a
great decision.</p>

<p>The HiFive Unmatched is based on the SiFive Freedom U740 SOC with four U74 cores
and one S7 core and features an all important PCIe slot. With 16GB of onboard
RAM and a M.2 Key M for an SSD, my goal was to get the Unmatched setup as a desk
top. For those looking to learn more about RISCV, I&rsquo;d recommend starting with
the RISCV International foundation <a href="https://riscv.org/">site</a>. As per the RISCV
site, &ldquo;RISC-V is a free and open ISA enabling a new era of processor innovation
through open standard collaboration.&rdquo; In simple terms, the ISA or instruction
set architecture defines the set of instructions that are supported by the
processor – so things like arithmetic, logic, and branch instructions to name
a few. So it’s the way that programmers can issue commands to the processor to
do “things”.</p>

<p><strong>First impressions</strong></p>

<p>I’ve become accustomed to developer boards being packaged in rather non-descript
packaging. The first impression of the Unmatched board could not be further from
this. The board was shipped in a lovely box and included an SD card with a
bootable Freedom U SDK image and I/O shield and a USB cable. So the first
impression for me was quite positive.</p>

<figure><img src="https://www.gaborsamu.com/images/unmatched_collage.jpg" />
</figure>

<p><strong>Bootstrapping</strong></p>

<p>I mounted the Unmatched board to my Streacom BC1 benchmark table and installed
a XFX Radeon 2GB Heatsink edition to the PCIe slot. It’s an old GPU, but fanless
– which I always appreciate. Plus, I’m not looking to do any serious gaming on
the system.</p>

<p>The first boot of the system from the SD card was a success (albeit a bit slow).
I monitored the boot over the serial console (minicom) from another system. The
Unmatched sprang to life and eventually booted up to a fully working XFCE
desktop. This was actually a lot smoother than what I anticipated. Once I
confirmed that everything was working as expected, I installed a Samsung 780
NVME SSD to the M.2 Key M slot and turned my focus to Ubuntu 21.04. The <a href="https://forums.sifive.com/">SiFive
Forums</a> have proven an invaluable resource to help
me get Ubuntu up and runing on the system and to make sure the board was booting
Ubuntu with a clock of 1.2 Ghz. Of course, I followed the steps to install
Ubuntu to the NVME onboard, so I/O performance is much better now naturally.</p>

<p><strong>Burning in</strong></p>

<p>Does it run Linpack?  Of course it does :) As with any new board I receive,
running a High Performance Linpack benchmark is often one of the first things I
do. It’s a well known bechmark which provides data for the Top500 ranking of
supercomputers.</p>

<p>I used the current <a href="https://www.netlib.org/benchmark/hpl/">HPL v2.3</a> and
compiled it using the Ubuntu supplied gcc, openmpi and math libraries.
A few runs of HPL yielded a result of <em>2 GFlops</em> (see screenshots below).
Although I&rsquo;ve not looked closely at what the theoretical peak of the U740 SOC
is, the result is roughly what I expected given what I&rsquo;ve been reading up on
the board. Ultimately, I was pleased that HPL compiled and ran to completion and it was a great way to stress the board.</p>

<figure><img src="https://www.gaborsamu.com/images/linpack_collage.jpg" />
</figure>

<p>Stay tuned to this channel for more risqué computing escapades&hellip;</p>