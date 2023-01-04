---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2014-03-23 16:53:38'
layout: post
original_url: https://www.gaborsamu.com/blog/udoo_test/
slug: udoo-quad-test-drive
title: Udoo Quad test drive
---

<p>Here is a brief update regarding my experiences so far with the Udoo Quad
board. I call this <em>kicking the tires</em>, but it simply amounts to tinkering
with the board and getting a better understanding of it&rsquo;s capabilities.</p>

<p>My choice of OS for this round of testing is Ubuntu Studio 12.04 armHF,
which I obtained from the Udoo Community site downloads page.</p>

<p>As the Udoo Quad includes an on-board SATA connected, I followed the necessary
steps to install the OS to the external disk, and to boot from it by selecting
the appropriate device from the U-Boot environment.  I used the following <a href="https://elinux.org/UDOO_boot_from_sata">page</a> as a high-level guide.</p>

<p>The disk in this case was an older ~80GB Hitachi disk that I had in my spares
and suitable for the intended purpose.  With the system booted up, here is what we see:</p>

<div class="highlight"><pre><code class="language-bash"> root@udoo-studio-hfp:~# uname -a

Linux udoo-studio-hfp 3.0.35 <span style="color: #75715e;">#1 SMP PREEMPT Mon Dec 16 14:46:12 CET 2013 armv7l armv7l armv7l GNU/Linux</span>

root@udoo-studio-hfp:~# cat /proc/cpuinfo

Processor : ARMv7 Processor rev <span style="color: #ae81ff;">10</span> <span style="color: #f92672;">(</span>v7l<span style="color: #f92672;">)</span>
processor : <span style="color: #ae81ff;">0</span>
BogoMIPS : 1988.28

processor : <span style="color: #ae81ff;">1</span>
BogoMIPS : 1988.28

processor : <span style="color: #ae81ff;">2</span>
BogoMIPS : 1988.28

processor : <span style="color: #ae81ff;">3</span>
BogoMIPS : 1988.28

Features : swp half thumb fastmult vfp edsp neon vfpv3 

CPU implementer : 0x41
CPU architecture: <span style="color: #ae81ff;">7</span>
CPU variant : 0x2
CPU part : 0xc09
CPU revision : <span style="color: #ae81ff;">10</span>

Hardware : SECO i.Mx6 UDOO Board
Revision : <span style="color: #ae81ff;">63012</span>
Serial : <span style="color: #ae81ff;">0000000000000000</span>

root@udoo-studio-hfp:~# lsscsi

<span style="color: #f92672;">[</span>0:0:0:0<span style="color: #f92672;">]</span>    disk    ATA      Hitachi HTS54128 HP3O  /dev/sda</code></pre></div>

<p>Using the trusty <em>gnome-disk-utility</em>, the read benchmark returns the following results.  If this all looks a bit Mac OS X ish - don&rsquo;t be alarmed.  I&rsquo;m
connecting to my Udoo from my Macbook and tunneling X over ssh.  Again keep in
mind here that this is an old disk.</p>

<figure><img src="https://www.gaborsamu.com/images/udoo_sata2.png" />
</figure>

<p>I was surprised to find the the <em>cpufreq</em> utilities all worked as expected on
the system also.  By default, the system booted in a conservative mode
(~396 MHz) and with <em>cpufreq-set</em> I successfully enabled the performance governor.</p>

<div class="highlight"><pre><code class="language-bash"> root@udoo-studio-hfp:/usr/bin# ./cpufreq-info

cpufrequtils 007: cpufreq-info <span style="color: #f92672;">(</span>C<span style="color: #f92672;">)</span> Dominik Brodowski 2004-2009

Report errors and bugs to cpufreq@vger.kernel.org, please.

analyzing CPU 0:

  driver: imx

  CPUs which run at the same hardware frequency: <span style="color: #ae81ff;">0</span> <span style="color: #ae81ff;">1</span> <span style="color: #ae81ff;">2</span> <span style="color: #ae81ff;">3</span>

  CPUs which need to have their frequency coordinated by software: <span style="color: #ae81ff;">0</span> <span style="color: #ae81ff;">1</span> <span style="color: #ae81ff;">2</span> <span style="color: #ae81ff;">3</span>

  maximum transition latency: 61.0 us.

  hardware limits: <span style="color: #ae81ff;">396</span> MHz - <span style="color: #ae81ff;">996</span> MHz

  available frequency steps: <span style="color: #ae81ff;">996</span> MHz, <span style="color: #ae81ff;">792</span> MHz, <span style="color: #ae81ff;">396</span> MHz

  available cpufreq governors: interactive, conservative, ondemand, userspace, powersave, performance

  current policy: frequency should be within <span style="color: #ae81ff;">396</span> MHz and <span style="color: #ae81ff;">996</span> MHz.

                  The governor <span style="color: #e6db74;">"performance"</span> may decide which speed to use

                  within this range.

  current CPU frequency is <span style="color: #ae81ff;">996</span> MHz <span style="color: #f92672;">(</span>asserted by call to hardware<span style="color: #f92672;">)</span>.

  cpufreq stats: <span style="color: #ae81ff;">996</span> MHz:8.10%, <span style="color: #ae81ff;">792</span> MHz:0.63%, <span style="color: #ae81ff;">396</span> MHz:91.27%  <span style="color: #f92672;">(</span>172036<span style="color: #f92672;">)</span>

