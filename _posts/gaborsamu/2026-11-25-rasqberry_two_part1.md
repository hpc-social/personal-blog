---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2026-11-25 14:20:58'
layout: post
original_url: https://www.gaborsamu.com/blog/rasqberry_two_part1/
slug: building-my-own-rasqberry-two
title: Building my own RasQberry Two
---

<p>There&rsquo;s a new resident on my desk, and it has been quietly blinking away for a few months now. It&rsquo;s called <a href="https://rasqberry.org/">RasQberry Two</a>, and with <a href="https://sc26.supercomputing.org/">SC26</a> wrapped up, it&rsquo;s high time I wrote about it. What on earth is a RasQberry Two, you might ask? Those who I mentioned it to assumed I mispronounced Raspberry Pi. And although Raspberry Pi is an integral piece of the project, it goes much beyond that.</p>

<p>In simple terms, the RasQberry Two is a functional model of an <a href="https://www.ibm.com/quantum">IBM Quantum System Two</a>. It brings together a very cool 3D printed model with a Raspberry Pi and the <a href="https://www.ibm.com/quantum/qiskit">Qiskit SDK</a> for, in the words of the site, &ldquo;teaching, meetups and demo booths.&rdquo; What they neglect to mention is that it also happens to make an exceedingly cool item for your desk!</p>

<p>And my interest in quantum computing isn&rsquo;t new. Those who have been following me closely will note that a year ago at #SC25 our team showcased #IBM LSF managing a hybrid quantum-classical workflow, which was the result of a collaboration between IBM Research and the LSF team. Since #SC25 that work has evolved. Read the latest on the LSF integration with the Quantum Resource Management Interface (QRMI) <a href="https://community.ibm.com/community/user/blogs/gbor-samu/2026/11/19/how-ibm-lsf-brings-qpus-into-enterprise-hpc">here</a>.</p>

<p>This writeup is not intended to provide detailed information on the bill of materials nor exhaustive steps on assembly, as that is very well covered on the <a href="https://rasqberry.org/">RasQberry Two site</a>, along with photos. Rather, it&rsquo;s meant to highlight my experience with the build.</p>

<p>Building the RasQberry Two required sourcing parts from several companies, including Amazon for the magnets and 90 degree USB-C elbow for power, AliExpress for the LED panels, the local country RPi shop for the Raspberry Pi 5, power supply and cooling fan, and finally there was the model itself, which needed to be 3D printed.</p>

<ul>
<li>Raspberry Pi 5 16GB</li>
<li>Raspberry Pi 27W USB-C Power Supply (White US)</li>
<li>Raspberry Pi Active Cooler</li>
<li>Duttek Right Angle USB-C Male to USB-C Female</li>
<li>4 x WS2812IC 4x12 LED matrix 5V</li>
<li>Requisite magnets</li>
<li>Nylon nuts and bolts to fasten the floor pieces</li>
</ul>
<p>As I don&rsquo;t own a 3D printer at home, I turned to a local maker space called <a href="https://www.newmakeit.com/">NewMakeIt</a>, based in Newmarket, Ontario. A big shout-out to Ryan Dibisch at NewMakeIt for helping to make the process as smooth as possible. Ryan was even able to source the Prusa PLA Galaxy Black translucent filament that was needed for part of the model, and commented that the 3D models themselves were of exceptional quality.</p>

<p>Fresh from becoming an <a href="https://www.credly.com/badges/61bacbe0-2014-42e3-b1ef-a4b44fff1b50">IBM Associate Quantum Ambassador</a> in June 2026, I embarked on assembling all the components to build my very own RasQberry Two. I started by writing the RasQberry Two software image to an SD card and tested the operation of the Raspberry Pi 5. Next, I began the assembly of the model from the 3D printed parts. Overall, I found the instructions to be very straightforward, from building the base and Quantum System Two model to wiring up the LED panels and installing the RPi 5.</p>

<p>After starting with the collection of parts shown in the following X post…</p>

<blockquote class="twitter-tweet"><p dir="ltr" lang="en">This collection of components is the starting point for a RasQberry Two — for exploring <a href="https://x.com/hashtag/quantumcomputing?src=hash&amp;ref_src=twsrc%5Etfw">#quantumcomputing</a> and <a href="https://x.com/hashtag/Qiskit?src=hash&amp;ref_src=twsrc%5Etfw">#Qiskit</a> with a <a href="https://x.com/hashtag/RaspberryPi?src=hash&amp;ref_src=twsrc%5Etfw">#RaspberryPi</a> and 3D printed parts. Looking forward to assembling it over the next week during the evenings. Learn more here <a href="https://t.co/aHbo1x0HKy">https://t.co/aHbo1x0HKy</a> <a href="https://x.com/hashtag/IBM?src=hash&amp;ref_src=twsrc%5Etfw">#IBM</a> <a href="https://t.co/Fk6dDW3kiF">pic.twitter.com/Fk6dDW3kiF</a></p>
&mdash; Gábor SAMU (@gabor_samu) <a href="https://x.com/gabor_samu/status/2065067567497007113?ref_src=twsrc%5Etfw">June 11, 2026</a></blockquote>



<p>…I ended up with the following deskside companion!</p>

<figure><img alt="My completed RasQberry Two model" src="https://www.gaborsamu.com/images/rasqberry-two.jpg" /><figcaption>
      <p>My completed RasQberry Two</p>

    </figcaption>
</figure>

<p>Late in August 2026, I had the opportunity to see the Quantum System Two in person at IBM TJ Watson Research Center in Yorktown Heights, NY.</p>

<blockquote class="twitter-tweet"><p dir="ltr" lang="en">Had the privilege to visit the <a href="https://x.com/hashtag/IBM?src=hash&amp;ref_src=twsrc%5Etfw">#IBM</a> TJ Watson Research Center last week. Seeing IBM Quantum System Two in person was a highlight. Amazing technology and an exciting glimpse into the future of computing ⚛️🚀 <a href="https://x.com/hashtag/QuantumComputing?src=hash&amp;ref_src=twsrc%5Etfw">#QuantumComputing</a> <a href="https://x.com/hashtag/IBMQuantum?src=hash&amp;ref_src=twsrc%5Etfw">#IBMQuantum</a> <a href="https://x.com/hashtag/IBMResearch?src=hash&amp;ref_src=twsrc%5Etfw">#IBMResearch</a> <a href="https://t.co/TwTr8sIoZP">pic.twitter.com/TwTr8sIoZP</a></p>
&mdash; Gábor SAMU (@gabor_samu) <a href="https://x.com/gabor_samu/status/2092236188081873348?ref_src=twsrc%5Etfw">August 25, 2026</a></blockquote>



<p>So, you may be wondering what I&rsquo;ve been up to on this RPi 5 powered IBM Quantum System Two model? Well, a hint comes in the following blog from the recent SC26 event regarding the integration of QPUs into enterprise HPC using #IBM LSF and QRMI: <a href="https://community.ibm.com/community/user/blogs/gbor-samu/2026/11/19/how-ibm-lsf-brings-qpus-into-enterprise-hpc">How IBM LSF brings QPUs into enterprise HPC</a>.</p>

<p>And with that, stay tuned for another chapter in the story of my own RasQberry Two!</p>