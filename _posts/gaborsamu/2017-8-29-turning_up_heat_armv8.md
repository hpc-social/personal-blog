---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2017-08-29 20:07:13'
layout: post
original_url: https://www.gaborsamu.com/blog/turning_up_heat_armv8/
slug: turning-up-the-heat-on-my-armada-8040
title: Turning up the heat...on my Armada 8040
---

<p>Although I took delivery of a shiny new <a href="https://www.solid-run.com/marvell-armada-family/macchiatobin/">SolidRun Marvell macchiatoBIN</a> a few months back (end May), I&rsquo;ve not really had a chance
to put it through it&rsquo;s paces until now.  For those of you who are not familiar with the board, it&rsquo;s a high-performance 64-bit Arm
(v8) board designed really for networking.  It&rsquo;s based on the Marvell ARMADA 8040 processor for those who like to keep track. For
those looking for more information about the board, there is a community page <a href="http://macchiatobin.net/">here</a>.</p>

<p>What struck me about the board when I originally unpacked it were the shiny heatsinks.  Definitely looks cool on my workbench
(desk)! They did seem up to the task of keeping the mighty ARMADA 8040 cool as a cucumber -
or so I thought.</p>

<figure><img src="https://www.gaborsamu.com/images/8040_heatsinks.jpg" />
</figure>

<p>Following the procedure (struggling) to install Ubuntu as described on the macchiatoBIN <a href="http://wiki.macchiatobin.net/tiki-index.php?page=BSP+HowTo">wiki</a> - which ironically required me to use
an x86 box to compile some necessary bits, I was off to the races with Ubuntu 16.04.3 LTS (Xenial Xerus).  Note that this whole
procedure left much to be desired as it was my understanding that this board was to be ARM <a href="https://en.wikipedia.org/wiki/Server_Base_System_Architecture">SBSA</a> compliant - which would allow any
compliant OS distro to be used.  This is something which at the time of writing is not the case - hope that an update does address
this.</p>

<p>Being a high-performance computing kind of guy, my first challenge was to run the High-Performance Linpack (HPL) on the system.  HPL
you say?  Yes, I know we can debate the merits of HPL all day long, but nevertheless it&rsquo;s still a measure of some specific dimensions
of system performance - and indeed it&rsquo;s used to rank systems on the TOP500 list of Supercomputers.  Because I was looking to run more
than just HPL on the system, I opted to install Phoronix test suite which includes HPCC (HPC Challenge) as an available benchmark.</p>

<p>To get warmed up, I decided to first run the well know <em>stream</em> memory benchmark.  Via Phoronix, I installed the stream benchmark and
executed it.</p>

<div class="highlight"><pre><code class="language-bash">root@flotta:~# phoronix-test-suite install-test pts/stream

 

Phoronix Test Suite v5.2.1
 

    To Install: pts/stream-1.3.1
 

    Determining File Requirements ...........................................
    Searching Download Caches ...............................................
 

    <span style="color: #ae81ff;">1</span> Test To Install

        <span style="color: #ae81ff;">1</span> File To Download <span style="color: #f92672;">[</span>0.01MB<span style="color: #f92672;">]</span>
        1MB Of Disk Space Is Needed
 

    pts/stream-1.3.1:

        Test Installation <span style="color: #ae81ff;">1</span> of <span style="color: #ae81ff;">1</span>
        <span style="color: #ae81ff;">1</span> File Needed <span style="color: #f92672;">[</span>0.01 MB / <span style="color: #ae81ff;">1</span> Minute<span style="color: #f92672;">]</span>
        Downloading: stream-2013-01-17.tar.bz2                       <span style="color: #f92672;">[</span>0.01MB<span style="color: #f92672;">]</span>
        Estimated Download Time: 1m .........................................
        Installation Size: 0.1 MB
        Installing Test @ 19:45:06

<span style="color: #f92672;">[</span>NOTICE<span style="color: #f92672;">]</span> Undefined: <span style="color: #ae81ff;">0</span> in phodevi_cpu:267

<span style="color: #f92672;">[</span>NOTICE<span style="color: #f92672;">]</span> Undefined: <span style="color: #ae81ff;">0</span> in phodevi_cpu:272</code></pre></div>

