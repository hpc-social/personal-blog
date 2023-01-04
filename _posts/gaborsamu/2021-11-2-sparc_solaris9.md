---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2021-11-03 00:49:40'
layout: post
original_url: https://www.gaborsamu.com/blog/sparc_solaris9/
slug: weekend-it-sparc-eology
title: Weekend IT SPARC-eology
---

<p>Well it&rsquo;s that time of the year again in the northern hemisphere where the weather starts to change and the autumn
colours are in full swing. And the gray, rainy weekends have left me looking for - something to provide a spark.
And there’s no better way than revisiting some of my old hobby IT projects which have been languishing in my basement.
This time I decided to turn my attention to something SPARC powered. And before you wonder which SUN Microsystems
server or desktop, this blog is actually about a laptop - an UltraSPARC IIe based laptop.</p>

<p>In the mid-2000&rsquo;s I managed to pick-up a laptop curiosity. Up to that point, I only had x86_64, and PowerPC-based
laptops. In case you&rsquo;re wondering, the PowerPC based laptops were of course from Apple. When I came across the blue and
grey coloured Naturetech 777 laptop, I simply couldn&rsquo;t resist. After all, my daily driver from 2000-2005 doing tech
support at Platform Computing was a SUN Microsystems Ultra 5 with a SUN PCi card. It served me well over those years,
albeit being somewhat crippled by having poor IO performance.</p>

<p>I&rsquo;ve written about the Naturetech 777 before and even posted some rather shaky videos to YouTube. You can find all of
the links in the blog <a href="https://www.gaborsamu.com/blog/ultrasparc_laptop/">UltraSPARC powered laptop - circa 2001</a>.
There&rsquo;s not too much information available about the Naturetech SPARC laptops. <a href="https://www.cnet.com/news/pc-maker-ships-sun-based-workstation/">Here is one write-up</a> I found from early
2002 with some information. If you’re after more details just let me know. Otherwise, your favourite search engine is
your friend.</p>

<p>So what was the goal here with this IT archeology weekend project? Well it was to get the latest version of Solaris 9
installed - which is Update 9. Although the system had a working copy of Solaris 9 already installed, it&rsquo;s an
older update. Furthermore, the system was configured to point to the now defunct Blastwave CSW open-source software
repositry.</p>

<p>Solaris 9 already installed, it was an older update. Furthermore, the system was configured to point to the now
defunct Blastwave CSW software repository. And I wanted to configure the system to use <a href="https://www.opencsw.org/">OpenCSW</a>
for which the latest Solaris 9 update is recommended.</p>

<p>On the surface, this seemed an easy enough project. After all, I had the Solaris 9 U9 DVD media in hand. I had
previously installed an older update of Solaris 9 on the system.  What could possibly go wrong? As Murphy’s law would
have it, the Solaris 9 Update 9 media simply refused to boot on the laptop.</p>

<div class="highlight"><pre><code class="language-plaintext">GENIALstation 777S (UltraSPARC-Ile 500MHz), Keyboard Present
OpenBoot 4.0.2.12, 256 MB memory installed, Serial #12648430.
Ethernet address 8:0:20:13: de: ad, Host ID: 00c0ffee.

ok boot cdrom

Boot device: /pcielf,0/pci@1, 1/ide@d/cdrom@2,0: f Short disk read File and args:

The file just loaded does not appear to be executable. </code></pre></div>

<p>The only clue I could come up with was that the likely culprit was the old OpenBOOT version,
which I really didn&rsquo;t even want to explore updating at this stage out of fear of bricking the system.
At this point I was ready to throw in the towel. What do you do with a system from early 2000 that simply refuses
to boot the OS media?  Well much like an old car, you jumpstart it!</p>

<p>A bit of digging showed that it&rsquo;s possible to do a Jumpstart installation of Solaris from Linux or NetBSD.
There were a number of very good resources to help with the setup of this.  In particular I used the
following for reference:</p>

<ul>
<li><a href="http://www.pbandjelly.org/2005/07/solaris-10-jumpstart-from-freebsd/comment-page-1/">Solaris 10 Jumpstart from FreeBSD</a></li>
<li><a href="http://www.asgaur.com/wp/solaris-jumpstart-from-a-linux-server/">Solaris jumpstart from a Linux server</a></li>
</ul>
<p>Surprisingly a few tweets of my efforts, significantly helped by retweets from some folks including <a href="https://twitter.com/PCzanik">Peter Czanik</a> resulted in a lot of attention on the interwebs. And a fellow IBMer, <a href="https://twitter.com/CrivetiMihai">Mihai Criveti</a> as well offered some useful tips which helped me get through some of the jumpstart challenges.</p>

