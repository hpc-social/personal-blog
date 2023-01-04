---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2022-03-29 13:53:09'
layout: post
original_url: https://www.gaborsamu.com/blog/reliving_amiga/
slug: relivin-the-90-s-amiga-style
title: Relivin' the 90's - Amiga style
---

<p>Although I very much started my experience with home computers with IBM
compatibles running MSDOS in the late 1980&rsquo;s, I&rsquo;m a lifelong, self-professed
Commodore-Amiga addict. I distinctly recall the launch of the Amiga A1000 and
being dazzled by it&rsquo;s multimedia capabilities around the same time that
I had a PC XT with CGA graphics. I was instantly hooked. Having great video
games for the time was just icing on the cake.</p>

<p>I started my Amiga experience with an A500, which I quickly traded in for an
A2000 model, which I still have today. I came across an A3000 in the late 1990&rsquo;s
for a small sum which I added to my collection. The A3000 is my favourite
Amiga system with onboard SCSI and it&rsquo;s design is reminiscent of pizza-box
UNIX servers which were common back in the day.</p>

<p>The majority of my friends at the time were all-in on PCs. But for me there
was just something a bit clinical and boring about them. The Amiga filled
this gap for me and continues to do so. It&rsquo;s probably one of big
reasons why I still to this day tinker so much with non-X86 systems.</p>

<p>Retro computing is a hobby that requires much time. So it&rsquo;s sometimes
challenging to juggle this hobby with other things, especially as the weather
turns warmer here in Southern Ontario. My A3000 system was one that I was
looking to prioritize for resurrection this spring. This is in particular
because the last thing I tinkered with on the A3000 roughly 20 years back was Amiga UNIX. Yes, my A3000 sat in storage for around 20 years!  In the mid to late 90&rsquo;s, I ventured out to an Amiga speciality shop in London, Ontario (Canada) for a clearance they were having.  It&rsquo;s there that I happened across Amiga UNIX software (tape and manuals), as well the <a href="http://amiga.resource.cx/exp/a2410">Commodore A2410 High Resolution Graphics board</a>, <a href="http://amiga.resource.cx/exp/a2065">Commodore A2065 Ethernet board</a> and a Wangtek 5150ES tape drive
(which is mounted in a SUN Microsystems external case). Here is a view of the
Amiga <em>ShowConfig</em> output.</p>

<figure><img src="https://www.gaborsamu.com/images/system_config.png" />
</figure>

<p>I had the foresight to remove the motherboard RTC batteries before
storing the systems. But my A3000 refused to boot when I took it out of storage
late in 2021. After much fiddling, I decided to reach out to a local Amiga
repair specialist. The gentleman worked at Comspec(?), which did repair work
for Commodore back in the day.</p>

<p>I recently got my A3000 back after the fault was corrected, and a new
replaceable coin battery for the RTC was installed. The fault turned
out to be an issue with some of the ZIP memory sockets. Because of the
difficulty and cost in purchasing ZIP memory back in the day, I purchased
a <a href="http://amiga.resource.cx/exp/amifast">ProvTech AmiFast 3000</a> ZIP to SIMM converter which allows me to use 72-pin SIMM memory.</p>

<p>With a working A3000 system, it was time to look at software once again.
I found my old dusty Amiga OS 3.5 and OS 3.9 original media sets. With
some effort I was able to get Amiga OS 3.9 installed on the system. It&rsquo;s
not that the installation is difficult, it was more a matter of getting my
CDROM working and clearing out some of the cobwebs on my Amiga knowledge.</p>

<p>Additionally, I was able to successfully boot Amiga UNIX off an external
SCSI disk which I installed back in roughly &lsquo;98 or &lsquo;99. I plan to write
more about Amiga UNIX in a subsequent post. For those who are curious about
Amiga UNIX there is a fantastic wiki <a href="https://www.amigaunix.com/doku.php">here</a>.</p>

<p>Back to Amiga OS 3.9. After getting it installed successfully I had a few goals:</p>

<ul>
<li>Get my Amiga online via the A2065 Ethernet board</li>
<li>Get a high resolution Workbench (desktop) via the A2410</li>
<li>Relive the memories!</li>
</ul>
<p><strong>Amiga on the &lsquo;Net</strong></p>

<p>I recall back in the day various Amiga TCP/IP implementations such as
AS225 and AmiTCP. Consulting with the gentleman who repaired my Amiga,
he suggested <a href="http://roadshow.apc-tcp.de/index-en.php">Roadshow</a>. I&rsquo;d never
heard of Roadshow before, but downloaded and got the trial version working
easily. I required to copy the a2065.device driver for the
A2065 board to the system and created the necessary configuration file
in <em>SYS:Devs/NetInterfaces</em>. The configuration file A2065 is shown in the
image below.</p>

