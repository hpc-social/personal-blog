---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2017-08-31 18:01:46'
layout: post
original_url: https://www.gaborsamu.com/blog/spectrumlsf_armv8/
slug: standing-up-a-ibm-spectrum-lsf-community-edition-cluster-on-arm-v8
title: Standing up a IBM Spectrum LSF Community Edition cluster on Arm v8
---

<p>So, you&rsquo;ve got yourself a shiny new (or maybe not) system based upon a 64-bit Arm (Arm v8) processor that you want to put through it&rsquo;s
paces.  For me, this happens to be a MACCHIATObin board powered by a Marvell ARAMADA 8040 based on ARM Cortex A-72 cores and installed
with Ubuntu 16.04.3 LTS.  You can read about my <a href="https://www.gaborsamu.com/blog/turning_up_heat_armv8/">shenanigans running HPL</a> on my system with a passively cooled CPU and running up
against some overheating conditions - much like the head gasket failure in my car this past summer - but I digress!</p>

<p>While I wait for the Noctua cooling fan to arrive, it&rsquo;s given me the opportunity to revisit installing a job scheduler on the system.
I&rsquo;ll be using this as a way to manage access to the system resources which will be necessary to arbitrate the various benchmark jobs
that I expect to be running over time.  There exists a number of workload schedulers today, from open source to closed source
proprietary.  I&rsquo;ve selected IBM Spectrum LSF Community Edition as it&rsquo;s free to download and use (with restrictions) and supports Linux
on Arm v8.  Did you know that Spectrum LSF (known previously as Platform LSF) has been around for 25 years?  That&rsquo;s quite a pedigree
and because it&rsquo;s shipped as binaries, I won&rsquo;t have to muck about compiling it - which is an added bonus.  To download IBM Spectrum LSF
Community Edition follow the QR code below :)</p>

<figure><img src="https://www.gaborsamu.com/images/LSFcommunity.jpg" />
</figure>

<p>Below I walk through the steps to install IBM Spectrum LSF Community Edition on my Arm v8 based system.  The steps should be the same
for the other platforms supported by IBM Spectrum LSF Community Edition including Linux on POWERLE and Linux on x86-64.  The procedure
below assumes that you have a supported OS installed and have configured networking, and necessary user accounts.  This is not meant
to be an exhaustive tutorial on IBM Spectrum LSF Community Edition.  If you&rsquo;re looking for help, check out the forum <a href="https://ibm.biz/LSF_UserGroup">here</a>.</p>

<p><strong>1. Download and extract</strong></p>

<p>We begin by downloading the <em>armv8</em> IBM Spectrum LSF Community Edition package and quick start guide.  We expand the gzipped tarball
to get the installer and &ldquo;armv8&rdquo; binary compressed tarballs.  Next, we extract the <em>lsfinstall</em> tarball.  This contains the installer
for IBM Spectrum LSF Community Edition.</p>

<div class="highlight"><pre><code class="language-bash">root@flotta:/tmp# ls
lsfce10.1-armv8.tar.gz  lsfce10.1_quick_start.pdf
root@flotta:/tmp# gunzip lsfce10.1-armv8.tar.gz 
root@flotta:/tmp# tar -xvf lsfce10.1-armv8.tar 
lsfce10.1-armv8/
lsfce10.1-armv8/lsf/
lsfce10.1-armv8/lsf/lsf10.1_lnx312-lib217-armv8.tar.Z
lsfce10.1-armv8/lsf/lsf10.1_no_jre_lsfinstall.tar.Z

root@flotta:/tmp/lsfce10.1-armv8/lsf# zcat lsf10.1_no_jre_lsfinstall.tar.Z | tar xvf -
lsf10.1_lsfinstall/
lsf10.1_lsfinstall/instlib/
lsf10.1_lsfinstall/instlib/lsflib.sh
lsf10.1_lsfinstall/instlib/lsferror.tbl
lsf10.1_lsfinstall/instlib/lsfprechkfuncs.sh
lsf10.1_lsfinstall/instlib/lsflicensefuncs.sh
lsf10.1_lsfinstall/instlib/lsfunpackfuncs.sh
lsf10.1_lsfinstall/instlib/lsfconfigfuncs.sh
lsf10.1_lsfinstall/instlib/resconnectorconfigfuncs.sh
lsf10.1_lsfinstall/instlib/lsf_getting_started.tmpl
....
....</code></pre></div>