<p>Next, we execute the benchmark <em>stream</em></p>

<div class="highlight"><pre><code class="language-bash">root@flotta:~# phoronix-test-suite benchmark pts/stream

 

Phoronix Test Suite v5.2.1

 

    Installed: pts/stream-1.3.1

 

 

Stream 2013-01-17:

    pts/stream-1.3.1

    Memory Test Configuration

        1: Copy

        2: Scale

        3: Add

        4: Triad

        5: Test All Options

        Type: <span style="color: #ae81ff;">5</span>

 

 

System Information

 

Hardware:

Processor: Unknown @ 1.30GHz <span style="color: #f92672;">(</span><span style="color: #ae81ff;">4</span> Cores<span style="color: #f92672;">)</span>, Memory: 4096MB, Disk: 8GB 8GME4R

 

Software:

OS: Ubuntu 16.04, Kernel: 4.4.8-armada-17.02.2-g4126e30 <span style="color: #f92672;">(</span>aarch64<span style="color: #f92672;">)</span>, Compiler: GCC 5.4.0 20160609, File-System: ext4

 

    Would you like to save these test results <span style="color: #f92672;">(</span>Y/n<span style="color: #f92672;">)</span>: n

 

 

Stream 2013-01-17:

    pts/stream-1.3.1 <span style="color: #f92672;">[</span>Type: Copy<span style="color: #f92672;">]</span>

    Test <span style="color: #ae81ff;">1</span> of <span style="color: #ae81ff;">4</span>

    Estimated Trial Run Count:    <span style="color: #ae81ff;">5</span>

    Estimated Test Run-Time:      <span style="color: #ae81ff;">7</span> Minutes

    Estimated Time To Completion: <span style="color: #ae81ff;">25</span> Minutes

        Started Run <span style="color: #ae81ff;">1</span> @ 19:46:03

        Started Run <span style="color: #ae81ff;">2</span> @ 19:48:09

        Started Run <span style="color: #ae81ff;">3</span> @ 19:50:14

        Started Run <span style="color: #ae81ff;">4</span> @ 19:52:19

        Started Run <span style="color: #ae81ff;">5</span> @ 19:54:24  <span style="color: #f92672;">[</span>Std. Dev: 0.35%<span style="color: #f92672;">]</span>

 

    Test Results:

        6701.1

        6669.1

        6655.9

        6637.4

        6657.1

 

    Average: 6664.12 MB/s

 

 

Stream 2013-01-17:

    pts/stream-1.3.1 <span style="color: #f92672;">[</span>Type: Scale<span style="color: #f92672;">]</span>

    Test <span style="color: #ae81ff;">2</span> of <span style="color: #ae81ff;">4</span>

    Estimated Trial Run Count:    <span style="color: #ae81ff;">5</span>

    Estimated Test Run-Time:      <span style="color: #ae81ff;">7</span> Minutes

    Estimated Time To Completion: <span style="color: #ae81ff;">19</span> Minutes

        Started Run <span style="color: #ae81ff;">1</span> @ 19:56:27

        Started Run <span style="color: #ae81ff;">2</span> @ 19:56:27

        Started Run <span style="color: #ae81ff;">3</span> @ 19:56:27

        Started Run <span style="color: #ae81ff;">4</span> @ 19:56:27

        Started Run <span style="color: #ae81ff;">5</span> @ 19:56:27  <span style="color: #f92672;">[</span>Std. Dev: 0.12%<span style="color: #f92672;">]</span>

 

    Test Results:

        7248.8

        7261.8

        7252.8

        7245.6

        7266.1

 

    Average: 7255.02 MB/s

 

 

Stream 2013-01-17:

    pts/stream-1.3.1 <span style="color: #f92672;">[</span>Type: Triad<span style="color: #f92672;">]</span>

    Test <span style="color: #ae81ff;">3</span> of <span style="color: #ae81ff;">4</span>

    Estimated Trial Run Count:    <span style="color: #ae81ff;">5</span>

    Estimated Test Run-Time:      <span style="color: #ae81ff;">7</span> Minutes

    Estimated Time To Completion: <span style="color: #ae81ff;">13</span> Minutes

        Started Run <span style="color: #ae81ff;">1</span> @ 19:56:27

        Started Run <span style="color: #ae81ff;">2</span> @ 19:56:27

        Started Run <span style="color: #ae81ff;">3</span> @ 19:56:27

        Started Run <span style="color: #ae81ff;">4</span> @ 19:56:27

        Started Run <span style="color: #ae81ff;">5</span> @ 19:56:27  <span style="color: #f92672;">[</span>Std. Dev: 0.47%<span style="color: #f92672;">]</span>

 

    Test Results:

        6872.3

        6895.9

        6934.9

        6847.9

        6889.5

 

    Average: 6888.10 MB/s

 

 

