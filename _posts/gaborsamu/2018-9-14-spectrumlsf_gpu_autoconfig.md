---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2018-09-14 16:38:29'
layout: post
original_url: https://www.gaborsamu.com/blog/spectrumlsf_gpu_autoconfig/
slug: a-hands-on-look-at-gpu-autoconfig-in-ibm-spectrum-lsf
title: A hands-on look at GPU "autoconfig" in IBM Spectrum LSF
---

<p>It&rsquo;s been a long time since I&rsquo;ve posted to my goulash blog.  I&rsquo;ve not disappeared, rather I&rsquo;ve been writing articles for the
IBM Accelerated Insights solution channel on <a href="https://www.hpcwire.com/solution_channel/ibm/">HPCWire</a>.  Since then, I&rsquo;ve been
fortunate enough to have access to a POWER9 based developer system equipped with a NVIDIA Tesla V100 PCIe card to put through it&rsquo;s
paces.  This is very timely for me as there is some exciting new functionality in IBM Spectrum LSF known as GPU auto detect, which
I recently wrote about in the article <a href="https://www.hpcwire.com/solution_content/ibm/cross-industry/the-taming-of-the-gpu/">The Taming of the GPU</a> that I&rsquo;ve been meaning to try out hands on.</p>

<blockquote class="twitter-tweet"><p dir="ltr" lang="en">Much like the number of CPUs and cores, <a href="https://twitter.com/hashtag/IBM?src=hash&amp;ref_src=twsrc%5Etfw">#IBM</a> Spectrum LSF automatically detects the presence of <a href="https://twitter.com/hashtag/NVIDIA?src=hash&amp;ref_src=twsrc%5Etfw">#NVIDIA</a> GPUs on each node in the cluster - so LSF can immediately schedule GPU workloads, correctly. Read more <a href="https://t.co/dGVbXBo1Ly">https://t.co/dGVbXBo1Ly</a>  <a href="https://twitter.com/hashtag/HPC?src=hash&amp;ref_src=twsrc%5Etfw">#HPC</a></p>
&mdash; Gábor SAMU (@gabor_samu) <a href="https://twitter.com/gabor_samu/status/1121569605707862016?ref_src=twsrc%5Etfw">April 26, 2019</a></blockquote>


<p>Back in Dark Ages (no not literally), administrators of HPC clusters had to specify in the configuration of the workload scheduler
which nodes were equipped with GPUs, the model of the GPUs and so on.  This was relatively straightforward when nodes were equipped
with single GPUs and clusters were smaller.  With the proliferation of GPUs, nodes are frequently equipped with multiple GPUs and
often times we can end up with a mix of GPU models in a single cluster where rolling upgrades of hardware has occurred.  Factor in
hybrid cloud environments where nodes can come and go as needed, and what is seemingly an easy update to configuration files of a
workload scheduler can become complex, quickly. Take into account that if a user is requesting a GPU for a job they&rsquo;ve  submitted
and the scheduler is not fully aware of which nodes are equipped with GPUs, you can end up with under-utilization of these assets.</p>

<p>Enter Spectrum LSF with a new capability known as GPU auto detect, which helps simplify the administration of heterogeneous computing
environments by detecting the presence of NVIDIA GPUs in nodes and automatically performing the necessary scheduler configuration.<br />
For a detailed list of GPU support enhancements in the latest update to IBM Spectrum LSF please refer to the following <a href="https://www.ibm.com/support/knowledgecenter/en/SSWRJV_10.1.0/lsf_release_notes/lsf_relnotes_gpu10.1.0.6.html">page</a>.</p>

<p>My testing environment is configured as follows:</p>

<ul>
<li>dual-socket POWER9 development system</li>
<li>1 x NVIDIA Tesla V100 (PCIe)</li>
<li>Ubuntu 18.04.1 LTS (Bionic Beaver)</li>
<li>IBM Spectrum LSF Suite for Enterprise</li>
<li>NVIDIA CUDA 9.2</li>
</ul>
<p>Note that the following assumes that NVIDIA CUDA and IBM Spectrum LSF Suite for Enterprise are installed and functioning nominally.</p>

<p>By default, the latest version of IBM Spectrum LSF Suite v10.2.0.6 has the following parameters enabled by default in
<em>$LSF_ENVDIR/lsf.conf</em>:</p>

<div class="highlight"><pre><code class="language-bash">LSF_GPU_AUTOCONFIG<span style="color: #f92672;">=</span>Y
LSB_GPU_NEW_SYNTAX<span style="color: #f92672;">=</span>extend</code></pre></div>

<p>The above parameters enable the new GPU support wizardry in the product.</p>