<p><strong>2. Configure the installer</strong></p>

<p>After extracting the <em>lsfinstall</em> tarball, you&rsquo;ll find the installation configuration file <em>install.config</em>.  This file controls the
installation location, LSF administrator account, name of cluster, master node (where scheduler daemon runs), location of binary
source packages among other things.  I&rsquo;ve run a diff here to show the settings.  In brief, I&rsquo;ve configured the following:</p>

<ul>
<li>installation location: <em>/raktar/LSFCE</em></li>
<li>LSF administrator account: <em>gsamu</em></li>
<li>LSF cluster name: <em>Klaszter</em></li>
<li>scheduler node: <em>flotta</em></li>
<li>location of LSF binary source packages: <em>/tmp/lsfce10.1-armv8/lsf</em> (here is located <em>lsf10.1_lnx312-lib217-armv8.tar.Z</em> from step 1)</li>
</ul>
<div class="highlight"><pre><code class="language-bash">root@flotta:/tmp/lsfce10.1-armv8/lsf/lsf10.1_lsfinstall# diff -u4 install.config install.config.org 
--- install.config 2017-08-30 20:17:13.148583971 -0400
+++ install.config.org 2017-08-30 20:15:30.283904454 -0400
@@ -40,9 +40,8 @@
 <span style="color: #75715e;">#     (During an upgrade, specify the existing value.)</span>
 <span style="color: #75715e;">#**********************************************************</span>
 <span style="color: #75715e;"># -----------------</span>
 <span style="color: #75715e;"># LSF_TOP="/usr/share/lsf"</span>
-LSF_TOP<span style="color: #f92672;">=</span><span style="color: #e6db74;">"/raktar/LSFCE"</span>
 <span style="color: #75715e;"># -----------------</span>
 <span style="color: #75715e;"># Full path to the top-level installation directory {REQUIRED}</span>
 <span style="color: #75715e;">#</span>
 <span style="color: #75715e;"># The path to LSF_TOP must be shared and accessible to all hosts</span>
@@ -51,9 +50,8 @@
 <span style="color: #75715e;"># all host types (approximately 300 MB per host type).</span>
 <span style="color: #75715e;">#</span>
 <span style="color: #75715e;"># -----------------</span>
 <span style="color: #75715e;"># LSF_ADMINS="lsfadmin user1 user2"</span>
-LSF_ADMINS<span style="color: #f92672;">=</span><span style="color: #e6db74;">"gsamu"</span>
 <span style="color: #75715e;"># -----------------</span>
 <span style="color: #75715e;"># List of LSF administrators {REQUIRED}</span>
 <span style="color: #75715e;">#</span>
 <span style="color: #75715e;"># The first user account name in the list is the primary LSF</span>
@@ -69,9 +67,8 @@
 <span style="color: #75715e;"># Secondary LSF administrators are optional.</span>
 <span style="color: #75715e;">#</span>
 <span style="color: #75715e;"># -----------------</span>
 <span style="color: #75715e;"># LSF_CLUSTER_NAME="cluster1"</span>
-LSF_CLUSTER_NAME<span style="color: #f92672;">=</span><span style="color: #e6db74;">"Klaszter"</span>
 <span style="color: #75715e;"># -----------------</span>
 <span style="color: #75715e;"># Name of the LSF cluster {REQUIRED}</span>
 <span style="color: #75715e;">#</span>
 <span style="color: #75715e;"># It must be 39 characters or less, and cannot contain any</span>
@@ -85,9 +82,8 @@
 <span style="color: #75715e;">#**********************************************************</span>
 <span style="color: #75715e;">#</span>
 <span style="color: #75715e;"># -----------------</span>
 <span style="color: #75715e;"># LSF_MASTER_LIST="hostm hosta hostc"</span>