Stream 2013-01-17:

    pts/stream-1.3.1 <span style="color: #f92672;">[</span>Type: Add<span style="color: #f92672;">]</span>

    Test <span style="color: #ae81ff;">4</span> of <span style="color: #ae81ff;">4</span>

    Estimated Trial Run Count:    <span style="color: #ae81ff;">5</span>

    Estimated Time To Completion: <span style="color: #ae81ff;">7</span> Minutes

        Started Run <span style="color: #ae81ff;">1</span> @ 19:56:27

        Started Run <span style="color: #ae81ff;">2</span> @ 19:56:27

        The test run ended prematurely.

        Started Run <span style="color: #ae81ff;">3</span> @ 19:56:27

        Started Run <span style="color: #ae81ff;">4</span> @ 19:56:27

        Started Run <span style="color: #ae81ff;">5</span> @ 19:56:27

        The test run ended prematurely.  <span style="color: #f92672;">[</span>Std. Dev: 0.09%<span style="color: #f92672;">]</span>

 

    Test Results:

        6559.5

        6549.8

        6560.8

 

    Average: 6556.70 MB/s</code></pre></div>

<p>We see during execution that stream definitely puts the system through it&rsquo;s paces</p>

<div class="highlight"><pre><code class="language-bash">top - 19:47:20 up  3:07,  <span style="color: #ae81ff;">2</span> users,  load average: 3.10, 1.21, 0.70

