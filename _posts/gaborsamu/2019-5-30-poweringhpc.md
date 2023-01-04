---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2019-05-30 03:21:37'
layout: post
original_url: https://www.gaborsamu.com/blog/poweringhpc/
slug: powering-the-future-of-hpc-ai-with-openpower
title: Powering the Future of HPC & AI with OpenPOWER
---

<p>It is coming up on one year that the Summit supercomputer based on IBM POWER9 at Oak Ridge National Lab claimed the number one spot on the Top500 ranking. This system represents the culmination of a significant collaboration between OpenPOWER foundation members IBM, Nvidia, Mellanox and Red Hat with the goal of producing well a balanced computing platform for not only traditional HPC workloads such as modelling and simulation, but also AI workloads. With this milestone approaching, we took the opportunity to catch-up with Hugh Blemings, Executive Director at the OpenPOWER Foundation to chat about the foundation, and what lies ahead.</p>

<p><strong>Q: Our readership may not have heard of the OpenPOWER Foundation, what’s your 30 second summary?</strong></p>

<p>A: About six years ago now, IBM realised that to widen the reach of their POWER technology it’d make sense to have other companies making use of it. Accordingly they worked with Google, Mellanox, Nvidia and Tyan as founding members to set up the OpenPOWER Foundation with that goal in mind.</p>

<p>The Foundation promotes OpenPOWER technologies, including software and hardware while facilitating an open ecosystem around them that provides industry specifications, helps members collaborate with each other on their various endeavours and provides training and promotion of a Power architecture and Member solutions</p>

<p>As it stands in mid 2019 we’ve got over a dozen companies making system hardware – servers, desktops, adapter cards etc based around POWER8 and POWER9, something north of 50 ISVs with OpenPOWER optimised software and well over 300 members overall – academic, individual and corporate.</p>

<p><strong>Q: You mentioned that you’ve been doing a bit of a tour of open source conferences and similar events exhibiting OpenPOWER these last 18 months, what have been some of the takeaways from those conversations?</strong></p>

<p>A: It’s been very interesting. We try and have a mix of member solutions at events – we’ve had everything from systems for the traditional data center, Open Compute kit, desktop solutions, as well as demos of world class software, accelerators and I/O devices on the booth to show the diversity of the OpenPOWER ecosystem as well as illustrate the common threads.</p>

<p>We almost always have a system or two from <a href="https://raptorcs.com/">Raptor Engineering</a> – they’re based in the US and specialise in very open and high performance systems. Both their Talos II and Blackbird systems are entirely open hardware designs – so you have information down to a schematic and PCB layout level – as well as an entirely open software stack – firmware, BMC, operating system and apps.</p>

<p>This openness is, I think, unique in the industry for a high performance system, and the Blackbird is at a very keen price point too. We put a sign on one the other week at the Red Hat Summit that said “POWER9 under two grand” that got people’s attention.</p>

<p>We usually have a Google Zaius or other hyperscale nodes in the booth. As Google announced at our 2018 Summit, the Zaius is “Google Strong” and has gone into production and is again an Open Hardware design – but very much the other end of the design point to the Raptor systems being intended for hyperscale.</p>

<p>So my first takeaway has been that performance is almost a given – these are the same chips and basic designs that power the #1 and #2 systems in the Top500, so of course they’re fast and truly built for HPC and AI workloads. But when you walk people through the openness of the systems and what that can mean for building trusted systems, that’s where people get excited.</p>

<p>It’s probably worth noting that POWER9 doesn’t have a management engine on chip like most x86 architecture parts, so this helps a lot with that trust factor. It does mean that if you want to run Minix on it you have to do it yourself though.</p>

<p>The second takeaway has been that there are significant numbers of customers buying OpenPOWER systems from companies other than IBM and using them to develop and fine tune their HPC codes before deploying them up to (say) Sierra or Summit. Sometime this is a rack or two, other times a couple of boxes to go under a (lucky) developers desk.</p>

<p>In fact I think you can now get a system from Raptor that has a couple of POWER9s, an NVIDIA GPU and the IBM PowerVision software all ready to go – so you can put one under your desk for development – the ultimate developer workstation there!</p>