-LSF_MASTER_LIST<span style="color: #f92672;">=</span><span style="color: #e6db74;">"flotta"</span>
 <span style="color: #75715e;"># -----------------</span>
 <span style="color: #75715e;"># List of LSF server hosts to be master or master candidate in the</span>
 <span style="color: #75715e;"># cluster {REQUIRED when you install for the first time or during</span>
 <span style="color: #75715e;"># upgrade if the parameter does not already exist.}</span>
@@ -96,9 +92,8 @@
 <span style="color: #75715e;"># cluster. The first host listed is the LSF master host.</span>
 <span style="color: #75715e;">#</span>
 <span style="color: #75715e;"># -----------------</span>
 <span style="color: #75715e;"># LSF_TARDIR="/usr/share/lsf_distrib/"</span>
-LSF_TARDIR<span style="color: #f92672;">=</span><span style="color: #e6db74;">"/tmp/lsfce10.1-armv8/lsf"</span>
 <span style="color: #75715e;"># -----------------</span>
 <span style="color: #75715e;"># Full path to the directory containing the LSF distribution tar files.</span>
 <span style="color: #75715e;">#</span>
 <span style="color: #75715e;"># Default: Parent directory of the current working directory.</span></code></pre></div>

<p><strong>3. Install IBM Spectrum LSF Community Edition</strong></p>

<p>With the installation configuration complete, we can now invoke the installer script.  Note that I had to install JRE on my system
(in my case <em>apt-get install default-jre</em>) as it&rsquo;s a requirement for IBM Spectrum LSF Community Edition.</p>

<div class="highlight"><pre><code class="language-bash">root@flotta:/tmp/lsfce10.1-armv8/lsf/lsf10.1_lsfinstall# ./lsfinstall -f ./install.config

Logging installation sequence in /tmp/lsfce10.1-armv8/lsf/lsf10.1_lsfinstall/Install.log

International License Agreement <span style="color: #66d9ef;">for</span> Non-Warranted Programs

Part <span style="color: #ae81ff;">1</span> - General Terms

BY DOWNLOADING, INSTALLING, COPYING, ACCESSING, CLICKING ON 
AN <span style="color: #e6db74;">"ACCEPT"</span> BUTTON, OR OTHERWISE USING THE PROGRAM, 
LICENSEE AGREES TO THE TERMS OF THIS AGREEMENT. IF YOU ARE 
ACCEPTING THESE TERMS ON BEHALF OF LICENSEE, YOU REPRESENT 
AND WARRANT THAT YOU HAVE FULL AUTHORITY TO BIND LICENSEE 
TO THESE TERMS. IF YOU DO NOT AGREE TO THESE TERMS,

* DO NOT DOWNLOAD, INSTALL, COPY, ACCESS, CLICK ON AN 
<span style="color: #e6db74;">"ACCEPT"</span> BUTTON, OR USE THE PROGRAM; AND

* PROMPTLY RETURN THE UNUSED MEDIA AND DOCUMENTATION TO THE 

Press Enter to <span style="color: #66d9ef;">continue</span> viewing the license agreement, or 
enter <span style="color: #e6db74;">"1"</span> to accept the agreement, <span style="color: #e6db74;">"2"</span> to decline it, <span style="color: #e6db74;">"3"</span> 
to print it, <span style="color: #e6db74;">"4"</span> to read non-IBM terms, or <span style="color: #e6db74;">"99"</span> to go back 
to the previous screen.
<span style="color: #ae81ff;">1</span>
LSF pre-installation check ...

Checking the LSF TOP directory /raktar/LSFCE ...
... Done checking the LSF TOP directory /raktar/LSFCE ...
You are installing IBM Spectrum LSF - 10.1 Community Edition.

Checking LSF Administrators ...
   LSF administrator<span style="color: #f92672;">(</span>s<span style="color: #f92672;">)</span>:       <span style="color: #e6db74;">"gsamu"</span>
   Primary LSF administrator:  <span style="color: #e6db74;">"gsamu"</span>