<ol>
<li>So let&rsquo;s get right into it.  We start by checking if the Spectrum LSF cluster is up and running.</li>
</ol>
<div class="highlight"><pre><code class="language-bash">test@kilenc:~$ lsid
IBM Spectrum LSF 10.1.0.6, May <span style="color: #ae81ff;">25</span> <span style="color: #ae81ff;">2018</span>
Suite Edition: IBM Spectrum LSF Suite <span style="color: #66d9ef;">for</span> Enterprise 10.2.0
Copyright International Business Machines Corp. 1992, 2016.
US Government Users Restricted Rights - Use, duplication or disclosure restricted by GSA ADP Schedule Contract with IBM Corp.

My cluster name is Klaszter
My master name is kilenc

 

test@kilenc:~$ lsload
HOST_NAME       status  r15s   r1m  r15m   ut    pg  ls    it   tmp   swp   mem
kilenc              ok   0.6   0.3   0.3   0%   0.0   <span style="color: #ae81ff;">1</span>    <span style="color: #ae81ff;">18</span>  791G  853M  7.3G

 

test@kilenc:~$ bhosts
HOST_NAME          STATUS       JL/U    MAX  NJOBS    RUN  SSUSP  USUSP    RSV 
kilenc             ok              -     <span style="color: #ae81ff;">32</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span>      <span style="color: #ae81ff;">0</span></code></pre></div>

<p>We confirm above that the status of the cluster is OK.  Meaning it&rsquo;s up and ready to accept jobs.  Note that I have not done any
supplementary configuration in Spectrum LSF for GPUs apart from enabling the two above variables noted above.</p>

<ol start="2">
<li>Eureka!  Spectrum LSF has automatically detected the presence of GPUs on the system.  The single GPU in this case is now
configured as a resource for Spectrum LSF and can be scheduled to.  We have used the new -<em>gpu</em> and -<em>gpuload</em> options for the
Spectrum LSF user commands to check this.</li>
</ol>
<div class="highlight"><pre><code class="language-bash">test@kilenc:~$ lshosts -gpu
HOST_NAME   gpu_id       gpu_model   gpu_driver   gpu_factor      numa_id
kilenc           <span style="color: #ae81ff;">0</span> TeslaV100_PCIE_       396.37          7.0            <span style="color: #ae81ff;">8</span>

 

test@kilenc:~$ lsload -gpu
HOST_NAME       status  ngpus  gpu_shared_avg_mut  gpu_shared_avg_ut  ngpus_physical
kilenc              ok      <span style="color: #ae81ff;">1</span>                  0%                 0%               <span style="color: #ae81ff;">1</span>
 

test@kilenc:~$ lsload -gpuload
HOST_NAME       gpuid   gpu_model   gpu_mode  gpu_temp   gpu_ecc  gpu_ut  gpu_mut gpu_mtotal gpu_mused   gpu_pstate   gpu_status   gpu_error
kilenc              <span style="color: #ae81ff;">0</span> TeslaV100_P        0.0       46C       0.0      0%       0%      31.7G        0M            <span style="color: #ae81ff;">0</span>           ok           - </code></pre></div>

<p>As we can see above, Spectrum LSF has correctly detected the presence of the single Telsa V100 which is present in the node.  It&rsquo;s
also displaying a number of metrics about the CPU including mode, temperature, and memory.</p>

<ol start="3">
<li>Next, let&rsquo;s submit some GPU workloads to the environment.  I found the samples included with NVIDIA CUDA to be fairly short running
on the Tesla V100, so I turned to the trusty Multi-GPU CUDA stress test aka <em>gpu-burn</em>.  You can read more about that utility here.
To submit a job to GPU workload to Spectrum LSF, we use the -<em>gpu</em> option.  This can be used to specify the detailed requirements for
your GPU job including the number of GPUs, GPU mode, GPU model, etc.  For the purpose of this test, we&rsquo;ll use the default value &ldquo;-&rdquo;
which specifies the following options: &ldquo;<em>num=1:mode=shared:mps=no:j_exclusive=nonvlink=no</em>&rdquo;.</li>
</ol>
<div class="highlight"><pre><code class="language-bash">test@kilenc:~/gpu-burn$ bsub -gpu - ./gpu_burn <span style="color: #ae81ff;">300</span>
Job &lt;51662&gt; is submitted to default queue &lt;normal&gt;.</code></pre></div>

<ol start="4">
<li>Next we confirm that the job has started successfully.</li>
</ol>
<div class="highlight"><pre><code class="language-bash">test@kilenc:~/gpu-burn$ bjobs -l <span style="color: #ae81ff;">51662</span>

Job &lt;51662&gt;, User &lt;test&gt;, Project &lt;default&gt;, Status &lt;RUN&gt;, Queue &lt;normal&gt;, Comm
                     and &lt;./gpu_burn 300&gt;, Share group charged &lt;/test&gt;
