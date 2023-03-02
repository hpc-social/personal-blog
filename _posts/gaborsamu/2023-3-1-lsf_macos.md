---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2023-03-01 19:10:58'
layout: post
original_url: https://www.gaborsamu.com/blog/lsf_macos/
slug: lsf-client-on-macos-submitting-from-your-laptop
title: LSF client on macOS - submitting from your laptop
---

<p>In traditional HPC environments, login nodes are typically used as an access point for users to submit
and manage jobs. Although login nodes are still used today, HPC environments are
increasingly being used by a broad class of users with domain expertise and not necessarily IT experts.
In other words, such users may be more comfortable using their native desktop
environment rather than the CLI. Given the factors, in the commercial HPC space, organizations are always looking
for ways to lower the barto access and interact with HPC environments.</p>

<p>Spectrum LSF provides many ways to submit and manage jobs in an HPC cluster. For power users, the rich
CLI functionality exists. There is also an available web-based interface for job
submission and management which provides customizable application templates to greatly simplify job sub
mission, while hiding complexity of the underlying infrastructure. A RESTful API
is also available to users of IBM Spectrum LSF Application Center or IBM Spectrum LSF Suites, which ena
bles organizations to access the HPC environment via web services.</p>

<p>I&rsquo;ve written previously in detail about the the LSF web-based interface in the blog
<a href="https://www.gaborsamu.com/blog/easy_hpc/">The Easy HPC Button</a>. Here, we&rsquo;ll take a closer look at the
available LSF client for macOS that uses the RESTful API. First, a bit about LSF clients. LSF clients
can access resources on LSF server hosts without running the LSF daemons. LSF clients don&rsquo;t require a software
license and from clients, users can run all of the familiar LSF commands. Additionally, LSF clients are
submit only, and don&rsquo;t execute jobs.</p>

<p><strong>Note:</strong> The macOS LSF client uses the LSF RESTful API. This means that it will function in environments
running LSF Standard Edition with LSF Application Center or LSF Suites.</p>

<p><strong>Configuration</strong></p>

<p>The configuration used for the example below is as follows:</p>

<table>
<thead>
<tr>
<th style="text-align: left;">Hostname</th>
<th>OS</th>
<th>Detail</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: left;"><em>kilenc</em></td>
<td>CentOS Stream 8.4</td>
<td>LSF Suite for HPC v10.2.0.13</td>
</tr>
<tr>
<td style="text-align: left;"><em>My-Macbook-Air</em></td>
<td>macOS Ventura 13.2.1 (Apple M1)</td>
<td>LSF client</td>
</tr>
</tbody>
</table>
<ol>
<li>On the Spectrum LSF Suite for HPC management host (<em>kilenc</em>), add the following variables to the Parameter
section in the file lsf.cluster.<em>name</em>. The FLOAT_CLIENTS variable determines how many floating clients can
join the LSF cluster, The FLOAT_CLIENTS_ADDR_RANGE specifies the allowable IP addresses. In this case, the
client system is on a 192.168.x.x network.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">Begin Parameters
FLOAT_CLIENTS=2
FLOAT_CLIENTS_ADDR_RANGE=192.*
End Parameters</code></pre></div>

<ol start="2">
<li>To make the changes take effect, issue the following commands as the LSF administrator:</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">lsadmin reconfig
badmin reconfig</code></pre></div>

<ol start="3">
<li>
<p>Obtain the tarball <em>pacdesktop_client10.2.0.13_macos-x86_64.tar</em>. For users with an LSF entitlement this package is available on
<a href="https://www.ibm.com/support/fixcentral/">IBM Fix Central</a>. Note that this package will work on systems with Apple M1 silicon through emulation.</p>

</li>
<li>
<p>Open a Terminal on the macOS client system, copy the tarball to the $HOME/Desktop directory of user lsfuser and uncompress the tarball.</p>