Checking the configuration template  ...
    Done checking configuration template ...
    Done checking ENABLE_STREAM ...

<span style="color: #f92672;">[</span>Wed Aug <span style="color: #ae81ff;">30</span> 20:36:46 EDT 2017:lsfprechk:WARN_2007<span style="color: #f92672;">]</span>
    Hosts defined in LSF_MASTER_LIST must be LSF server hosts. The
    following hosts will be added to server hosts automatically: flotta.

Checking the patch history directory  ...
Creating /raktar/LSFCE/patch ...
... Done checking the patch history directory /raktar/LSFCE/patch ...

Checking the patch backup directory ...
... Done checking the patch backup directory /raktar/LSFCE/patch/backup ...


Searching LSF 10.1 distribution tar files in /tmp/lsfce10.1-armv8/lsf Please wait ...

  1<span style="color: #f92672;">)</span> linux3.12-glibc2.17-armv8

Press <span style="color: #ae81ff;">1</span> or Enter to install this host type: <span style="color: #ae81ff;">1</span>

You have chosen the following tar file<span style="color: #f92672;">(</span>s<span style="color: #f92672;">)</span>:
    lsf10.1_lnx312-lib217-armv8

Checking selected tar file<span style="color: #f92672;">(</span>s<span style="color: #f92672;">)</span> ...
... Done checking selected tar file<span style="color: #f92672;">(</span>s<span style="color: #f92672;">)</span>.


Pre-installation check report saved as text file: 
/tmp/lsfce10.1-armv8/lsf/lsf10.1_lsfinstall/prechk.rpt.

... Done LSF pre-installation check.

Installing LSF binary files <span style="color: #e6db74;">" lsf10.1_lnx312-lib217-armv8"</span>...
Creating /raktar/LSFCE/10.1 ...

Copying lsfinstall files to /raktar/LSFCE/10.1/install
Creating /raktar/LSFCE/10.1/install ...

....

....

lsfinstall is <span style="color: #66d9ef;">done</span>.

To complete your LSF installation and get your 
cluster <span style="color: #e6db74;">"Klaszter"</span> up and running, follow the steps in 
<span style="color: #e6db74;">"/tmp/lsfce10.1-armv8/lsf/lsf10.1_lsfinstall/lsf_getting_started.html"</span>.

After setting up your LSF server hosts and verifying 
your cluster <span style="color: #e6db74;">"Klaszter"</span> is running correctly, 
see <span style="color: #e6db74;">"/raktar/LSFCE/10.1/lsf_quick_admin.html"</span> 
to learn more about your new LSF cluster.

After installation, remember to bring your cluster up to date 
by applying the latest updates and bug fixes. </code></pre></div>

<p><strong>4. Siesta time!</strong></p>

<p>Wow, that was easy. IBM Spectrum LSF Community Edition is now installed.  Pat yourself on the back and grab your favourite BEvERage
as a reward.</p>

<p><strong>5. Fire it up!</strong></p>

<p>Now that IBM Spectrum LSF Community Edition is installed, we can start it up so that it&rsquo;s ready to accept and manage work!  As the root
user we source the environment for IBM Spectrum LSF Community Edition which sets the PATH and other needed environment variables. Next,
we issue 3 commands to start up the IBM Spectrum LSF Community Edition daemons.</p>

<div class="highlight"><pre><code class="language-bash">root@flotta:/raktar/LSFCE/conf# . ./profile.lsf

root@flotta:/raktar/LSFCE/conf# lsadmin limstartup
Starting up LIM on &lt;flotta.localdomain&gt; ...... <span style="color: #66d9ef;">done</span>
root@flotta:/raktar/LSFCE/conf# lsadmin resstartup
Starting up RES on &lt;flotta.localdomain&gt; ...... <span style="color: #66d9ef;">done</span>
root@flotta:/raktar/LSFCE/conf# badmin hstartup
Starting up slave batch daemon on &lt;flotta.localdomain&gt; ...... <span style="color: #66d9ef;">done</span></code></pre></div>