Tasks: <span style="color: #ae81ff;">119</span> total,   <span style="color: #ae81ff;">2</span> running, <span style="color: #ae81ff;">117</span> sleeping,   <span style="color: #ae81ff;">0</span> stopped,   <span style="color: #ae81ff;">0</span> zombie
%Cpu<span style="color: #f92672;">(</span>s<span style="color: #f92672;">)</span>: 96.3 us,  0.4 sy,  0.0 ni,  1.7 id,  0.0 wa,  0.0 hi,  1.7 si,  0.0 st
KiB Mem :  <span style="color: #ae81ff;">3779668</span> total,   <span style="color: #ae81ff;">986744</span> free,  <span style="color: #ae81ff;">2410052</span> used,   <span style="color: #ae81ff;">382872</span> buff/cache
KiB Swap:        <span style="color: #ae81ff;">0</span> total,        <span style="color: #ae81ff;">0</span> free,        <span style="color: #ae81ff;">0</span> used.  <span style="color: #ae81ff;">1287376</span> avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND    
<span style="color: #ae81ff;">18920</span> root      <span style="color: #ae81ff;">20</span>   <span style="color: #ae81ff;">0</span> <span style="color: #ae81ff;">2370508</span> 2.236g   <span style="color: #ae81ff;">1336</span> R 389.1 62.0   4:54.12 stream-bin
<span style="color: #ae81ff;">6854</span> root      <span style="color: #ae81ff;">20</span>   <span style="color: #ae81ff;">0</span>       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   2.3  0.0   0:07.30 kworker/u8+
<span style="color: #ae81ff;">18924</span> root     <span style="color: #ae81ff;">20</span>   <span style="color: #ae81ff;">0</span>    <span style="color: #ae81ff;">9288</span>   <span style="color: #ae81ff;">3208</span>   <span style="color: #ae81ff;">2632</span> R   0.7  0.1   0:00.37 top        
    <span style="color: #ae81ff;">3</span> root      <span style="color: #ae81ff;">20</span>   <span style="color: #ae81ff;">0</span>       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   0.3  0.0   0:00.50 ksoftirqd/0
    <span style="color: #ae81ff;">1</span> root      <span style="color: #ae81ff;">20</span>   <span style="color: #ae81ff;">0</span>    <span style="color: #ae81ff;">6868</span>   <span style="color: #ae81ff;">5144</span>   <span style="color: #ae81ff;">3476</span> S   0.0  0.1   0:03.90 systemd    
    <span style="color: #ae81ff;">2</span> root      <span style="color: #ae81ff;">20</span>   <span style="color: #ae81ff;">0</span>       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   0.0  0.0   0:00.00 kthreadd   
    <span style="color: #ae81ff;">5</span> root       <span style="color: #ae81ff;">0</span> -20       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   0.0  0.0   0:00.00 kworker/0:+
    <span style="color: #ae81ff;">7</span> root      <span style="color: #ae81ff;">20</span>   <span style="color: #ae81ff;">0</span>       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   0.0  0.0   0:01.82 rcu_preempt
    <span style="color: #ae81ff;">8</span> root      <span style="color: #ae81ff;">20</span>   <span style="color: #ae81ff;">0</span>       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   0.0  0.0   0:00.00 rcu_sched
    <span style="color: #ae81ff;">9</span> root      <span style="color: #ae81ff;">20</span>   <span style="color: #ae81ff;">0</span>       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   0.0  0.0   0:00.00 rcu_bh     
   <span style="color: #ae81ff;">10</span> root      rt   <span style="color: #ae81ff;">0</span>       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   0.0  0.0   0:00.08 migration/0
   <span style="color: #ae81ff;">11</span> root      rt   <span style="color: #ae81ff;">0</span>       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   0.0  0.0   0:00.14 watchdog/0
   <span style="color: #ae81ff;">12</span> root      rt   <span style="color: #ae81ff;">0</span>       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   0.0  0.0   0:00.11 watchdog/1
   <span style="color: #ae81ff;">13</span> root      rt   <span style="color: #ae81ff;">0</span>       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   0.0  0.0   0:00.07 migration/1
   <span style="color: #ae81ff;">14</span> root      <span style="color: #ae81ff;">20</span>   <span style="color: #ae81ff;">0</span>       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   0.0  0.0   0:00.03 ksoftirqd/1
   <span style="color: #ae81ff;">16</span> root       <span style="color: #ae81ff;">0</span> -20       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   0.0  0.0   0:00.00 kworker/1:+
   <span style="color: #ae81ff;">17</span> root      rt   <span style="color: #ae81ff;">0</span>       <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span> S   0.0  0.0   0:00.13 watchdog/2</code></pre></div>

<p>Onward and upward as they say.  Moving to the HPCC benchmark which contains HPL.  We install the <em>pts/hpcc</em> test for Phoronix.</p>

<div class="highlight"><pre><code class="language-bash">root@flotta:~# phoronix-test-suite install-test pts/hpcc

 

Phoronix Test Suite v5.2.1

 

    To Install: pts/hpcc-1.2.0

 

    Determining File Requirements ...........................................

    Searching Download Caches ...............................................

 

    <span style="color: #ae81ff;">1</span> Test To Install

        <span style="color: #ae81ff;">1</span> File To Download <span style="color: #f92672;">[</span>0.63MB<span style="color: #f92672;">]</span>

        9MB Of Disk Space Is Needed

 

    pts/hpcc-1.2.0:

        Test Installation <span style="color: #ae81ff;">1</span> of <span style="color: #ae81ff;">1</span>

        <span style="color: #ae81ff;">1</span> File Needed <span style="color: #f92672;">[</span>0.63 MB / <span style="color: #ae81ff;">1</span> Minute<span style="color: #f92672;">]</span>

        Downloading: hpcc-1.5.0.tar.gz                               <span style="color: #f92672;">[</span>0.63MB<span style="color: #f92672;">]</span>

        Estimated Download Time: 1m .........................................

        Installation Size: <span style="color: #ae81ff;">9</span> MB

        Installing Test @ 20:00:53

 

        <span style="color: #f92672;">[</span>NOTICE<span style="color: #f92672;">]</span> Supported install-time optional variables include $MPI_PATH,

        $MPI_INCLUDE, $MPI_CC, $MPI_LIBS, $LA_PATH, $LA_INCLUDE, $LA_LIBS,

        $CFLAGS, $LD_FLAGS, and $MPI_LD

 

        <span style="color: #f92672;">[</span>NOTICE<span style="color: #f92672;">]</span> Supported run-time optional environment variables include

        $N, $NB, $MPI_NUM_THREADS, $HOSTFILE</code></pre></div>