<p>So following the above articles, I worked to setup all of the necessary services and configuration that was required.
For brevity and to avoid repetition here is a brief rundown. Note that the jumpstart server used is running
NetBSD 9.2/amd64.</p>

<p><!-- raw HTML omitted -->1.<!-- raw HTML omitted --> Configure the MAC address of the jumpstart client in <em>/etc/ethers</em> on the server.
Note that in my case because of a bad NVRAM battery, I need to always set the MAC address manually at the
OpenBOOT prompt. For this I use the procedure described <a href="http://www.alyon.org/InfosTechniques/informatique/SunHardwareReference/sun-nvram-hostid.faq">here</a>.</p>

<div class="highlight"><pre><code class="language-plaintext"># cat /etc/ethers
08:00:20:13:de:ad  sparc</code></pre></div>

<p><!-- raw HTML omitted -->2.<!-- raw HTML omitted --> Configure an IP that will be assigned to the jumpstart client in <em>/etc/hosts</em> on the server.</p>

<div class="highlight"><pre><code class="language-plaintext"># cat /etc/hosts
...
...
::1                     localhost localhost.
127.0.0.1               localhost localhost.
192.168.1.187 sparc 
...
...</code></pre></div>

<p><!-- raw HTML omitted -->3.<!-- raw HTML omitted --> Next, we require to mount the Solaris 9 ISO. This will ultimately be NFS exported for the jumpstart client.
We also require to copy the appropriate Solaris <em>inetboot</em> file from the Solaris 9 media. This will be delivered via
tftpd to the jumpstart client.</p>

<div class="highlight"><pre><code class="language-plaintext"># ls -la sol-9-905hw-ga-sparc-dvd.iso
-rw-r--r--  1 root  wheel  3104112640 Oct 30 00:37 sol-9-905hw-ga-sparc-dvd.iso
# vnconfig vnd0 sol-9-905hw-ga-sparc-dvd.iso
# mount -t cd9660 /dev/vnd0a /data/jumpstart/sol9u9</code></pre></div>

<p><!-- raw HTML omitted -->4.<!-- raw HTML omitted --> The Solaris 9 media includes multiple versions of <em>inetboot</em>. You require to select the one that’s appropriate for
your jumpstart client system. In this case, we use <em>inetboot</em> for the sun4u architecture as the system is UltraSPARC
IIe based. The file is copied to the <em>/tftpboot</em> directory, which will be used by <em>tftpd</em>.</p>

<div class="highlight"><pre><code class="language-plaintext"># cd /data/jumpstart/sol9u9
# find ./ -name inetboot -print
./Solaris_9/Tools/Boot/usr/platform/sun4m/lib/fs/nfs/inetboot
./Solaris_9/Tools/Boot/usr/platform/sun4u/lib/fs/nfs/inetboot
# cp ./Solaris_9/Tools/Boot/usr/platform/sun4u/lib/fs/nfs/inetboot /tftpboot</code></pre></div>

<p><!-- raw HTML omitted -->5.<!-- raw HTML omitted --> The request that will be made by the jumpstart client to the tftpd will be for a filename with the IP address of
the client in hexadecimal. The IP address of the client we’ve specified in <em>/etc/hosts</em> is 192.168.1.187. So we convert
that to a hexadecimal string as follows:</p>

<div class="highlight"><pre><code class="language-plaintext"># printf "%02X%02X%02X%02Xn" 192 168 1 187  
C0A801BB</code></pre></div>

<p>And create a symbolic link C0A801BB which points to inetboot</p>

<div class="highlight"><pre><code class="language-plaintext"># cd /tftpboot
# ln -s ./inetboot C0A801BB
# ls -la
total 320
drwxrwxrwx   2 root  wheel     512 Nov  2 19:26 .
drwxr-xr-x  24 root  wheel    1024 Nov  2 19:16 ..
lrwxr-xr-x   1 root  wheel      10 Nov  2 19:26 C0A801BB -&gt; ./inetboot
-rw-r--r--   1 root  wheel  158224 Nov  2 19:13 inetboot</code></pre></div>

<p><!-- raw HTML omitted -->6.<!-- raw HTML omitted --> The Solaris 9 ISO which was mounted previously, needs to be NFS exported from the jumpstart server. The jumpstart
installation process will NFS mount this as part of the installation process. The <em>/etc/exports</em> is configured as follows:</p>

