---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2022-06-06 18:54:07'
layout: post
original_url: https://www.gaborsamu.com/blog/neunundneunzig_reform/
slug: neunundneunzig-mnt-reform-s-
title: Neunundneunzig MNT Reform(s)
---

<p>I&rsquo;ll admit it. I sat on the fence for a long time before placing an order
for the MNT Reform 2 laptop. At the time, I was in the market for a laptop
as my 2 Macbook Pro retina laptops were repurposed for online schooling
for my children during the pandemic (and as it turns out were never
returned to me).</p>

<p>I have fairly extensive experience with Arm-based systems and was aware of potential
angst with custom distros when specific system support is not in the Linux mainline.
Yes this has been pretty much addressed - for servers with the Arm SBSA
specifications. However the MNT Reform 2 was never marketed as SBSA.</p>

<p>With eyes wide open, I ultimately decided to go ahead an order an MNT Reform 2.
My laptop needs were really for light coding/scripting, occasional browsing,
writing (blogs, etc), tinkering and as a terminal to my other systems. Sure,
these requirements coud have bee met by some less expensive x86 laptops or
even Chromebooks. But those are distinctly lacking a cool factor. What really helped
to reach this decision was the following:</p>

<ul>
<li>Put together by a small, enthusiastic team</li>
<li>A proper keyboard and cool trackball in a laptop</li>
<li>Intel outside</li>
<li>Swappable CPUs (there are some drop in replacements in the works)</li>
<li>User replaceable, 18650 batteries</li>
<li>Antithesis of paper thin laptops</li>
</ul>
<p>Of course, knowing that the Reform is based on the NXP/Freescale i.MX8MQ with
4 x Arm Cortex-A53 cores (1.5 GHz), I knew it was not going to be a barn burner
in terms of performance.</p>

<p><strong>Late to the party</strong></p>

<p>Because of my procrastination, I only recieved my Reform this past week.
Given that they&rsquo;ve been shipping for some time, I definitely had that
<em>late to the party feeling</em>. In a way this was good though as all of the
write-ups and videos that have been posted over time gave me a good
idea of what to expect. So in this blog I don&rsquo;t expect to cover
anything ground breaking - just my experience so far.</p>

<p>Much has been written about the packaging of the system. And in this sense
it didn&rsquo;t disappoint. You could tell that it was packaged with great care
by the team at MNT and it was frankly a very enjoyable experience to
unwrap the components. I&rsquo;ve included a collage of photos at the end of this blog
of the Reform.</p>

<p>And that wasn&rsquo;t the only fun. Right away I had to remove the transparent bottom
cover of the Reform in order to connect up the batteries and install the Wifi and
NVMe SSD.  At this time I also replaced the plastic port covers with the optional metal
versions that I ordered earlier this year. Once this was done, the system sprang to
life and I was able to very quickly get it booting from the encrypted NVMe thanks
to the detailed handbook that was also included in the bundle I purchased and
tips on the <a href="https://community.mnt.re/">MNT Community site</a>.</p>

<p>As for the keyboard, I really enjoy the tactile feel of it. It&rsquo;s quite a refreshing
experience from the mushy keyboard on the MacBook Air M1 that I use for work. And
although I ordered both the trackball and trackpad for the Reform, I&rsquo;ll likely
stick with the trackball for now as it&rsquo;s just a pleasure to use. Note that
my Reform appears to have the updated trackball which has ball bearings for a
smoother action.</p>

<p><strong>Fanless bitte</strong></p>

<p>Of course one of the first things an #HPC minded person like myself will do with
a system is run <a href="https://www.netlib.org/benchmark/hpl/">High-performance Linpack</a> (HPL) on it. This is a force of habit for me
and thought it may also prove to be a good way to burn in the system.</p>

<p>So I started with <a href="https://www.open-mpi.org/">Open MPI</a>. I downloaded and compiled Open MPI v4.1.4. This completed
without a hitch. Note that I didn&rsquo;t specify any specific flags configuring Open MPI
other than a prefix for the installation location (under $HOME).</p>

<p>HPL was easy to compile as well. Note that I simply used the OS ATLAS and BLAS
libraries and the OS supplied compiler(s). So we can say that this is not an
optimized build of HPL.</p>

<p>And below we see the results of the run of xhpl below, which achieved a result of
4.2 GFLOPS.</p>

<div class="highlight"><pre><code class="language-plaintext">$ mpirun -np 4 ./xhpl 
================================================================================
HPLinpack 2.3  --  High-Performance Linpack benchmark  --   December 2, 2018
Written by A. Petitet and R. Clint Whaley,  Innovative Computing Laboratory, UTK
Modified by Piotr Luszczek, Innovative Computing Laboratory, UTK
Modified by Julien Langou, University of Colorado Denver
================================================================================