....</code></pre></div>

<p>As I indicated at the outset, the system has been installed with a ARM HF
prepared Linux distribution.  This implies that the distro has been compiled
with the appropriate flags to enable hardware Floating Point Unit support.<br />
Which should help us to attain better performance for applications which make
use of floating point arithmetic.</p>

<p>The system <em>readelf</em> tool can be used to interrogate a binary for architecture
information.  In this case, I&rsquo;ve installed the OS supplied HPC Challenge
package to give the board it&rsquo;s baptism into the world of Technical Computing.</p>

<div class="highlight"><pre><code class="language-bash"> root@udoo-studio-hfp:/etc/apt# dpkg --get-selections |grep hpcc
hpcc install

root@udoo-studio-hfp:/etc/apt# readelf -A /usr/bin/hpcc
Attribute Section: aeabi
File Attributes
  Tag_CPU_name: <span style="color: #e6db74;">"7-A"</span>
  Tag_CPU_arch: v7
  Tag_CPU_arch_profile: Application
  Tag_ARM_ISA_use: Yes
  Tag_THUMB_ISA_use: Thumb-2
  Tag_FP_arch: VFPv3-D16
  Tag_ABI_PCS_wchar_t: <span style="color: #ae81ff;">4</span>
  Tag_ABI_FP_denormal: Needed
  Tag_ABI_FP_exceptions: Needed
  Tag_ABI_FP_number_model: IEEE <span style="color: #ae81ff;">754</span>
  Tag_ABI_align_needed: 8-byte
  Tag_ABI_align_preserved: 8-byte, except leaf SP
  Tag_ABI_enum_size: int
  Tag_ABI_HardFP_use: SP and DP
  Tag_ABI_VFP_args: VFP registers
  Tag_CPU_unaligned_access: v6
  Tag_DIV_use: Not allowed</code></pre></div>

<p>Now that we&rsquo;re done kicking the tires, lets take it for a drive!</p>

<p>The intent here was not for a Top 500 run.  Rather, just to stress the Udoo
Quad with a more intensive workload.  For this purpose, I wrote a small
Qt program to display the CPU temperature.  I was curious to understand how
the system would heat up given that it&rsquo;s passively cooled (with a nice
heatsink).</p>

<p>The output from my Linpack run is below:</p>

<div class="highlight"><pre><code class="language-bash"> <span style="color: #f92672;">================================================================================</span>
HPLinpack 2.0  --  High-Performance Linpack benchmark  --   September 10, <span style="color: #ae81ff;">2008</span>
Written by A. Petitet and R. Clint Whaley,  Innovative Computing Laboratory, UTK
Modified by Piotr Luszczek, Innovative Computing Laboratory, UTK
Modified by Julien Langou, University of Colorado Denver
<span style="color: #f92672;">================================================================================</span>
 

An explanation of the input/output parameters follows:
T/V    : Wall time / encoded variant.
N      : The order of the coefficient matrix A.
NB     : The partitioning blocking factor.
P      : The number of process rows.
Q      : The number of process columns.
Time   : Time in seconds to solve the linear system.
Gflops : Rate of execution <span style="color: #66d9ef;">for</span> solving the linear system.

The following parameter values will be used:
 
N      :    <span style="color: #ae81ff;">7000</span> 
NB     :      <span style="color: #ae81ff;">90</span>      <span style="color: #ae81ff;">192</span>      <span style="color: #ae81ff;">110</span> 
PMAP   : Row-major process mapping
P      :       <span style="color: #ae81ff;">2</span> 
Q      :       <span style="color: #ae81ff;">2</span> 
PFACT  :   Right 
NBMIN  :       <span style="color: #ae81ff;">4</span> 
NDIV   :       <span style="color: #ae81ff;">2</span> 
RFACT  :   Crout 
BCAST  :  1ringM 
DEPTH  :       <span style="color: #ae81ff;">1</span> 
SWAP   : Mix <span style="color: #f92672;">(</span>threshold <span style="color: #f92672;">=</span> 64<span style="color: #f92672;">)</span>
L1     : transposed form
U      : transposed form
EQUIL  : yes
ALIGN  : <span style="color: #ae81ff;">8</span> double precision words

