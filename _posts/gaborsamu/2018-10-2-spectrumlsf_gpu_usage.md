---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2018-10-02 03:21:37'
layout: post
original_url: https://www.gaborsamu.com/blog/spectrumlsf_gpu_usage/
slug: gpu-usage-information-for-jobs-in-ibm-spectrum-lsf
title: GPU usage information for jobs in IBM Spectrum LSF
---

<p>In my last blog, we ran through an example showing how IBM Spectrum LSF now automatically detects the presence of NVIDIA GPUs on hosts in the cluster and performs the necessary configuration of the scheduler automatically.</p>

<p>In this blog, we take a closer look at the integration between Spectrum LSF and
NVIDIA DCGM which provides GPU usage information for jobs submitted to the
system.</p>

<blockquote class="twitter-tweet"><p dir="ltr" lang="en"><a href="https://twitter.com/hashtag/IBMSpectrum?src=hash&amp;ref_src=twsrc%5Etfw">#IBMSpectrum</a> <a href="https://twitter.com/hashtag/LSF?src=hash&amp;ref_src=twsrc%5Etfw">#LSF</a> supports <a href="https://twitter.com/hashtag/NVIDIA?src=hash&amp;ref_src=twsrc%5Etfw">#NVIDIA</a> DCGM, allowing you to get the most our of your <a href="https://twitter.com/hashtag/GPUs?src=hash&amp;ref_src=twsrc%5Etfw">#GPUs</a> <a href="https://t.co/aCo9cFHNkq">https://t.co/aCo9cFHNkq</a> <a href="https://twitter.com/hashtag/HPCmatters?src=hash&amp;ref_src=twsrc%5Etfw">#HPCmatters</a></p>
&mdash; Gábor SAMU (@gabor_samu) <a href="https://twitter.com/gabor_samu/status/806540074888396800?ref_src=twsrc%5Etfw">December 7, 2016</a></blockquote>


<p>To enable the integration between Spectrum LSF and NVIDIA DCGM, we
need to specify the <em>LSF_DCGM_PORT=&lt;port number&gt;</em> parameter in
<em>LSF_ENVDIR/lsf.conf</em></p>

<div class="highlight"><pre><code class="language-plaintext">root@kilenc:/etc/profile.d# cd $LSF_ENVDIR
root@kilenc:/opt/ibm/lsfsuite/lsf/conf# cat lsf.conf |grep -i DCGM
LSF_DCGM_PORT=5555</code></pre></div>

<p>You can find more details about the variable <em>LSF_DCGM_PORT</em> and what it
enables <a href="https://www.ibm.com/support/knowledgecenter/en/SSWRJV_10.1.0/lsf_config_ref/lsf.conf.lsf_dcgm_port.5.html">here</a>.</p>

<p>Before continuing, please ensure that the DCGM daemon is up and running.  Below
we start DCGM on the default port and run a query command to confirm that it&rsquo;s
up and running.</p>

<div class="highlight"><pre><code class="language-plaintext">root@kilenc:/opt/ibm/lsfsuite/lsf/conf# nv-hostengine
Started host engine version 1.4.6 using port number: 5555

root@kilenc:/opt/ibm/lsfsuite/lsf/conf# dcgmi discovery -l
1 GPU found.
+--------+-------------------------------------------------------------------+
| GPU ID | Device Information                                                |
+========+===================================================================+
| 0      |  Name: Tesla V100-PCIE-32GB                                       |
|        |  PCI Bus ID: 00000033:01:00.0                                     |
|        |  Device UUID: GPU-3622f703-248a-df97-297e-df1f4bcd325c            |
+--------+-------------------------------------------------------------------+ </code></pre></div>

<p>Next, let&rsquo;s submit a GPU job to IBM Spectrum LSF to demonstrate the collection
of GPU accounting.  Note that the GPU job must be submitted to Spectrum LSF
with the exclusive mode specified in order for the resource usage to be
collected. As was the case in my previous blog, we submit the <em>gpu-burn</em> test
job (formally known as Multi-GPU CUDA stress test).</p>