Fri Sep <span style="color: #ae81ff;">14</span> 12:52:09: Submitted from host &lt;kilenc&gt;, CWD &lt;$HOME/gpu-burn&gt;, Reques
                     ted GPU;
Fri Sep <span style="color: #ae81ff;">14</span> 12:52:09: Started <span style="color: #ae81ff;">1</span> Task<span style="color: #f92672;">(</span>s<span style="color: #f92672;">)</span> on Host<span style="color: #f92672;">(</span>s<span style="color: #f92672;">)</span> &lt;kilenc&gt;, Allocated <span style="color: #ae81ff;">1</span> Slot<span style="color: #f92672;">(</span>s<span style="color: #f92672;">)</span>
                      on Host<span style="color: #f92672;">(</span>s<span style="color: #f92672;">)</span> &lt;kilenc&gt;, Execution Home &lt;/home/test&gt;, Executi
                     on CWD &lt;/home/test/gpu-burn&gt;;
Fri Sep <span style="color: #ae81ff;">14</span> 12:52:10: Resource usage collected.
                     MEM: <span style="color: #ae81ff;">4</span> Mbytes;  SWAP: <span style="color: #ae81ff;">0</span> Mbytes;  NTHREAD: <span style="color: #ae81ff;">3</span>
                     PGID: 95095;  PIDs: <span style="color: #ae81ff;">95095</span> <span style="color: #ae81ff;">95096</span> <span style="color: #ae81ff;">95097</span> 


 MEMORY USAGE:
 MAX MEM: <span style="color: #ae81ff;">4</span> Mbytes;  AVG MEM: <span style="color: #ae81ff;">4</span> Mbytes

 SCHEDULING PARAMETERS:
           r15s   r1m  r15m   ut      pg    io   ls    it    tmp    swp    mem
 loadSched   -     -     -     -       -     -    -     -     -      -      -  
 loadStop    -     -     -     -       -     -    -     -     -      -      -  

 EXTERNAL MESSAGES:
 MSG_ID FROM       POST_TIME      MESSAGE                             ATTACHMENT 
 <span style="color: #ae81ff;">0</span>      test       Sep <span style="color: #ae81ff;">14</span> 12:52   kilenc:gpus<span style="color: #f92672;">=</span>0;                          N     </code></pre></div>

<ol start="5">
<li>We cross confirm with the NVIDIA nvidia-smi command that the gpu-burn process is running on the GPU.</li>
</ol>
<div class="highlight"><pre><code class="language-bash">test@kilenc:~/gpu-burn$ nvidia-smi
Fri Sep <span style="color: #ae81ff;">14</span> 12:52:18 <span style="color: #ae81ff;">2018</span>       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 396.37                 Driver Version: 396.37                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|<span style="color: #f92672;">===============================</span>+<span style="color: #f92672;">======================</span>+<span style="color: #f92672;">======================</span>|
|   <span style="color: #ae81ff;">0</span>  Tesla V100-PCIE...  Off  | 00000033:01:00.0 Off |                    <span style="color: #ae81ff;">0</span> |
| N/A   68C    P0   247W / 250W |  29303MiB / 32510MiB |    100%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|<span style="color: #f92672;">=============================================================================</span>|
|    <span style="color: #ae81ff;">0</span>     <span style="color: #ae81ff;">95107</span>      C   ./gpu_burn                                 29292MiB |
+-----------------------------------------------------------------------------+</code></pre></div>

<ol start="6">
<li>Next we use the Spectrum LSF <em>lsload</em> command with the -<em>gpuload</em> option to check the GPU utilization.  This should closely match
what we see above.</li>
</ol>
<div class="highlight"><pre><code class="language-bash">test@kilenc:~/gpu-burn$ lsload -gpuload
HOST_NAME       gpuid   gpu_model   gpu_mode  gpu_temp   gpu_ecc  gpu_ut  gpu_mut gpu_mtotal gpu_mused   gpu_pstate   gpu_status   gpu_error
kilenc              <span style="color: #ae81ff;">0</span> TeslaV100_P        0.0       70C       0.0    100%      29%      31.7G     28.6G            <span style="color: #ae81ff;">0</span>           ok           -</code></pre></div>

<ol start="7">
<li>After 300 seconds (5 minutes), the job completes and exits without error.  We inspect the history of the job using the
Spectrum LSF <em>bhist</em> command which shows the changes in state of the job from start to finish.</li>
</ol>
<div class="highlight"><pre><code class="language-bash">test@kilenc:~/gpu-burn$ bhist -l <span style="color: #ae81ff;">51662</span>

Job &lt;51662&gt;, User &lt;test&gt;, Project &lt;default&gt;, Command &lt;./gpu_burn 300&gt;
Fri Sep <span style="color: #ae81ff;">14</span> 12:52:09: Submitted from host &lt;kilenc&gt;, to Queue &lt;normal&gt;, CWD &lt;$HOM
                     E/gpu-burn&gt;, Requested GPU;