<p>Before starting the HPL run, I put together a quick script to monitor the temperature utilization during the HPL run.  The script simply
prints out the values of <em>/sys/class/thermal/thermal_zone[X]/temp</em> in human readable format.</p>

<div class="highlight"><pre><code class="language-bash"><span style="color: #75715e;">#!/bin/sh
</span><span style="color: #75715e;"></span>
<span style="color: #66d9ef;">while</span> <span style="color: #f92672;">[</span> true <span style="color: #f92672;">]</span>

<span style="color: #66d9ef;">do</span>

echo <span style="color: #e6db74;">"</span><span style="color: #66d9ef;">$(</span>date<span style="color: #66d9ef;">)</span><span style="color: #e6db74;"> @ </span><span style="color: #66d9ef;">$(</span>hostname<span style="color: #66d9ef;">)</span><span style="color: #e6db74;">"</span>

echo <span style="color: #e6db74;">"-----------------------"</span>

cpu0<span style="color: #f92672;">=</span><span style="color: #e6db74;">`</span>cat /sys/class/thermal/thermal_zone0/temp<span style="color: #e6db74;">`</span>

cpu1<span style="color: #f92672;">=</span><span style="color: #e6db74;">`</span>cat /sys/class/thermal/thermal_zone1/temp<span style="color: #e6db74;">`</span>

cpu2<span style="color: #f92672;">=</span><span style="color: #e6db74;">`</span>cat /sys/class/thermal/thermal_zone2/temp<span style="color: #e6db74;">`</span>

echo <span style="color: #e6db74;">"thermal_zone0 = </span><span style="color: #66d9ef;">$((</span>cpu0/1000<span style="color: #66d9ef;">))</span><span style="color: #e6db74;"> 'C"</span>

echo <span style="color: #e6db74;">"thermal_zone1 = </span><span style="color: #66d9ef;">$((</span>cpu1/1000<span style="color: #66d9ef;">))</span><span style="color: #e6db74;"> 'C"</span>

echo <span style="color: #e6db74;">"thermal_zone2 = </span><span style="color: #66d9ef;">$((</span>cpu2/1000<span style="color: #66d9ef;">))</span><span style="color: #e6db74;"> 'C"</span>

/bin/sleep <span style="color: #ae81ff;">10</span>

<span style="color: #66d9ef;">done</span></code></pre></div>

<p>After starting HPL and monitoring the temperatures, I found that they rapidly climbed to some rather uncomfortable levels - especially given
that my ARMADA 8040 board currently has no cooling fan.  So after kicking off the initial run (see below) I decided to err on the side of
caution and get a fan setup before I do any serious damage to the board.</p>

<div class="highlight"><pre><code class="language-bash">gsamu@flotta:~$ phoronix-test-suite benchmark pts/hpcc

 

Phoronix Test Suite v5.2.1

 

    Installed: pts/hpcc-1.2.0

 

 

HPC Challenge 1.5.0:

    pts/hpcc-1.2.0

    Processor Test Configuration

        1:  G-HPL

        2:  G-Ptrans

        3:  G-Random Access

        4:  G-Ffte

        5:  EP-STREAM Triad

        6:  EP-DGEMM

        7:  Random Ring Latency

        8:  Random Ring Bandwidth

        9:  Max Ping Pong Bandwidth

        10: Test All Options

        Test / Class: <span style="color: #ae81ff;">1</span>

 

 

System Information

 

 

<span style="color: #f92672;">[</span>NOTICE<span style="color: #f92672;">]</span> Undefined: <span style="color: #ae81ff;">0</span> in phodevi_cpu:267

 

<span style="color: #f92672;">[</span>NOTICE<span style="color: #f92672;">]</span> Undefined: <span style="color: #ae81ff;">0</span> in phodevi_cpu:272

Hardware:

Processor: Unknown @ 1.30GHz <span style="color: #f92672;">(</span><span style="color: #ae81ff;">4</span> Cores<span style="color: #f92672;">)</span>, Memory: 4096MB, Disk: 8GB 8GME4R

 