<p>With IBM Spectrum LSF Community Edition now running, we should be able to query the cluster for status. Note that we&rsquo;ve setup a single
node cluster. IBM Spectrum LSF Community Edition allows you to build up clusters with up to 10 nodes. We run a series of commands to
check if the cluster is alive and well.</p>

<div class="highlight"><pre><code class="language-bash">root@flotta:/raktar/LSFCE/conf# lsid
IBM Spectrum LSF Community Edition 10.1.0.0, Jun <span style="color: #ae81ff;">15</span> <span style="color: #ae81ff;">2016</span>
Copyright IBM Corp. 1992, 2016. All rights reserved.
US Government Users Restricted Rights - Use, duplication or disclosure restricted by GSA ADP Schedule Contract with IBM Corp.

My cluster name is Klaszter
My master name is flotta.localdomain

 

root@flotta:/raktar/LSFCE/conf# lsload
HOST_NAME       status  r15s   r1m  r15m   ut    pg  ls    it   tmp   swp   mem
flotta.localdom     ok   0.0   0.3   0.4  13%   0.0   <span style="color: #ae81ff;">1</span>     <span style="color: #ae81ff;">0</span> 1444M    0M  3.3

 

root@flotta:/raktar/LSFCE/conf# bhosts
HOST_NAME          STATUS       JL/U    MAX  NJOBS    RUN  SSUSP  USUSP    RSV 
flotta.localdomain ok              -      <span style="color: #ae81ff;">4</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span></code></pre></div>

<p>The above commands show that the IBM Spectrum LSF Community Edition cluster is up and running. The batch system is now ready to accept
workload!</p>

<p><strong>6. Now what?</strong></p>

<p>We are ready to rock n' roll!  To christen this environment, I decided to run some MPI tests.  Coincidentally, MPI is also celebrating
a silver anniversary this year.</p>

<p>And what better MPI tests to run on my Arm system than the <a href="https://software.intel.com/en-us/articles/intel-mpi-benchmarks">Intel MPI Benchmarks</a> :)  Of course, the Intel MPI Benchmarks have to be
compiled.  To keep things simple, I only compiled the MPI1 benchmark set.  This required me to change the CC designation in the
make_ict makefie from mpiicc to mpicc (as I am obviously not using Intel Compilers).</p>

<div class="highlight"><pre><code class="language-bash">root@flotta:/raktar/imb/imb/src# gmake -f make_ict IMB-MPI1
sleep 1; touch exe_mpi1 *.c; rm -rf exe_io exe_ext  exe_nbc exe_rma
gmake -f Makefile.base MPI1 CPP<span style="color: #f92672;">=</span>MPI1
gmake<span style="color: #f92672;">[</span>1<span style="color: #f92672;">]</span>: Entering directory <span style="color: #e6db74;">'/raktar/imb/imb/src'</span>
mpicc    -DMPI1  -c IMB.c
mpicc    -DMPI1  -c IMB_utils.c
mpicc    -DMPI1  -c IMB_declare.c
mpicc    -DMPI1  -c IMB_init.c
mpicc    -DMPI1  -c IMB_mem_manager.c
mpicc    -DMPI1  -c IMB_parse_name_mpi1.c
mpicc    -DMPI1  -c IMB_benchlist.c
mpicc    -DMPI1  -c IMB_strgs.c
mpicc    -DMPI1  -c IMB_err_handler.c
mpicc    -DMPI1  -c IMB_g_info.c
mpicc    -DMPI1  -c IMB_warm_up.c
mpicc    -DMPI1  -c IMB_output.c
mpicc    -DMPI1  -c IMB_pingpong.c
mpicc    -DMPI1  -c IMB_pingping.c
mpicc    -DMPI1  -c IMB_allreduce.c

....

