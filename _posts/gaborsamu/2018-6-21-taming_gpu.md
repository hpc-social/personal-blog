---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2018-06-21 03:21:37'
layout: post
original_url: https://www.gaborsamu.com/blog/taming_gpu/
slug: the-taming-of-the-gpu
title: The Taming of the GPU
---

<p>The media has been alight with articles regarding the groundbreaking Summit supercomputer recently unveiled at Oak Ridge National Laboratory. It sports a mind boggling 9,216 IBM POWER9 CPUs, 27,648 NVIDIA Tesla GPUs, underpinned with 250 petabytes of storage. This muscle will be put to good use running traditional HPC as well as AI workloads across a broad range of sciences.</p>

<p>Looking at the landscape of systems being built for HPC and now AI, there is one commonality – many are hybrid CPU-GPU systems. Whether we’re considering systems at the pinnacle of computing such as Summit, or commercial HPC and AI systems, GPUs have become a defacto method for accelerating code and providing copious amounts of floating point performance.</p>

<p>The early days of clustered computing saw the advent of workload and resource managers which were a means of taming environments by orchestrating access to, and bringing computing resources to bear, in a predictable manner – aligned with the needs of scientists and businesses alike. As environments have grown in scale to meet the growing thirst for HPC, GPUs and accelerated computing have stepped out on stage to take a bow.</p>

<p>Software developers have and continue to port and optimize applications to benefit from the capabilities provided by GPUs. According to a recent report from November 2017, a high percentage of HPC applications now offer GPU support.</p>

<blockquote>
<p><strong>“According to the latest HPC User Site Census data and additional research, of the 50 most popular application packages mentioned by HPC users, 34 offer GPU support (including two under current development), including 9 of the top 10.”</strong></p>

</blockquote>
<p>Indeed, the recent Top 500 list (November 2017) includes no less than 87 hybrid CPU-GPU systems (and more counting other types of accelerators).</p>

<p>So how do GPU-heavy systems impact the task of the workload and resource managers? Fundamentally, as GPUs are resources, workload schedulers have had to adapt too.</p>

<p><strong>A wild west land grab</strong></p>

<p>It’s not just large-scale supercomputers that face the challenges of compute supply versus user demands.  Commercial HPC environments are also now increasingly hybrid CPU-GPU based with potentially hundreds of users and millions of jobs per day in high-throughput computing use cases. These are complex environments and large investments requiring workload management software with sophisticated capabilities to reign in all the resources – so that users end up with GPU workloads running on the right servers.</p>

<p>Computing environments today can have some servers with GPUs, some without, varied GPU configurations including models and memory, and a different number of GPUs per node. Adding to this complexity, in a typical data center, servers can come and go so the configuration is not always static.</p>

<p>In general, workload schedulers require the administrator to specify in the configuration whether a given server is equipped with GPUs, often requiring additional information such as the GPU model, etc. Without this crucial information, the workload scheduler cannot effectively route jobs to nodes – potentially leading to a Wild West grab for resources.</p>

<p><strong>Call in the Cavalry</strong></p>

<p>IBM Spectrum LSF has been continuously innovating to address the needs of increasingly complex HPC environments of scale since 1992. Support for NVIDIA GPUs was first introduced in IBM Spectrum LSF in 2007. Continuing this long tradition of enhancements to NVIDIA GPU support, IBM Spectrum LSF now includes a new capability designed to dramatically simplify the administration of GPU servers and enables users to be more productive faster. With “zero config” for NVIDIA GPUs, IBM Spectrum LSF detects the presence of GPUs and automatically performs the necessary scheduler configuration – without any interaction from the administrator. IBM Spectrum LSF will help tame the GPU environment for you, allowing users with GPU ready codes to be productive from the moment the environment is setup.</p>