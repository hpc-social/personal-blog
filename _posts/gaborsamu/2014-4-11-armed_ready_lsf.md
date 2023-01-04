---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2014-04-11 16:39:53'
layout: post
original_url: https://www.gaborsamu.com/blog/armed_ready_lsf/
slug: armed-and-ready-with-ibm-platform-lsf
title: Armed and ready with IBM Platform LSF
---

<p>These days it&rsquo;s not uncommon to hear about CPUs based upon ARM cores. They can
be found in mobile phones, embedded systems, laptops and even servers. Indeed,
recently there have been a number of major announcements from vendors building
processors based ARM cores. This includes the AMD Opteron A1100, NVIDIA Tegra
K1 and even the Apple A7, which is used the iPhone 5s. What these all have in
common is that they are 64-bit and based on the ARM v8 ISA. At the same time,
the ARM-server chip startup Calxeda announced it was shutting down. Surging
power requirements, as well as the announcement of 64-bit chips have led to
renewed interest in energy efficient ARM based processors for high performance
computing.</p>

<p>When building out an infrastructure for Technical Computing, a workload manager
is typically used to control access to the computing resources. As it turns out,the leading workload manager IBM Platfom LSF (formerly Platform Computing) has
supported Linux on ARM for about 10 years. In fact, today there are IBM
clients using Platform LSF on Linux ARM-based clusters as part of mobile
device design and testing.</p>

<p>The current release of IBM Platform LSF 9.1.2 supports Linux on ARM v7 with
upcoming support for ARM v8. Given that Platform LSF provides the ability to
build out heterogeneous clusters, creating a compute cluster containing ARM,
Power and x86 based nodes is a snap. Jobs may be targetted to a specific
processor type and the optional portal IBM Platform Application Centre
provides an easy to use, highly configurable, application-centric web based
interface for job management.</p>

<p><strong>Hello. How do you &ldquo;doo&rdquo;?</strong></p>

<p>I&rsquo;ve recently had the opportunity to test IBM Platform LSF on two node, ARM
based cluster . The IBM Platform LSF master node was a Udoo Quad system running Debian Wheezy ARMv7 EABI hard-float. The second node was running Fedora on a
ARM v8 simulator. Installation and operation of the software was identical to
other platforms. Using the Platform LSF ELIM (External LIM) facility for
adding external load indices, I was able to quickly create a script to load
the processor temperature on the Udoo Quad system.</p>

<p>Now, putting Platform LSF through it&rsquo;s paces, we see the type and model and
other physical characteristics of the nodes are detected.</p>

<div class="highlight"><pre><code class="language-bash">$ lshosts -w
HOST_NAME type model cpuf ncpus maxmem maxswp server RESOURCES
udoo LINUX_ARM ARM7l 60.0 <span style="color: #ae81ff;">4</span> 875M - Yes <span style="color: #f92672;">(</span>mg<span style="color: #f92672;">)</span>
ma1arms4 LINUX_ARM   ARM8  60.0     <span style="color: #ae81ff;">1</span>   1.8G   1.9G    Yes <span style="color: #f92672;">()</span></code></pre></div>

<p>Looking at the load information on the system, we see the built-in load
indices, in addition to the cputemp metric which I introduced to report the
CPU temperature (Celsius). At this point the system is essentially idle.</p>

<div class="highlight"><pre><code class="language-bash">$ lsload -l
HOST_NAME status r15s r1m r15m ut pg io ls it tmp swp mem cputemp
udoo ok 0.5 0.6 1.5 4% 0.0 <span style="color: #ae81ff;">311</span> <span style="color: #ae81ff;">1</span> <span style="color: #ae81ff;">0</span> 1297M 0M 701M 45.0
ma1arms4   busy   3.6  *7.7   6.2  52%   0.0   <span style="color: #ae81ff;">50</span> <span style="color: #ae81ff;">3</span>   <span style="color: #ae81ff;">0</span>  954M  1.9G  1.6G 0.0</code></pre></div>