....
mpicc    -o IMB-MPI1 IMB.o IMB_utils.o IMB_declare.o  IMB_init.o IMB_mem_manager.o IMB_parse_name_mpi1.o  IMB_benchlist.o IMB_strgs.o IMB_err_handler.o IMB_g_info.o  IMB_warm_up.o IMB_output.o IMB_pingpong.o IMB_pingping.o IMB_allreduce.o IMB_reduce_scatter.o IMB_reduce.o IMB_exchange.o IMB_bcast.o IMB_barrier.o IMB_allgather.o IMB_allgatherv.o IMB_gather.o IMB_gatherv.o IMB_scatter.o IMB_scatterv.o IMB_alltoall.o IMB_alltoallv.o IMB_sendrecv.o IMB_init_transfer.o  IMB_chk_diff.o IMB_cpu_exploit.o IMB_bandwidth.o   
gmake<span style="color: #f92672;">[</span>1<span style="color: #f92672;">]</span>: Leaving directory <span style="color: #e6db74;">'/raktar/imb/imb/src'</span></code></pre></div>

<p>So we have our MPI benchmark compiled and our workload scheduler up and running.  Let&rsquo;s get busy! As user <em>gsamu</em> we source the
environment for IBM Spectrum LSF Community Edition, and submit a 4-way instance of the Intel MPI Benchmark MPI1 test suite for
execution on our cluster.</p>

<div class="highlight"><pre><code class="language-bash">gsamu@flotta:/$ . /raktar/LSFCE/conf/profile.lsf 
gsamu@flotta:/$ lsid
IBM Spectrum LSF Community Edition 10.1.0.0, Jun <span style="color: #ae81ff;">15</span> <span style="color: #ae81ff;">2016</span>
Copyright IBM Corp. 1992, 2016. All rights reserved.
US Government Users Restricted Rights - Use, duplication or disclosure restricted by GSA ADP Schedule Contract with IBM Corp.

My cluster name is Klaszter
My master name is flotta.localdomain</code></pre></div>

<p>Drumroll please&hellip;The MPI benchmark runs through successfully.  Note that we&rsquo;ve submitted the job to IBM Spectrum LSF Community Edition
interactively - with the -I parameter.  Jobs can also be run non-interactively and users can peek at the standard output during
runtime using the bpeek command.</p>

<div class="highlight"><pre><code class="language-bash">gsamu@flotta:/$ bsub -I -q interactive -n <span style="color: #ae81ff;">4</span> mpirun -np <span style="color: #ae81ff;">4</span> /raktar/imb/imb/src/IMB-MPI1
Job &lt;1396&gt; is submitted to queue &lt;interactive&gt;.
&lt;&lt;Waiting <span style="color: #66d9ef;">for</span> dispatch ...&gt;&gt;
<span style="color: #e6db74;">&lt;&lt;Starting on flotta.localdomain&gt;&gt;
</span><span style="color: #e6db74;">#------------------------------------------------------------
</span><span style="color: #e6db74;">#    Intel (R) MPI Benchmarks 2017 update 2, MPI-1 part    
</span><span style="color: #e6db74;">#------------------------------------------------------------
</span><span style="color: #e6db74;"># Date                  : Thu Aug 30 21:46:28 2017
</span><span style="color: #e6db74;"># Machine               : aarch64
</span><span style="color: #e6db74;"># S</span>ystem                : Linux
<span style="color: #75715e;"># Release               : 4.4.8-armada-17.02.2-g4126e30</span>
<span style="color: #75715e;"># Version               : #1 SMP PREEMPT Sat May 27 18:52:53 CDT 2017</span>
<span style="color: #75715e;"># MPI Version           : 3.0</span>
<span style="color: #75715e;"># MPI Thread Environment: </span>


<span style="color: #75715e;"># Calling sequence was: </span>

<span style="color: #75715e;"># /raktar/imb/imb/src/IMB-MPI1</span>

<span style="color: #75715e;"># Minimum message length in bytes:   0</span>
<span style="color: #75715e;"># Maximum message length in bytes:   4194304</span>
<span style="color: #75715e;">#</span>
<span style="color: #75715e;"># MPI_Datatype                   :   MPI_BYTE </span>
<span style="color: #75715e;"># MPI_Datatype for reductions    :   MPI_FLOAT</span>
<span style="color: #75715e;"># MPI_Op                         :   MPI_SUM  </span>
<span style="color: #75715e;">#</span>
<span style="color: #75715e;">#</span>