Fri Sep <span style="color: #ae81ff;">14</span> 12:52:09: Dispatched <span style="color: #ae81ff;">1</span> Task<span style="color: #f92672;">(</span>s<span style="color: #f92672;">)</span> on Host<span style="color: #f92672;">(</span>s<span style="color: #f92672;">)</span> &lt;kilenc&gt;, Allocated <span style="color: #ae81ff;">1</span> Slot
                     <span style="color: #f92672;">(</span>s<span style="color: #f92672;">)</span> on Host<span style="color: #f92672;">(</span>s<span style="color: #f92672;">)</span> &lt;kilenc&gt;, Effective RES_REQ &lt;<span style="color: #66d9ef;">select</span><span style="color: #f92672;">[((</span>ngpus
                     &gt;0<span style="color: #f92672;">))</span> <span style="color: #f92672;">&amp;&amp;</span> <span style="color: #f92672;">(</span>type <span style="color: #f92672;">==</span> local<span style="color: #f92672;">)]</span> order<span style="color: #f92672;">[</span>gpu_maxfactor<span style="color: #f92672;">]</span> rusage<span style="color: #f92672;">[</span>ngpus
                     _physical<span style="color: #f92672;">=</span>1.00<span style="color: #f92672;">]</span> &gt;;
Fri Sep <span style="color: #ae81ff;">14</span> 12:52:09: External Message <span style="color: #e6db74;">"GPU_ALLOC="</span>kilenc<span style="color: #f92672;">{</span>0<span style="color: #f92672;">[</span>0:0<span style="color: #f92672;">]}</span><span style="color: #e6db74;">"GPU_MODELS="</span>Te
                     slaV100_PCIE_32GB-32510<span style="color: #f92672;">{</span>0<span style="color: #f92672;">[</span>0<span style="color: #f92672;">]}</span><span style="color: #e6db74;">"GPU_FACTORS="</span>7.0<span style="color: #f92672;">{</span>0<span style="color: #f92672;">[</span>0<span style="color: #f92672;">]}</span><span style="color: #e6db74;">"GPU_S
</span><span style="color: #e6db74;">                     OCKETS="</span>8<span style="color: #f92672;">{</span>0<span style="color: #f92672;">[</span>0<span style="color: #f92672;">]}</span><span style="color: #e6db74;">"GPU_NVLINK="</span>0<span style="color: #f92672;">[</span>0#0<span style="color: #f92672;">]</span><span style="color: #e6db74;">""</span> was posted from <span style="color: #e6db74;">"_sys
</span><span style="color: #e6db74;">                     tem_"</span> to message box 131;
Fri Sep <span style="color: #ae81ff;">14</span> 12:52:10: Starting <span style="color: #f92672;">(</span>Pid 95095<span style="color: #f92672;">)</span>;
Fri Sep <span style="color: #ae81ff;">14</span> 12:52:10: Running with execution home &lt;/home/test&gt;, Execution CWD &lt;/
                     home/test/gpu-burn&gt;, Execution Pid &lt;95095&gt;;
Fri Sep <span style="color: #ae81ff;">14</span> 12:52:10: External Message <span style="color: #e6db74;">"kilenc:gpus=0;EFFECTIVE GPU REQ: num=1:m
</span><span style="color: #e6db74;">                     ode=shared:mps=no:j_exclusive=no;"</span> was posted from <span style="color: #e6db74;">"test"</span> 
                     to message box 0;
Fri Sep <span style="color: #ae81ff;">14</span> 12:57:12: Done successfully. The CPU time used is 302.0 seconds;
Fri Sep <span style="color: #ae81ff;">14</span> 12:57:12: Post job process <span style="color: #66d9ef;">done</span> successfully;


MEMORY USAGE:
MAX MEM: <span style="color: #ae81ff;">220</span> Mbytes;  AVG MEM: <span style="color: #ae81ff;">214</span> Mbytes

Summary of time in seconds spent in various states by  Fri Sep <span style="color: #ae81ff;">14</span> 12:57:12
  PEND     PSUSP    RUN      USUSP    SSUSP    UNKWN    TOTAL
  <span style="color: #ae81ff;">0</span>        <span style="color: #ae81ff;">0</span>        <span style="color: #ae81ff;">303</span>      <span style="color: #ae81ff;">0</span>        <span style="color: #ae81ff;">0</span>        <span style="color: #ae81ff;">0</span>        <span style="color: #ae81ff;">303</span>         </code></pre></div>

<p>This has only been a teaser of the GPU support capabilities in Spectrum LSF.  Spectrum LSF also includes support for NVIDIA DCGM which
is used to collect GPU resource utilization per job.  But that&rsquo;s a topic for another blog :).  À la prochaine fois!</p>