<div class="highlight"><pre><code class="language-plaintext">test@kilenc:~/gpu-burn$ bsub -gpu "num=1:mode=exclusive_process" ./gpu_burn 120
Job &lt;54086&gt; is submitted to default queue &lt;normal&gt;</code></pre></div>

<p>Job <em>54086</em> runs to successful completion and we use the Spectrum LSF <em>bjobs</em> command with the <em>-gpu</em> option to display the GPU usage
information in the output below.</p>

<div class="highlight"><pre><code class="language-plaintext">test@kilenc:~/gpu-burn$ bjobs -l -gpu 54086

Job &lt;54086&gt;, User &lt;test&gt;, Project &lt;default&gt;, Status &lt;DONE&gt;, Queue &lt;normal&gt;, Com
                     mand &lt;./gpu_burn 120&gt;, Share group charged &lt;/test&gt;
Mon Oct  1 11:14:04: Submitted from host &lt;kilenc&gt;, CWD &lt;$HOME/gpu-burn&gt;, Reques
                     ted GPU &lt;num=1:mode=exclusive_process&gt;;
Mon Oct  1 11:14:05: Started 1 Task(s) on Host(s) &lt;kilenc&gt;, Allocated 1 Slot(s)
                      on Host(s) &lt;kilenc&gt;, Execution Home &lt;/home/test&gt;, Executi
                     on CWD &lt;/home/test/gpu-burn&gt;;
Mon Oct  1 11:16:08: Done successfully. The CPU time used is 153.0 seconds.
                     HOST: kilenc; CPU_TIME: 153 seconds
                        GPU ID: 0
                            Total Execution Time: 122 seconds
                            Energy Consumed: 25733 Joules
                            SM Utilization (%): Avg 99, Max 100, Min 64
                            Memory Utilization (%): Avg 28, Max 39, Min 9
                            Max GPU Memory Used: 30714888192 bytes


GPU Energy Consumed: 25733.000000 Joules


 MEMORY USAGE:
 MAX MEM: 219 Mbytes;  AVG MEM: 208 Mbytes

 SCHEDULING PARAMETERS:
           r15s   r1m  r15m   ut      pg    io   ls    it    tmp    swp    mem
 loadSched   -     -     -     -       -     -    -     -     -      -      -  
 loadStop    -     -     -     -       -     -    -     -     -      -      -  

 EXTERNAL MESSAGES:
 MSG_ID FROM       POST_TIME      MESSAGE                             ATTACHMENT
 0      test       Oct  1 11:14   kilenc:gpus=0;                          N     

 RESOURCE REQUIREMENT DETAILS:
 Combined: select[(ngpus&gt;0) &amp;&amp; (type == local)] order[gpu_maxfactor] rusage[ngp
                     us_physical=1.00]
 Effective: select[((ngpus&gt;0)) &amp;&amp; (type == local)] order[gpu_maxfactor] rusage[
                     ngpus_physical=1.00]

 GPU REQUIREMENT DETAILS:
 Combined: num=1:mode=exclusive_process:mps=no:j_exclusive=yes
 Effective: num=1:mode=exclusive_process:mps=no:j_exclusive=yes

 GPU_ALLOCATION:
 HOST             TASK ID  MODEL        MTOTAL  FACTOR MRSV    SOCKET NVLINK                           
 kilenc           0    0   TeslaV100_PC 31.7G   7.0    0M      8      -               </code></pre></div>

<p>And to close, yours truly spoke at the HPC User Forum in April 2018 (Tucson, AZ) giving a
short update in the vendor panel about Spectrum LSF, focusing on GPU support.</p>


<div style="padding-bottom: 56.25%; height: 0; overflow: hidden;">
  
</div>