<div class="highlight"><pre><code class="language-plaintext"># cat /etc/exports
/data/jumpstart/sol9u9 -maproot=0 -network 192.168.1.0 -mask 255.255.255.0 
/data/jumpstart/sol9u9/Solaris_9/Tools/Boot -maproot=0 -network 192.168.1.0 -mask 255.255.255.0 </code></pre></div>

<p><!-- raw HTML omitted -->7.<!-- raw HTML omitted --> Finally we move on to the configuration of the boot parameter server, rpc.bootparamd. This will essentially pass
information to the jumpstart client in order to boot. In this case, it will specify NFS exported paths from the
jumpstart server for the Solaris 9 media. For this we configure the following in <em>/etc/bootparams</em>.
Note that 192.168.1.154 is the IP address of the jumpstart server:</p>

<div class="highlight"><pre><code class="language-plaintext"># cat /etc/bootparams
sparc root=192.168.1.154:/data/jumpstart/sol9u9/Solaris_9/Tools/Boot install=192.168.1.154:/data/jumpstart/sol9u9 boottype=:in rootopts=:rsize=4096</code></pre></div>

<p>After configuring steps 1-7 above, I enabled <em>tftpd</em>, started up: <em>rarpd</em>, <em>rpc.bootparamd</em> and the NFS server.
In my case, I ran these in debug mode where possible in order to keep an eye on things during the jumpstart process.
On the jumpstart client, I ran the following command from the OpenBoot prompt:</p>

<div class="highlight"><pre><code class="language-plaintext">boot net -v - install</code></pre></div>

<p>And this kicked off the whole process. Note that I didn’t have the correct serial cable handy in order to be able to
capture the bootup of the jumpstart client. However a  short snippet of the initial messages is provided here:</p>

<div class="highlight"><pre><code class="language-plaintext">ok boot net -v-install
Boot device: /pcielf,0/pcie1, 1/network@c, 1 File and args: -v- install Using Onboard Transceiver - Link Up.
Timeout waiting for ARP/RARP packet
Timeout waiting for ARP/RARP packet
2aa00
Server IP address: 192.168.1.154
Client IP address: 192.168.1.187 Using Onboard Transceiver - Link Up.

Using RARP/BOOTPARAMS...

Requesting Internet address for 8:0:20:13:de:ad
Internet address is: 192.168.1.187
hostname: sparc
Found 192.168.1.154 @ 0:1e:37:82:a6:b6 root server: 192.168.1.154 (192.168.1.154)
root directory: /data/jumpstart/so19u9/Solaris_9/Tools/Boot Size: 0x5f163+0x14c3d+0x25307 Bytes
SunOS Release 5.9 Version Generic_118558-34 64-bit. Copyright 1983-2003 Sun Microsystems, Inc. All rights reserved.
Use is subject to license terms.
os-io Ethernet address = 8:0:20:13:de:ad Using default device instance data
mem= 262144K (0x10000000)
avail mem = 245637120
root nexus = = GENIAL station 777S (UltraSPARC-IIe 500MHz)
...</code></pre></div>

<p>I ran the services on the jumpstart server in verbose mode where possible in order to follow along.
I&rsquo;ve provided some excerpts of the output from the various services below for completeness.</p>

<p>An excerpt from the various services running on the jumpstart server is provided below for completeness.</p>

<!-- raw HTML omitted -->
<div class="highlight"><pre><code class="language-plaintext"># ./rarpd -a -d
rarpd: wm0: 0:1e:37:82:a6:b6
rarpd: received packet on wm0
rarpd: 08:00:20:13:de:ad asked; sparc replied
rarpd: received packet on wm0
rarpd: 08:00:20:13:de:ad asked; sparc replied
rarpd: received packet on wm0
rarpd: 08:00:20:13:de:ad asked; sparc replied
rarpd: received packet on wm0
rarpd: 08:00:20:13:de:ad asked; sparc replied
rarpd: received packet on wm0
rarpd: 08:00:20:13:de:ad asked; sparc replied
rarpd: received packet on wm0
rarpd: 08:00:20:13:de:ad asked; sparc replied</code></pre></div>