<p>The last takeaway has been simply that folks are pleasantly surprised to hear they can get OpenPOWER hardware from companies in addition to IBM which shows the diversity of the ecosystem and solutions available.</p>

<p><strong>Q: So what are the other hardware offerings that come to your mind?</strong></p>

<p>A: The gear from Yadro is interesting – they have a POWER8 design that focuses on huge memory and storage capacity in a single box. Something like 384 threads with 8TB of RAM and 24 NVMe Drives, all in a 2U 19” rack. Great for in memory databases and very memory intensive workloads – AI, ML that sort of thing.</p>

<p>Wistron have their Mihawk, a couple of POWER9s, PCIe Gen3 and Gen4 slots a plenty, OpenCAPI for GPU/FPGA accelerators and a ton of solid state storage in the front of the rack which makes for a powerful inferencing system. Inspur Power Systems have five new OpenPOWER models about to drop too which they showed off at the NVIDIA GTC Conference and our 1st OpenPOWER Summit in Tokyo last week.</p>

<p><strong>Q: What do you think attracts HPC and AI users to OpenPOWER?</strong></p>

<p>A: One of the senior technical folks from a US national lab once explained to me that it’s been about feeds for a while now – memory and I/O bandwidth. If you look at POWER9 you’ve a processor design that is architected with this in mind internally, then to get off chip you’ve got lots of NVLink for GPUs, CAPI and OpenCAPI connections, DDR4 channels plus PCIe Gen4 – so you’ve doubled the bandwidth down at the physical layer.</p>

<p>NVLink and CAPI/OpenCAPI are, unsurprisingly, a big draw too. Having a cache coherent interconnect between your CPUs and off chip accelerator simplifies programming models immensely and there is a significant performance improvement too just from reducing if not eliminating copying data about, for example from memory to the CPU, on to the GPU and back.</p>

<p><strong>Q: What’s the state of the Open Source software ecosystem around OpenPOWER nowadays?</strong></p>

<p>A: As you might recall, OpenPOWER, or PowerPC as it was back then was one of the earliest non-x86 ports of things like the Linux Kernel, the GNU Compiler Collection etc. is very mature. All the Linux distros just work – Red Hat, SuSE, Ubuntu, Gentoo, Red Flag, Debian etc. Given that the POWER ISA has evolved very cleanly over the years it’s not unknown to see people running old 32 bit PowerPC code with minimal if any modification.</p>

<p><strong>Q: ISC19 is coming up in Frankfurt next month, will OpenPOWER have a presence?</strong></p>

<p>A: Yes, we’ll have a booth there with some hardware for folks to look at. We’ll have at least Blackbird and Zaius planars to check out. We’re waiting for confirmation on some other machines, too. Oh, and free OpenPOWER stickers of course!</p>

<p>In addition to our booth, over 20 OpenPOWER members will be at ISC this year with their offerings. IBM will have a node from the world’s fastest super computer on display, Inspur and Inspur Power Systems, E4, Mellanox, Red Hat, Xilinx, NEC, numerous universities and software houses will have their OpenPOWER solutions on show. And don’t forget OpenCAPI will also be there. They are, I believe, the only open high speed cache coherent interconnect available across multiple architectures that is in production use.</p>

<p>For those interested in learning from subject matter experts on how OpenPOWER systems are used in HPC, the International Workshop on OpenPOWER for HPC (IWOPH’19) is featured as part of the ISC workshops on the Thursday of the event.</p>

<p><strong>Q: Can you share any insight about what the future holds for OpenPOWER?</strong></p>

<p>A: I’m a technical person at heart so I hate saying things that sound perilously close to marketing speak, however a couple of things come to mind.</p>

<p>The roadmap for OpenPOWER CPU-wise is looking really strong with details of both an updated POWER9 and POWER10 offering being discussed.</p>

<p>We’re seeing more adoption of low cost OpenPOWER hardware by individual developers, researchers and security conscious end users – broadens the ecosystem and overall user base.</p>

<p>There is at least one other announcement in the works that I think will truly be industry changing, but we’re a couple of months out from being able to discuss more widely. I’d perhaps simply recommend our OpenPOWER North American Summit in San Diego to your readership – it’s connected with the Linux Foundation Open Source Summit and will be the place to be in August.</p>