</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">lsfuser@My-MacBook-Air Desktop % pwd
/Users/lsfuser/Desktop
lsfuser@My-MacBook-Air Desktop % ls -la pacdesktop_client10.2.0.13_macos-x86_64.tar
-rw-r--r--@ 1 lsfuser  staff  18452480 27 Feb 17:12 pacdesktop_client10.2.0.13_macos-x86_64.tar
lsfuser@My-MacBook-Air Desktop % tar -xvf pacdesktop_client10.2.0.13_macos-x86_64.tar
x LSF_Desktop_Client/
x LSF_Desktop_Client/bapp
x LSF_Desktop_Client/btop
x LSF_Desktop_Client/bwait
x LSF_Desktop_Client/lseligible
x LSF_Desktop_Client/bsla
x LSF_Desktop_Client/blparams
x LSF_Desktop_Client/bhpart
x LSF_Desktop_Client/bclusters
x LSF_Desktop_Client/blstartup
x LSF_Desktop_Client/lsacct
x LSF_Desktop_Client/bsub
x LSF_Desktop_Client/bugroup
x LSF_Desktop_Client/bpeek
x LSF_Desktop_Client/bacct
x LSF_Desktop_Client/brequeue
x LSF_Desktop_Client/bjgroup
x LSF_Desktop_Client/bslots
x LSF_Desktop_Client/lsrun
x LSF_Desktop_Client/bjobs
x LSF_Desktop_Client/lshosts
x LSF_Desktop_Client/lsload
x LSF_Desktop_Client/brlainfo
x LSF_Desktop_Client/bresources
x LSF_Desktop_Client/bladmin
x LSF_Desktop_Client/bstatus
x LSF_Desktop_Client/bmod
x LSF_Desktop_Client/bpost
x LSF_Desktop_Client/lsid
x LSF_Desktop_Client/bentags
x LSF_Desktop_Client/ch
x LSF_Desktop_Client/bchkpnt
x LSF_Desktop_Client/bparams
x LSF_Desktop_Client/bjdepinfo
x LSF_Desktop_Client/bgmod
x LSF_Desktop_Client/brestart
x LSF_Desktop_Client/lsltasks
x LSF_Desktop_Client/blusers
x LSF_Desktop_Client/paclogon
x LSF_Desktop_Client/regnotify
x LSF_Desktop_Client/cacert.pem
x LSF_Desktop_Client/bresume
x LSF_Desktop_Client/blstat
x LSF_Desktop_Client/bhist
x LSF_Desktop_Client/bqueues
x LSF_Desktop_Client/bltasks
x LSF_Desktop_Client/bresize
x LSF_Desktop_Client/blcollect
x LSF_Desktop_Client/lsacctmrg
x LSF_Desktop_Client/bgadd
x LSF_Desktop_Client/bmig
x LSF_Desktop_Client/bstop
x LSF_Desktop_Client/bswitch
x LSF_Desktop_Client/blhosts
x LSF_Desktop_Client/blcstat
x LSF_Desktop_Client/brsvs
x LSF_Desktop_Client/brun
x LSF_Desktop_Client/blinfo
x LSF_Desktop_Client/lsgrun
x LSF_Desktop_Client/busers
x LSF_Desktop_Client/lsloadadj
x LSF_Desktop_Client/blkill
x LSF_Desktop_Client/bbot
x LSF_Desktop_Client/lsclusters
x LSF_Desktop_Client/bconf
x LSF_Desktop_Client/lsinfo
x LSF_Desktop_Client/lsmake
x LSF_Desktop_Client/blimits
x LSF_Desktop_Client/bmgroup
x LSF_Desktop_Client/bread
x LSF_Desktop_Client/bkill
x LSF_Desktop_Client/lstcsh
x LSF_Desktop_Client/lsrtasks
x LSF_Desktop_Client/README.TXT
x LSF_Desktop_Client/lsplace
x LSF_Desktop_Client/bhosts
x LSF_Desktop_Client/paclogout
x LSF_Desktop_Client/bgdel</code></pre></div>

<ol start="5">
<li>Following the directions in the file README.TXT, set the environment variable LSF_DESKTOP_CLIENT=yes, and set the PATH variable accordingly.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">lsfuser@My-MacBook-Air LSF_Desktop_Client % export LSF_DESKTOP_CLIENT=yes
lsfuser@My-MacBook-Air LSF_Desktop_Client % export PATH=`pwd`:$PATH</code></pre></div>

<ol start="6">
<li>Next, it&rsquo;s necessary to run the <em>paclogon</em> command to connect to the LSF Application Center (or LSF Suite installation). Here we point to the LSF server kilenc on port 8080.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">lsfuser@My-MacBook-Air LSF_Desktop_Client % paclogon
Log on to IBM Spectrum LSF Application Center
User account: lsfuser
Enter password: 
Specify the URL to connect to IBM Spectrum LSF Application Center. Format: 
http://host_name:port_number/platform or https://host_name:port_number/platform
URL: http://kilenc:8080/platform
You have successfully logged on to IBM Spectrum LSF Application Center.</code></pre></div>

<ol start="7">
<li>After successfully logging in using the paclogon command, it should be possible to run LSF &ldquo;base&rdquo; commands from the macOS terminal including <em>lsid</em>, <em>lsload</em>, <em>lshosts</em>.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">lsfuser@My-MacBook-Air LSF_Desktop_Client % lsid
IBM Spectrum LSF 10.1.0.13, Apr 15 2022
Suite Edition: IBM Spectrum LSF Suite for HPC 10.2.0.13
Copyright International Business Machines Corp. 1992, 2016.
US Government Users Restricted Rights - Use, duplication or disclosure restricted by GSA ADP Schedule Contract with IBM Corp.