<!-- raw HTML omitted -->
<!-- raw HTML omitted -->
<div class="highlight"><pre><code class="language-plaintext">Nov  2 20:44:21 netbsd syslogd[340]: last message repeated 2 times
Nov  2 18:51:51 netbsd dhcpcd[219]: wm0: Router Advertisement from fe80::da58:d7ff:fe00:6d83
Nov  2 18:51:51 netbsd dhcpcd[219]: wm0: Router Advertisement from fe80::da58:d7ff:fe00:6d83
Nov  2 20:58:23 netbsd rarpd[10179]: wm0: 0:1e:37:82:a6:b6
Nov  2 21:00:53 netbsd rarpd[10179]: received packet on wm0
Nov  2 21:00:53 netbsd rarpd[10179]: 08:00:20:13:de:ad asked; sparc replied
Nov  2 21:00:53 netbsd tftpd[9562]: 192.168.1.187: read request for C0A801BB: success
Nov  2 21:01:03 netbsd rarpd[10179]: received packet on wm0
Nov  2 21:01:03 netbsd rarpd[10179]: 08:00:20:13:de:ad asked; sparc replied
Nov  2 21:02:32 netbsd rarpd[10179]: received packet on wm0
Nov  2 21:02:32 netbsd rarpd[10179]: 08:00:20:13:de:ad asked; sparc replied
Nov  2 21:02:32 netbsd tftpd[10876]: 192.168.1.187: read request for C0A801BB: success
Nov  2 21:02:41 netbsd rarpd[10179]: received packet on wm0
Nov  2 21:02:41 netbsd rarpd[10179]: 08:00:20:13:de:ad asked; sparc replied
Nov  2 21:03:04 netbsd rarpd[10179]: received packet on wm0
Nov  2 21:03:04 netbsd rarpd[10179]: 08:00:20:13:de:ad asked; sparc replied
Nov  2 21:03:37 netbsd rarpd[10179]: received packet on wm0
Nov  2 21:03:37 netbsd rarpd[10179]: 08:00:20:13:de:ad asked; sparc replied</code></pre></div>

<!-- raw HTML omitted -->
<!-- raw HTML omitted -->
<div class="highlight"><pre><code class="language-plaintext"># ./rpc.bootparamd -d -r 0.0.0.0
rpc.bootparamd: whoami got question for 192.168.1.187
rpc.bootparamd: This is host sparc
rpc.bootparamd: match sparc with sparc
rpc.bootparamd: Returning sparc       192.168.1.154
rpc.bootparamd: getfile got question for "sparc" and file "root"
rpc.bootparamd: match sparc with sparc
rpc.bootparamd: returning server:192.168.1.154 path:/data/jumpstart/sol9u9/Solaris_9/Tools/Boot address: 192.168.1.154
rpc.bootparamd: whoami got question for 192.168.1.187
rpc.bootparamd: This is host sparc
rpc.bootparamd: match sparc with sparc
rpc.bootparamd: Returning sparc       192.168.1.154
rpc.bootparamd: getfile got question for "sparc" and file "root"
rpc.bootparamd: match sparc with sparc
rpc.bootparamd: returning server:192.168.1.154 path:/data/jumpstart/sol9u9/Solaris_9/Tools/Boot address: 192.168.1.154
rpc.bootparamd: getfile got question for "sparc" and file "rootopts"
rpc.bootparamd: match sparc with sparc
rpc.bootparamd: getfile can't resolve server  for sparc
rpc.bootparamd: getfile got question for "sparc" and file "rootopts"
rpc.bootparamd: match sparc with sparc
rpc.bootparamd: getfile can't resolve server  for sparc
rpc.bootparamd: getfile got question for "sparc" and file "rootopts"
rpc.bootparamd: match sparc with sparc
rpc.bootparamd: getfile can't resolve server  for sparc
rpc.bootparamd: getfile got question for "sparc" and file "root"
rpc.bootparamd: match sparc with sparc
rpc.bootparamd: returning server:192.168.1.154 path:/data/jumpstart/sol9u9/Solaris_9/Tools/Boot address: 192.168.1.154
rpc.bootparamd: whoami got question for 192.168.1.187
rpc.bootparamd: This is host sparc
rpc.bootparamd: match sparc with sparc
rpc.bootparamd: Returning sparc       192.168.1.154
rpc.bootparamd: getfile got question for "sparc" and file "root"
rpc.bootparamd: match sparc with sparc
rpc.bootparamd: returning server:192.168.1.154 path:/data/jumpstart/sol9u9/Solaris_9/Tools/Boot address: 192.168.1.154
rpc.bootparamd: getfile got question for "sparc" and file "install"
rpc.bootparamd: match sparc with sparc
rpc.bootparamd: returning server:192.168.1.154 path:/data/jumpstart/sol9u9 address: 192.168.1.154
rpc.bootparamd: getfile got question for "sparc" and file "sysid_config"
rpc.bootparamd: match sparc with sparc
rpc.bootparamd: match  with sparc
rpc.bootparamd: Unknown bootparams host 
rpc.bootparamd: getfile lookup failed for sparc
rpc.bootparamd: getfile got question for "sparc" and file "ns"
rpc.bootparamd: match sparc with sparc
rpc.bootparamd: match  with sparc
rpc.bootparamd: Unknown bootparams host 
rpc.bootparamd: getfile lookup failed for sparc</code></pre></div>