<p>Next, we submit a job for execution to Platform LSF. Rather than the requisite
sleep job, we submit something a bit more interesting, the HPC Challenge
Benchmark (HPCC). Debian Wheezy happens to include a pre-compiled binary which
is compiled against OpenMPI.</p>

<p>As the Udoo Quad is a 4 core system (as the name implies), hpcc is submitted
requesting 4 cores.</p>

<div class="highlight"><pre><code class="language-bash">$ bsub -n <span style="color: #ae81ff;">4</span> mpiexec -n <span style="color: #ae81ff;">4</span> /usr/bin/hpcc
Job &lt;2&gt; is submitted to default queue &lt;normal&gt;.</code></pre></div>

<p>With HPCC running, we quickly see the utilization as well as the CPU
temperature increase to 60C.</p>

<div class="highlight"><pre><code class="language-bash">$ lsload -l
HOST_NAME status r15s r1m r15m ut pg io ls it tmp swp mem cputemp
udoo ok 5.1 5.1 2.4 94% 0.0 <span style="color: #ae81ff;">49</span> <span style="color: #ae81ff;">1</span> <span style="color: #ae81ff;">0</span> 1376M 0M 497M 60.0
ma1arms4   ok   0.5  1.1   1.2  40%   0.0   <span style="color: #ae81ff;">50</span> <span style="color: #ae81ff;">3</span>   <span style="color: #ae81ff;">0</span>  954M  1.9G  1.6G 0.0</code></pre></div>

<p>During the life of the job, the resource utilization may be easily viewed using the Platform LSF user commands. This includes details such as the PIDs which
the job is comprised of.</p>

<div class="highlight"><pre><code class="language-bash">$ bjobs -l
 
Job &lt;2&gt;, User &lt;debian&gt;, Project &lt;default&gt;, Status &lt;RUN&gt;, Queue &lt;normal&gt;, 
                    Command &lt;mpiexec -n <span style="color: #ae81ff;">4</span> /usr/bin/hpcc&gt;, Share group charged &lt;/debian&gt;
Sun Feb <span style="color: #ae81ff;">2</span> 23:49:48: Submitted from host &lt;udoo&gt;, CWD &lt;/opt/ibm/lsf/conf&gt;, 
                    <span style="color: #ae81ff;">4</span> Processors Requested;
Sun Feb <span style="color: #ae81ff;">2</span> 23:49:48: Started on <span style="color: #ae81ff;">4</span> Hosts/Processors &lt;udoo&gt; &lt;udoo&gt; &lt;udoo&gt; &lt;udoo&gt;,
Execution Home &lt;/home/debian&gt;, Execution CWD &lt;/opt/ibm/lsf/conf&gt;;
Sun Feb <span style="color: #ae81ff;">2</span> 23:51:05: Resource usage collected.
The CPU time used is <span style="color: #ae81ff;">227</span> seconds.
MEM: <span style="color: #ae81ff;">140</span> Mbytes; SWAP: <span style="color: #ae81ff;">455</span> Mbytes; NTHREAD: <span style="color: #ae81ff;">8</span>
PGID: 15678; PIDs: <span style="color: #ae81ff;">15678</span> <span style="color: #ae81ff;">15679</span> <span style="color: #ae81ff;">15681</span> <span style="color: #ae81ff;">15682</span> <span style="color: #ae81ff;">15683</span> <span style="color: #ae81ff;">15684</span>
<span style="color: #ae81ff;">15685</span>
....
....</code></pre></div>

<p><strong>New Roads?</strong></p>

<p>Here we could speak of GFlops, and other such measures of performance, but
that was not my objective. The key, is that there is a growing interest in
non-x86 solutions for Technical Computing. IBM Platform LSF software has
supported and continues to support a wide variety of operating systems and
processor architectures, from ARM to IBM Power to IBM System z.</p>

<p>As for ARM based development boards such as the Udoo Quad, Parallela Board,
etc., they are inexpensive as well as being energy efficient. This fact makes
them of interest to HPC scientists looking at possible approaches to energy
efficiency for HPC workloads. Let us know your thoughts about the suitability
of ARM for HPC workloads.</p>