<span style="color: #75715e;"># List of Benchmarks to run:</span>

<span style="color: #75715e;"># PingPong</span>
<span style="color: #75715e;"># PingPing</span>
<span style="color: #75715e;"># Sendrecv</span>
<span style="color: #75715e;"># Exchange</span>
<span style="color: #75715e;"># Allreduce</span>
<span style="color: #75715e;"># Reduce</span>
<span style="color: #75715e;"># Reduce_scatter</span>
<span style="color: #75715e;"># Allgather</span>
<span style="color: #75715e;"># Allgatherv</span>
<span style="color: #75715e;"># Gather</span>
<span style="color: #75715e;"># Gatherv</span>
<span style="color: #75715e;"># Scatter</span>
<span style="color: #75715e;"># Scatterv</span>
<span style="color: #75715e;"># Alltoall</span>
<span style="color: #75715e;"># Alltoallv</span>
<span style="color: #75715e;"># Bcast</span>
<span style="color: #75715e;"># Barrier</span>

<span style="color: #75715e;">#---------------------------------------------------</span>
<span style="color: #75715e;"># Benchmarking PingPong </span>
<span style="color: #75715e;"># #processes = 2 </span>
<span style="color: #75715e;"># ( 2 additional processes waiting in MPI_Barrier)</span>
<span style="color: #75715e;">#---------------------------------------------------</span>
       <span style="color: #75715e;">#bytes #repetitions      t[usec]   Mbytes/sec</span>
            <span style="color: #ae81ff;">0</span>         <span style="color: #ae81ff;">1000</span>         1.03         0.00
            <span style="color: #ae81ff;">1</span>         <span style="color: #ae81ff;">1000</span>         1.17         0.85
            <span style="color: #ae81ff;">2</span>         <span style="color: #ae81ff;">1000</span>         1.18         1.69
            <span style="color: #ae81ff;">4</span>         <span style="color: #ae81ff;">1000</span>         1.19         3.36
            <span style="color: #ae81ff;">8</span>         <span style="color: #ae81ff;">1000</span>         0.70        11.43
           <span style="color: #ae81ff;">16</span>         <span style="color: #ae81ff;">1000</span>         0.67        23.89
           <span style="color: #ae81ff;">32</span>         <span style="color: #ae81ff;">1000</span>         0.68        47.37
           <span style="color: #ae81ff;">64</span>         <span style="color: #ae81ff;">1000</span>         0.70        90.84
          <span style="color: #ae81ff;">128</span>         <span style="color: #ae81ff;">1000</span>         0.72       176.78
          <span style="color: #ae81ff;">256</span>         <span style="color: #ae81ff;">1000</span>         0.80       319.61
          <span style="color: #ae81ff;">512</span>         <span style="color: #ae81ff;">1000</span>         1.12       455.51
         <span style="color: #ae81ff;">1024</span>         <span style="color: #ae81ff;">1000</span>         1.89       540.52
         <span style="color: #ae81ff;">2048</span>         <span style="color: #ae81ff;">1000</span>         2.20       932.37
         <span style="color: #ae81ff;">4096</span>         <span style="color: #ae81ff;">1000</span>         3.93      1042.37
         <span style="color: #ae81ff;">8192</span>         <span style="color: #ae81ff;">1000</span>         5.93      1380.77
        <span style="color: #ae81ff;">16384</span>         <span style="color: #ae81ff;">1000</span>         8.76      1869.69
        <span style="color: #ae81ff;">32768</span>         <span style="color: #ae81ff;">1000</span>        14.92      2195.65
        <span style="color: #ae81ff;">65536</span>          <span style="color: #ae81ff;">640</span>        24.37      2689.26
       <span style="color: #ae81ff;">131072</span>          <span style="color: #ae81ff;">320</span>        41.37      3168.39
       <span style="color: #ae81ff;">262144</span>          <span style="color: #ae81ff;">160</span>        81.48      3217.12
       <span style="color: #ae81ff;">524288</span>           <span style="color: #ae81ff;">80</span>       193.81      2705.22
      <span style="color: #ae81ff;">1048576</span>           <span style="color: #ae81ff;">40</span>       443.74      2363.05
      <span style="color: #ae81ff;">2097152</span>           <span style="color: #ae81ff;">20</span>       860.30      2437.71
      <span style="color: #ae81ff;">4194304</span>           <span style="color: #ae81ff;">10</span>      1692.45      2478.24