Software:

OS: Ubuntu 16.04, Kernel: 4.4.8-armada-17.02.2-g4126e30 <span style="color: #f92672;">(</span>aarch64<span style="color: #f92672;">)</span>, Compiler: GCC 5.4.0 20160609, File-System: ext4

 

    Would you like to save these test results <span style="color: #f92672;">(</span>Y/n<span style="color: #f92672;">)</span>: n

 

 

HPC Challenge 1.5.0:

    pts/hpcc-1.2.0 <span style="color: #f92672;">[</span>Test / Class: G-HPL<span style="color: #f92672;">]</span>

    Test <span style="color: #ae81ff;">1</span> of <span style="color: #ae81ff;">1</span>

    Estimated Trial Run Count:    <span style="color: #ae81ff;">3</span>

    Estimated Time To Completion: <span style="color: #ae81ff;">1</span> Hour, <span style="color: #ae81ff;">28</span> Minutes

        Started Run <span style="color: #ae81ff;">1</span> @ 20:12:13^C</code></pre></div>

<p>The above run was aborted when the temperature shown by my temperature monitoring script peaked 100 C.</p>

<div class="highlight"><pre><code class="language-bash">Tue Aug <span style="color: #ae81ff;">29</span> 20:23:12 EDT <span style="color: #ae81ff;">2017</span> @ flotta.localdomain

-----------------------

thermal_zone0 <span style="color: #f92672;">=</span> <span style="color: #ae81ff;">99</span> <span style="color: #e6db74;">'C
</span><span style="color: #e6db74;">
</span><span style="color: #e6db74;">thermal_zone1 = 91 '</span>C

thermal_zone2 <span style="color: #f92672;">=</span> <span style="color: #ae81ff;">92</span> <span style="color: #e6db74;">'C
</span><span style="color: #e6db74;">
</span><span style="color: #e6db74;">Tue Aug 29 20:23:22 EDT 2017 @ flotta.localdomain
</span><span style="color: #e6db74;">
</span><span style="color: #e6db74;">-----------------------
</span><span style="color: #e6db74;">
</span><span style="color: #e6db74;">thermal_zone0 = 100 '</span>C

thermal_zone1 <span style="color: #f92672;">=</span> <span style="color: #ae81ff;">92</span> <span style="color: #e6db74;">'C
</span><span style="color: #e6db74;">
</span><span style="color: #e6db74;">thermal_zone2 = 92 '</span>C

Tue Aug <span style="color: #ae81ff;">29</span> 20:23:32 EDT <span style="color: #ae81ff;">2017</span> @ flotta.localdomain

-----------------------

thermal_zone0 <span style="color: #f92672;">=</span> <span style="color: #ae81ff;">101</span> <span style="color: #e6db74;">'C
</span><span style="color: #e6db74;">
</span><span style="color: #e6db74;">thermal_zone1 = 93 '</span>C

thermal_zone2 <span style="color: #f92672;">=</span> <span style="color: #ae81ff;">93</span> <span style="color: #e6db74;">'C
</span><span style="color: #e6db74;">
</span><span style="color: #e6db74;">Tue Aug 29 20:23:42 EDT 2017 @ flotta.localdomain
</span><span style="color: #e6db74;">
</span><span style="color: #e6db74;">-----------------------
</span><span style="color: #e6db74;">
</span><span style="color: #e6db74;">thermal_zone0 = 101 '</span>C

thermal_zone1 <span style="color: #f92672;">=</span> <span style="color: #ae81ff;">94</span> <span style="color: #e6db74;">'C
</span><span style="color: #e6db74;">
</span><span style="color: #e6db74;">thermal_zone2 = 93 '</span>C

Tue Aug <span style="color: #ae81ff;">29</span> 20:23:52 EDT <span style="color: #ae81ff;">2017</span> @ flotta.localdomain

-----------------------

thermal_zone0 <span style="color: #f92672;">=</span> <span style="color: #ae81ff;">101</span> <span style="color: #e6db74;">'C
</span><span style="color: #e6db74;">
</span><span style="color: #e6db74;">thermal_zone1 = 94 '</span>C