My cluster name is Klaszter
My master name is kilenc
lsfuser@My-MacBook-Air LSF_Desktop_Client % lshosts -w
HOST_NAME                       type       model  cpuf ncpus maxmem maxswp server RESOURCES
kilenc                    LINUXPPC64LE      POWER9  25.0    32  30.7G  15.8G    Yes (mg docker)
lsfuser@My-MacBook-Air LSF_Desktop_Client % lsload -w
HOST_NAME               status  r15s   r1m  r15m   ut    pg  ls    it   tmp   swp   mem
kilenc                      ok   0.8   2.1   2.4   7%   0.0   0  1156  551M 15.6G   10G</code></pre></div>

<ol start="8">
<li>Next, run the LSF batch commands <em>bqueues</em> and <em>bhosts</em>.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">lsfuser@My-MacBook-Air LSF_Desktop_Client % bqueues
QUEUE_NAME      PRIO STATUS          MAX JL/U JL/P JL/H NJOBS  PEND   RUN  SUSP 
admin            50  Open:Active       -    -    -    -     0     0     0     0
owners           43  Open:Active       -    -    -    -     0     0     0     0
priority         43  Open:Active       -    -    -    - 75835 75803    32     0
night            40  Open:Inact        -    -    -    -     0     0     0     0
short            35  Open:Active       -    -    -    -     0     0     0     0
dataq            33  Open:Active       -    -    -    -     0     0     0     0
normal           30  Open:Active       -    -    -    -     0     0     0     0
interactive      30  Open:Active       -    -    -    -     0     0     0     0
sendq            30  Open:Active       -    -    -    -     0     0     0     0
idle             20  Open:Active       -    -    -    -     0     0     0     0
lsfuser@My-MacBook-Air LSF_Desktop_Client % bhosts
HOST_NAME          STATUS       JL/U    MAX  NJOBS    RUN  SSUSP  USUSP    RSV 
kilenc             ok              -     32     19     19      0      0      0</code></pre></div>

<ol start="9">
<li>Running the bjobs will result in a warning message appearing on macOS stating: <em>&ldquo;bjobs&rdquo; cannot be opened because the developer cannot be verified.</em></li>
</ol>
<figure><img src="https://www.gaborsamu.com/images/bjobs_unverified.png" />
</figure>

<ol start="10">
<li>To remedy the issue observed in step 9, click cancel on the warning message and browse to <strong>System Settings -&gt; Privacy &amp; Security -&gt; Security Settings</strong>. In the Security Settings view,
you&rsquo;ll see the message: <em>&ldquo;bjobs&rdquo; was blocked from use because it is not from an identified developer.</em> To allow the bjobs command to execute, click on the <strong>Allow Anyway</strong> button. You will
then be promped to authenticate to make the change take effect.</li>
</ol>
<p><figure><img src="https://www.gaborsamu.com/images/bjobs_allow.png" />
</figure>

<figure><img src="https://www.gaborsamu.com/images/bjobs_authenticate.png" />
</figure>
</p>

<ol start="11">
<li>Run the LSF <em>bjobs</em> command again. You will now receive a new warning error popup indicating: <em>macOS cannot verify the developer of &ldquo;bjobs&rdquo;. Are you sure you want to open it?</em>. To
proceed, click on the Open button.The bjobs command will then run to completion as expected.  Subsequent executions of bjobs will run without any system warnings. Finally, to submit
a job, run the bsub command. Here we try to submit a simple sleep job (i.e. bsub -q normal sleep 3600). As was the case with the bjobs command, the bsub command is also blocked. Here,
repeat the steps 10, 11 as described above but for the bsub command. Once the steps have been completed, repeat the bsub job submission command.</li>
</ol>
<figure><img src="https://www.gaborsamu.com/images/bjobs_open.png" />
</figure>

<ol start="12">
<li>Finally, to submit a job, run the <em>bsub</em> command. Here we try to submit a simple sleep job (i.e. <em>bsub -q normal sleep 3600</em>). As was the case with the <em>bjobs</em> command, the <em>bsub</em>
command is also blocked. Here, repeat the steps 10, 11 as described above but for the <em>bsub</em> command. Once the steps have been completed, repeat the <em>bsub</em> job submission command.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">lsfuser@My-MacBook-Air LSF_Desktop_Client % bsub -q normal sleep 3600
Job &lt;617551&gt; is submitted to queue &lt;normal&gt;.</code></pre></div>