An explanation of the input/output parameters follows:
T/V    : Wall time / encoded variant.
N      : The order of the coefficient matrix A.
NB     : The partitioning blocking factor.
P      : The number of process rows.
Q      : The number of process columns.
Time   : Time in seconds to solve the linear system.
Gflops : Rate of execution for solving the linear system.

The following parameter values will be used:

N      :   19000 
NB     :     192 
PMAP   : Row-major process mapping
P      :       2 
Q      :       2 
PFACT  :   Right 
NBMIN  :       4 
NDIV   :       2 
RFACT  :   Crout 
BCAST  :  1ringM 
DEPTH  :       1 
SWAP   : Mix (threshold = 64)
L1     : transposed form
U      : transposed form
EQUIL  : yes
ALIGN  : 8 double precision words

--------------------------------------------------------------------------------

- The matrix A is randomly generated for each test.
- The following scaled residual check will be computed:
      ||Ax-b||_oo / ( eps * ( || x ||_oo * || A ||_oo + || b ||_oo ) * N )
- The relative machine precision (eps) is taken to be               1.110223e-16
- Computational tests pass if scaled residuals are less than                16.0

================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       19000   192     2     2            1073.27             4.2610e+00
HPL_pdgesv() start time Mon Jun  6 12:08:36 2022

HPL_pdgesv() end time   Mon Jun  6 12:26:30 2022

--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   1.18409443e-03 ...... PASSED
================================================================================

Finished      1 tests with the following results:
              1 tests completed and passed residual checks,
              0 tests completed and failed residual checks,
              0 tests skipped because of illegal input values.
--------------------------------------------------------------------------------

End of Tests.
================================================================================</code></pre></div>

<p>Just for kicks, I&rsquo;ve also included a screenshot of <em>lstopo</em>, which is part of the
<a href="https://www.open-mpi.org/projects/hwloc/">Portal Hardware Locality (hwloc)</a> project. I am a bit confused as to why the
L1 and L2 cache sizes are zero though in the output.</p>

<figure><img src="https://www.gaborsamu.com/images/reform_lstopo.jpg" />
</figure>

<p>I&rsquo;ve included the output from some system commands below including <em>lscpu</em>, <em>lspci</em> and <em>lsusb</em>.</p>

<div class="highlight"><pre><code class="language-plaintext">$ lscpu
Architecture:                    aarch64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
CPU(s):                          4
On-line CPU(s) list:             0-3
Thread(s) per core:              1
Core(s) per socket:              4
Socket(s):                       1
NUMA node(s):                    1
Vendor ID:                       ARM
Model:                           4
Model name:                      Cortex-A53
Stepping:                        r0p4
CPU max MHz:                     1500.0000
CPU min MHz:                     1000.0000
BogoMIPS:                        16.66
NUMA node0 CPU(s):               0-3
Vulnerability Itlb multihit:     Not affected
Vulnerability L1tf:              Not affected
Vulnerability Mds:               Not affected
Vulnerability Meltdown:          Not affected
Vulnerability Spec store bypass: Not affected
Vulnerability Spectre v1:        Mitigation; \__user pointer sanitization
Vulnerability Spectre v2:        Not affected
Vulnerability Srbds:             Not affected
Vulnerability Tsx async abort:   Not affected
Flags:                           fp asimd evtstrm aes pmull sha1 sha2 crc32 cpuid</code></pre></div>

<div class="highlight"><pre><code class="language-plaintext">$ lspci
0000:00:00.0 PCI bridge: Synopsys, Inc. DWC_usb3 / PCIe bridge (rev 01)
0000:01:00.0 Network controller: Qualcomm Atheros AR928X Wireless Network Adapter (PCI-Express) (rev 01)
0001:00:00.0 PCI bridge: Synopsys, Inc. DWC_usb3 / PCIe bridge (rev 01)
0001:01:00.0 Non-Volatile memory controller: Silicon Motion, Inc. SM2262/SM2262EN SSD Controller (rev 03)</code></pre></div>

<div class="highlight"><pre><code class="language-plaintext">$ lsusb
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 002: ID 0451:8140 Texas Instruments, Inc. TUSB8041 4-Port Hub
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 004: ID 03eb:2041 Atmel Corp. LUFA Mouse Demo Application
Bus 001 Device 003: ID 03eb:2042 Atmel Corp. LUFA Keyboard Demo Application
Bus 001 Device 002: ID 0451:8142 Texas Instruments, Inc. TUSB8041 4-Port Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub</code></pre></div>

<p>So that&rsquo;s a very brief look at my initial experiences with the Reform laptop. I&rsquo;ve
only scratched the surface here, but so far I&rsquo;m liking what I&rsquo;m seeing. As for the
<em>neunundneunzig</em> title reference, well I suppose that&rsquo;s part of the vibe I got with
the laptop.</p>

<p>A few photos for your viewing pleasure!</p>

<p><figure><img src="https://www.gaborsamu.com/images/reform_bottom.jpg" />
</figure>

<figure><img src="https://www.gaborsamu.com/images/reform_firstboot.jpg" />
</figure>
</p>