<figure><img src="https://www.gaborsamu.com/images/roadshow1.png" />
</figure>

<p>A quick aside here. I had to create a CD with a bunch of software
including Roadshow and a number of utilities from <a href="http://aminet.net/">Aminet</a> such as the A2065 device driver. Aminet is one of the goto places for Amiga
software on the net.</p>

<p>I found Roadshow so easy to get setup and working that I purchased a license
for it. I also purchased licenses at the same time for <a href="http://www.bitplan.pl/goadf/">GoADF</a>, which is a great utility for managing ADF (Amiga Disk Format) files.</p>

<p>With a working TCP/IP stack, I installed the trial version of <a href="https://www.ibrowse-dev.net/">iBrowse</a>, in addition to the FTP utility RNOXfer (from Aminet).  With a working FTP client, I could now more easily move files to and from the A3000.  This definitely helped for the next stage.</p>

<figure><img src="https://www.gaborsamu.com/images/rnoxfer.png" />
</figure>

<p>Just a note that browsing on the Amiga is definitely a retro experience.
This is in no way a slight at the fine folks who develop and maintain
browsers such as iBrowse.  I&rsquo;m considering updating my iBrowse demo
license to a full license in the future as well.</p>

<p>I also took the opportunity at this time to install an NTP client. Even though
my Amiga now has a working RTC, I still like to use NTP to keep the clock
accurately set.  For this I used the AmiTimeKeeper utility from Aminet.
I pointed it as I do normally to the NTP servers at the National Research
Council (NRC) of Canada.  TimeKeeper has a CLI interface as well as a UI
status window to provide information on the synchronization status.</p>

<figure><img src="https://www.gaborsamu.com/images/timekeeper.png" />
</figure>

<p><strong>Workbench à la A2410</strong></p>

<p>It was time to move on to having a high resolution Workbench (desktop) experience. I also own a Picasso II video card which is presently in my Amiga 2000 system. Using P96 or CyberGraphX on the Picasso II was quite straightforward in
the past.  My goal this time was to use the A2410, which from what I could
read, was supported in CyberGraphX V4.</p>

<p>Thing is, when I went to install CyberGraphX V4 from my original media,
I did not see the A2410 listed. It was only when I applied the update(s) that I could see the A2410 listed as a supported video card. Note the final patch version of CyberGraphX I&rsquo;m using is from <em>cgxv42_rc6.lha</em> which I downloaded
from Aminet <a href="https://aminet.net/package/driver/video/CyberGraphX_4.3rc6">here</a>.</p>

<p>The A2410 CyberGraphX (CGX) driver installed without a hitch, getting
it to work was a challenge. Although I could get a Workbench to appear in the
desired resolution and colours, when I double clicked on any icon on the desktop, the system would hang. It was only through trial and error that I discovered
that some specific CyberGraphX variables had to be set.  The screenshot below of the CyberGraphX settings tool shows the current, woring settings. Ultimately,
the hang seemed to be addressed by enabling the CGX <em>SUPERGELS</em> variable.</p>

<figure><img src="https://www.gaborsamu.com/images/cyber_prefs.png" />
</figure>

<p>Here is a look at the CGX <em>showcgxconfig</em> tool output.</p>

<figure><img src="https://www.gaborsamu.com/images/cyber_config.png" />
</figure>

<p>A screenshot of the Workbench driven by the A2410 is shown below. The
performance is not great, but it does work, and I&rsquo;m super pleased about
that. On the subject of graphics cards, I&rsquo;ve had my eye on the MNT <a href="https://shop.mntmn.com/products/zz9000-for-amiga-preorder">ZZ9000</a> which I&rsquo;m considering purchasing to breathe more life into my A3000.</p>

<figure><img src="https://www.gaborsamu.com/images/desktop_clean.png" />
</figure>

<p>The next stage in this journey is to get the same configuration working
with Amiga OS 3.2, which I purchased from the folks at <a href="https://retrorewind.ca/">Retro Rewind</a> in Toronto. According to what I&rsquo;ve read, I need to downgrade the
intuition.library version to get CyberGraphX working with OS 3.2. I&rsquo;ll
write more about this when I have the opportunity.</p>

<p>And now, I&rsquo;m ready to begin to relive those memories!</p>

<p>Update!  Here are some photos of the A2065, A2410 and A3000 daughterboard
from my system.</p>

<figure><img src="https://www.gaborsamu.com/images/a2065.jpg" />
</figure>

<figure><img src="https://www.gaborsamu.com/images/a2410.jpg" />
</figure>

<figure><img src="https://www.gaborsamu.com/images/daughterboard.jpg" />
</figure>

<figure><img src="https://www.gaborsamu.com/images/1992.jpg" />
</figure>