--------------------------------------------------------------------------------

- The matrix A is randomly generated <span style="color: #66d9ef;">for</span> each test.
- The following scaled residual check will be computed:
      <span style="color: #f92672;">||</span>Ax-b<span style="color: #f92672;">||</span>_oo / <span style="color: #f92672;">(</span> eps * <span style="color: #f92672;">(</span> <span style="color: #f92672;">||</span> x <span style="color: #f92672;">||</span>_oo * <span style="color: #f92672;">||</span> A <span style="color: #f92672;">||</span>_oo + <span style="color: #f92672;">||</span> b <span style="color: #f92672;">||</span>_oo <span style="color: #f92672;">)</span> * N <span style="color: #f92672;">)</span>
- The relative machine precision <span style="color: #f92672;">(</span>eps<span style="color: #f92672;">)</span> is taken to be               1.110223e-16
- Computational tests pass <span style="color: #66d9ef;">if</span> scaled residuals are less than                16.0
 
<span style="color: #f92672;">================================================================================</span>
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4        <span style="color: #ae81ff;">7000</span>    <span style="color: #ae81ff;">90</span>     <span style="color: #ae81ff;">2</span>     <span style="color: #ae81ff;">2</span>             133.23              1.717e+00
--------------------------------------------------------------------------------
<span style="color: #f92672;">||</span>Ax-b<span style="color: #f92672;">||</span>_oo/<span style="color: #f92672;">(</span>eps*<span style="color: #f92672;">(||</span>A<span style="color: #f92672;">||</span>_oo*<span style="color: #f92672;">||</span>x<span style="color: #f92672;">||</span>_oo+<span style="color: #f92672;">||</span>b<span style="color: #f92672;">||</span>_oo<span style="color: #f92672;">)</span>*N<span style="color: #f92672;">)=</span>        0.0033466 ...... PASSED
<span style="color: #f92672;">================================================================================</span>
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4        <span style="color: #ae81ff;">7000</span>   <span style="color: #ae81ff;">192</span>     <span style="color: #ae81ff;">2</span>     <span style="color: #ae81ff;">2</span>             130.95              1.747e+00
--------------------------------------------------------------------------------
<span style="color: #f92672;">||</span>Ax-b<span style="color: #f92672;">||</span>_oo/<span style="color: #f92672;">(</span>eps*<span style="color: #f92672;">(||</span>A<span style="color: #f92672;">||</span>_oo*<span style="color: #f92672;">||</span>x<span style="color: #f92672;">||</span>_oo+<span style="color: #f92672;">||</span>b<span style="color: #f92672;">||</span>_oo<span style="color: #f92672;">)</span>*N<span style="color: #f92672;">)=</span>        0.0034782 ...... PASSED
<span style="color: #f92672;">================================================================================</span>
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4        <span style="color: #ae81ff;">7000</span>   <span style="color: #ae81ff;">110</span>     <span style="color: #ae81ff;">2</span>     <span style="color: #ae81ff;">2</span>             137.24              1.667e+00
--------------------------------------------------------------------------------
<span style="color: #f92672;">||</span>Ax-b<span style="color: #f92672;">||</span>_oo/<span style="color: #f92672;">(</span>eps*<span style="color: #f92672;">(||</span>A<span style="color: #f92672;">||</span>_oo*<span style="color: #f92672;">||</span>x<span style="color: #f92672;">||</span>_oo+<span style="color: #f92672;">||</span>b<span style="color: #f92672;">||</span>_oo<span style="color: #f92672;">)</span>*N<span style="color: #f92672;">)=</span>        0.0034961 ...... PASSED
<span style="color: #f92672;">================================================================================</span>

Finished      <span style="color: #ae81ff;">3</span> tests with the following results:
              <span style="color: #ae81ff;">3</span> tests completed and passed residual checks,
              <span style="color: #ae81ff;">0</span> tests completed and failed residual checks,
              <span style="color: #ae81ff;">0</span> tests skipped because of illegal input values.
--------------------------------------------------------------------------------</code></pre></div>

<p>During the runs of HPCC (in particular the HPLinpack portion), I observed the
CPU temperature climb to ~60 degrees Celsius.</p>

<p>I produced a short video showing a run of HPCC along with the Qt CPU temperature
app that I created.</p>


<div style="padding-bottom: 56.25%; height: 0; overflow: hidden;">
  
</div>


<p>That wraps up a successful first test drive. What&rsquo;s next? OpenCL sees like
the next logical step.</p>