<span style="color: #75715e;">#---------------------------------------------------</span>
<span style="color: #75715e;"># Benchmarking PingPing </span>
<span style="color: #75715e;"># #processes = 2 </span>
<span style="color: #75715e;"># ( 2 additional processes waiting in MPI_Barrier)</span>
<span style="color: #75715e;">#---------------------------------------------------</span>
       <span style="color: #75715e;">#bytes #repetitions      t[usec]   Mbytes/sec</span>
            <span style="color: #ae81ff;">0</span>         <span style="color: #ae81ff;">1000</span>         0.81         0.00
            <span style="color: #ae81ff;">1</span>         <span style="color: #ae81ff;">1000</span>         0.85         1.17
            <span style="color: #ae81ff;">2</span>         <span style="color: #ae81ff;">1000</span>         0.85         2.34
            <span style="color: #ae81ff;">4</span>         <span style="color: #ae81ff;">1000</span>         0.85         4.68
            <span style="color: #ae81ff;">8</span>         <span style="color: #ae81ff;">1000</span>         0.86         9.34
           <span style="color: #ae81ff;">16</span>         <span style="color: #ae81ff;">1000</span>         0.89        17.96
           <span style="color: #ae81ff;">32</span>         <span style="color: #ae81ff;">1000</span>         0.89        35.99
           <span style="color: #ae81ff;">64</span>         <span style="color: #ae81ff;">1000</span>         0.92        69.71
          <span style="color: #ae81ff;">128</span>         <span style="color: #ae81ff;">1000</span>         0.97       132.63
          <span style="color: #ae81ff;">256</span>         <span style="color: #ae81ff;">1000</span>         1.01       254.50
          <span style="color: #ae81ff;">512</span>         <span style="color: #ae81ff;">1000</span>         1.31       391.16
         <span style="color: #ae81ff;">1024</span>         <span style="color: #ae81ff;">1000</span>         2.45       418.61
         <span style="color: #ae81ff;">2048</span>         <span style="color: #ae81ff;">1000</span>         3.06       670.15
         <span style="color: #ae81ff;">4096</span>         <span style="color: #ae81ff;">1000</span>         5.27       776.63
         <span style="color: #ae81ff;">8192</span>         <span style="color: #ae81ff;">1000</span>         8.09      1012.73
        <span style="color: #ae81ff;">16384</span>         <span style="color: #ae81ff;">1000</span>        11.56      1417.31
        <span style="color: #ae81ff;">32768</span>         <span style="color: #ae81ff;">1000</span>        18.10      1810.29
        <span style="color: #ae81ff;">65536</span>          <span style="color: #ae81ff;">640</span>        31.07      2108.97
       <span style="color: #ae81ff;">131072</span>          <span style="color: #ae81ff;">320</span>        67.01      1955.95
       <span style="color: #ae81ff;">262144</span>          <span style="color: #ae81ff;">160</span>       134.24      1952.73
       <span style="color: #ae81ff;">524288</span>           <span style="color: #ae81ff;">80</span>       358.88      1460.91
      <span style="color: #ae81ff;">1048576</span>           <span style="color: #ae81ff;">40</span>       895.85      1170.49
      <span style="color: #ae81ff;">2097152</span>           <span style="color: #ae81ff;">20</span>      1792.75      1169.79
      <span style="color: #ae81ff;">4194304</span>           <span style="color: #ae81ff;">10</span>      3373.79      1243.20

....

....

<span style="color: #75715e;"># All processes entering MPI_Finalize</span></code></pre></div>

<p>There you have it.  if you&rsquo;re after more information about IBM Spectrum LSF, visit <a href="https://www.ibm.com/us-en/marketplace/hpc-workload-management">here</a>. .</p>