<!-- raw HTML omitted -->
<!-- raw HTML omitted -->
<div class="highlight"><pre><code class="language-plaintext"># ./mountd -d
Getting export list.
Got line 
Got line /data/jumpstart/sol9u9 -maproot=0 -network 192.168.1.0 -mask 255.255.255.0 
Making new ep fs=0xe00,0x1e5c8
doing opt -maproot=0 -network 192.168.1.0 -mask 255.255.255.0 
doing opt -network 192.168.1.0 -mask 255.255.255.0 
get_net: '192.168.1.0' v4 addr 1a8c0
doing opt -mask 255.255.255.0 
get_net: '255.255.255.0' v4 addr ffffff
Got line /data/jumpstart/sol9u9/Solaris_9/Tools/Boot -maproot=0 -network 192.168.1.0 -mask 255.255.255.0 
Found ep fs=0xe00,0x1e5c8
doing opt -maproot=0 -network 192.168.1.0 -mask 255.255.255.0 
doing opt -network 192.168.1.0 -mask 255.255.255.0 
get_net: '192.168.1.0' v4 addr 1a8c0
doing opt -mask 255.255.255.0 
get_net: '255.255.255.0' v4 addr ffffff
Getting mount list.
Here we go.
got mount request from 192.168.1.187
-&gt; rpcpath: /data/jumpstart/sol9u9/Solaris_9/Tools/Boot
-&gt; dirpath: /data/jumpstart/sol9u9/Solaris_9/Tools/Boot
comparing:
c0a801
c0a801
Mount successful.
got mount request from 192.168.1.187
-&gt; rpcpath: /data/jumpstart/sol9u9/Solaris_9/Tools/Boot
-&gt; dirpath: /data/jumpstart/sol9u9/Solaris_9/Tools/Boot
comparing:
c0a801
c0a801
Mount successful.
got mount request from 192.168.1.187
-&gt; rpcpath: /data/jumpstart/sol9u9/Solaris_9/Tools/Boot
-&gt; dirpath: /data/jumpstart/sol9u9/Solaris_9/Tools/Boot
comparing:
c0a801
c0a801
Mount successful.
got mount request from 192.168.1.187
-&gt; rpcpath: /data/jumpstart/sol9u9
-&gt; dirpath: /data/jumpstart/sol9u9
comparing:
c0a801
c0a801
Mount successful.</code></pre></div>

<!-- raw HTML omitted -->
<p>After a few fits and starts, I managed to get it all working.  Note that the final missing piece was that I had to ping
the SPARC laptop during the bootstrap of the installation for it to continue on.  Thanks to <a href="https://twitter.com/CrivetiMihai">@CrivetiMihai</a> for this tip again.</p>

<p>With Solaris 9 U9 installed and working, I turned my attention to OpenCSW. I followed the procedure defined <a href="https://www.opencsw.org/manual/for-administrators/setup-old-versions.html#old-solaris">here</a> for older Solaris versions. The recommended
utilities gzip, coreutils, wget all seemed to install and function without issue. However ssh still appears to have an
issue. I continue to see the following error running ssh from OpenCSW (Solaris 9):</p>

<div class="highlight"><pre><code class="language-plaintext">libresolv.so.2: version `SUNW_2.2.1' not found</code></pre></div>

<p>The error is discussed in the following <a href="http://lists.opencsw.org/pipermail/bug-notifications/2013-April/011736.html">thread</a>. I’ll be looking at compiling both ssh and sshd using the gcc version from OpenCSW.</p>

<p>With that, I’ll also be looking at the jumpstart installation of Solaris 10 on the laptop.  Although I know that with
256MB RAM, it may not be the way to go. As an aside, my initial attempts at this so far have failed and that will
hopefully be the subject of another writeup in the future.</p>

<p>And as for that Ultra 5 that graced my desk between 2000-2005?  Well it’s sitting in my basement too waiting for another
rainy weekend day.</p>