thermal_zone2 <span style="color: #f92672;">=</span> <span style="color: #ae81ff;">94</span> <span style="color: #960050; background-color: #1e0010;">'</span>C</code></pre></div>

<p>So as I wait for my new cooling fan and Open Benchtable to arrive, I&rsquo;ll get back to thrashing some good old Intel hardware&hellip;Hey for some
real fun, I can disconnect the CPU fans on those ones :)</p>

<p>Gábor out!</p>

<p><strong>UPDATE!!!</strong></p>

<p>Well I decided to press ahead tonight with a run of HPL on my macchiatoBIN board.  To monitor the temperature (recall that my current
configuration is with passive cooling) I put together a small script to dump the values of the following to a text file during the HPL run:</p>

<ul>
<li><em>/sys/class/thermal/thermal_zone0/temp</em></li>
<li><em>/sys/class/thermal/thermal_zone1/temp</em></li>
<li><em>/sys/class/thermal/thermal_zone2/temp</em></li>
</ul>
<p>The run started ok, but I lost contact with the macchiatoBIN after about 55 minutes&hellip;when it was on run 2 of HPL:</p>

<div class="highlight"><pre><code class="language-bash">gsamu@flotta:~$ phoronix-test-suite benchmark pts/hpcc

 

Phoronix Test Suite v5.2.1

 

    Installed: pts/hpcc-1.2.0

 

 

HPC Challenge 1.5.0:

    pts/hpcc-1.2.0

    Processor Test Configuration

        1:  G-HPL

        2:  G-Ptrans

        3:  G-Random Access

        4:  G-Ffte

        5:  EP-STREAM Triad

        6:  EP-DGEMM

        7:  Random Ring Latency

        8:  Random Ring Bandwidth

        9:  Max Ping Pong Bandwidth

        10: Test All Options

        Test / Class: <span style="color: #ae81ff;">1</span>

 

 

System Information

 

 

<span style="color: #f92672;">[</span>NOTICE<span style="color: #f92672;">]</span> Undefined: <span style="color: #ae81ff;">0</span> in phodevi_cpu:267

 

<span style="color: #f92672;">[</span>NOTICE<span style="color: #f92672;">]</span> Undefined: <span style="color: #ae81ff;">0</span> in phodevi_cpu:272

Hardware:

Processor: Unknown @ 1.30GHz <span style="color: #f92672;">(</span><span style="color: #ae81ff;">4</span> Cores<span style="color: #f92672;">)</span>, Memory: 4096MB, Disk: 8GB 8GME4R

 

Software:

OS: Ubuntu 16.04, Kernel: 4.4.8-armada-17.02.2-g4126e30 <span style="color: #f92672;">(</span>aarch64<span style="color: #f92672;">)</span>, Compiler: GCC 5.4.0 20160609, File-System: ext4

 

    Would you like to save these test results <span style="color: #f92672;">(</span>Y/n<span style="color: #f92672;">)</span>: n

 

 

HPC Challenge 1.5.0:

    pts/hpcc-1.2.0 <span style="color: #f92672;">[</span>Test / Class: G-HPL<span style="color: #f92672;">]</span>

    Test <span style="color: #ae81ff;">1</span> of <span style="color: #ae81ff;">1</span>

    Estimated Trial Run Count:    <span style="color: #ae81ff;">3</span>

    Estimated Time To Completion: <span style="color: #ae81ff;">1</span> Hour, <span style="color: #ae81ff;">28</span> Minutes

        Started Run <span style="color: #ae81ff;">1</span> @ 19:20:10

        Started Run <span style="color: #ae81ff;">2</span> @ 19:50:34</code></pre></div>

<p>And I guess this gives the reason (from <em>/var/log/messages</em>)&hellip;</p>

<div class="highlight"><pre><code class="language-bash">Feb  <span style="color: #ae81ff;">6</span> 20:18:35 flotta kernel: armada_thermal f06f808c.thermal: Overheat critical high threshold temperature reached</code></pre></div>

<p>Plotting the temperature metrics with <em>gnuplot</em> - we see that we were well in the triple digits.  Oh my!  At this stage,
I should probably stop abusing this poor board and wait until my Noctua industrial fan arrives :)</p>

<figure><img src="https://www.gaborsamu.com/images/armada_gnuplot.jpg" />
</figure>