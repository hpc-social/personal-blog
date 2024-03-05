---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2024-03-04 17:07:46'
layout: post
original_url: https://www.gaborsamu.com/blog/turingpi_hpl/
slug: pi-in-the-sky-a-compute-cluster-in-mini-itx-form-factor
title: Pi in the sky? A compute cluster in mini ITX form factor
---

<p><strong>Overview</strong></p>

<p>It&rsquo;s taken me a while to get the wheels off the ground in 2024 in terms of blogging. This blog idea has been in the works actually for some time. Back in 2021, I wrote a blog titled <a href="https://www.gaborsamu.com/blog/new_novena/">Late to the party and a few bits short</a>. This was a tongue in cheek title for a blog on the Novena Desktop System, which is based on a 32-bit processor, hence a few bits short. And late to the party referring to the fact that I was very late to purchase a second-hand Novena system.</p>

<p>This blog is similar in that it&rsquo;s about the original Turing Pi V1 system which was released back in 2021 when the Turing Pi V2 launch was imminent. The Turing Pi V1 is a 7 node cluster in a mini-ITX form factor. It&rsquo;s based on the Raspberry Pi CM3(+) modules. This was really an impulse purchase the dark days of COVID. And as I found out, getting a hold of RPi CM3&rsquo;s was much harder than expected. As luck would have it, I even eventually found a source via an online marketplace here in Southern Ontario that was not charging and arm and a leg for them. I purchased a total of 7 CM3+ modules with no onboard storage and relied upon SD cards for storage. As (bad) luck would have it, I ended up having to purchase a CM3 with onboard storage because one of the SD card slots is defecting on the board; the spring mechanism doesn&rsquo;t work properly. And as we&rsquo;ll see later on, this also had an unusual side effect when running Linpack.</p>

<p>I&rsquo;ve had the fully populated system for about 6 months now. And although the Turing Pi V1 is old news at this stage, I still wanted to write a bit about my experience with it. And of course, because it&rsquo;s a cluster, I definitely wanted to put it through it&rsquo;s paces running Linpack.</p>

<p>The official Turing Pi V1 <a href="https://docs.turingpi.com/docs/turing-pi1-intro-specs">documentation</a> was my goto for the system setup. The cluster was installed with the latest (at the time) Raspberry Pi OS (<em>2023-02-21-raspios-bullseye-arm64-lite.img</em>) based on Debian 11 (Bullseye).</p>

<p>The following additional software packages were installed/compiled. Note that the head node of the cluster acts as an NFS server for the remaining cluster nodes (<em>/opt</em>).</p>

<ul>
<li>Arm Optimizing Compilers V22.0.2 (for Ubuntu-20.04)</li>
<li>OpenMPI V4.1.5 (compiled with LSF support)</li>
<li>IBM Spectrum LSF v10.1.0.13</li>
<li>HPL V2.3 (compiled with Arm Optimizing Compilers)</li>
</ul>
<p>Here is the output of the LSF lshosts command. We see 6 CM3+ systems detected, and one CM3. Note that this required additional LSF configuration.</p>

<div class="highlight"><pre><code class="language-plaintext">lsfadmin@turingpi:/opt/HPC/hpl-2.3 $ lshosts -w
HOST_NAME                       type       model  cpuf ncpus maxmem maxswp server RESOURCES
turingpi                  LINUX_ARM64     CM3plus   6.0     4   910M   100M    Yes (mg)
neumann                   LINUX_ARM64     CM3plus   6.0     4   910M   100M    Yes ()
teller                    LINUX_ARM64     CM3plus   6.0     4   910M   100M    Yes ()
szilard                   LINUX_ARM64     CM3plus   6.0     4   910M   100M    Yes ()
wigner                    LINUX_ARM64     CM3plus   6.0     4   910M   100M    Yes ()
kemeny                    LINUX_ARM64         CM3   5.0     4   910M   100M    Yes ()
vonkarman                 LINUX_ARM64     CM3plus   6.0     4   910M   100M    Yes ()</code></pre></div>

<p>Those with a keen eye will note that the majority of the cluster nodes are named after Hungarian scient
ists:</p>

<ul>
<li><em>Neumann János</em> (John von Neumann)</li>
<li><em>Teller Ede</em> (Edward Teller)</li>
<li><em>Szilárd Leó</em> (Leo Szilard)</li>
<li><em>Wigner Jenő</em> (Eugene Wigner)</li>
<li><em>Kemény János</em> (John Kemeny)</li>
<li><em>Kármán Tódor</em> (Theodore von Karman)</li>
</ul>
<p>The odd one out here is of course turingpi, which is the name of the head node of the cluster, and is o
f course named after Alan Turing. But I digress.</p>

<p>For completeness, HPL V2.3 was compiled using the Arm Optimizing Compilers with the follwing flags:</p>

<ul>
<li>CCFLAGS      = $(HPL_DEFS) -Ofast -mcpu=native -fomit-frame-pointer</li>
<li>LINKER       = armclang -armpl -lamath -lm -Ofast -mcpu=native -fomit-frame-pointer</li>
</ul>
<p>For the first HPL run, we submit the job requesting a total of 24 cores. There are a total of 28 cores
in the cluster, but we&rsquo;ve isolated the head node of the cluster as it&rsquo;s the NFS server for the environm
ent. We see that the head node turingpi shows a closed status here, meaning that it won&rsquo;t accept any jo
bs from LSF.</p>

<div class="highlight"><pre><code class="language-plaintext">lsfadmin@turingpi:/opt/HPC/hpl-2.3/bin/cm3_3 $ bhosts
HOST_NAME          STATUS       JL/U    MAX  NJOBS    RUN  SSUSP  USUSP    RSV 
kemeny             ok              -      4      0      0      0      0      0
neumann            ok              -      4      0      0      0      0      0
szilard            ok              -      4      0      0      0      0      0
teller             ok              -      4      0      0      0      0      0
turingpi           closed          -      4      0      0      0      0      0
vonkarman          ok              -      4      0      0      0      0      0
wigner             ok              -      4      0      0      0      0      0</code></pre></div>

<p><strong>Turing up the heat - literally</strong></p>

<p>Submit HPL using the LSF <em>bsub</em> command requesting 24 cores in the cluser with core affinity specified.</p>

<div class="highlight"><pre><code class="language-plaintext">lsfadmin@turingpi:/opt/HPC/hpl-2.3/bin/cm3_3 $ bsub -n 24 -R "affinity[core(1)]" -Is mpirun --mca btl_t
cp_if_exclude lo,docker0 ./xhpl
Job &lt;41861&gt; is submitted to default queue &lt;interactive&gt;.
&lt;&lt;Waiting for dispatch ...&gt;&gt;
&lt;&lt;Starting on neumann&gt;&gt;
================================================================================
HPLinpack 2.3  --  High-Performance Linpack benchmark  --   December 2, 2018
Written by A. Petitet and R. Clint Whaley,  Innovative Computing Laboratory, UTK
Modified by Piotr Luszczek, Innovative Computing Laboratory, UTK
Modified by Julien Langou, University of Colorado Denver
================================================================================

An explanation of the input/output parameters follows:
T/V    : Wall time / encoded variant.
N      : The order of the coefficient matrix A.
NB     : The partitioning blocking factor.
P      : The number of process rows.
Q      : The number of process columns.
Time   : Time in seconds to solve the linear system.
Gflops : Rate of execution for solving the linear system.

The following parameter values will be used:

N      :   15968 
NB     :      48       96      192 
PMAP   : Row-major process mapping
P      :       4        6 
Q      :       6        4 
PFACT  :   Right 
NBMIN  :       4 
NDIV   :       2 
RFACT  :   Crout 
BCAST  :  1ringM 
DEPTH  :       1 
SWAP   : Mix (threshold = 64)
L1     : transposed form
U      : transposed form
EQUIL  : yes
ALIGN  : 8 double precision words

--------------------------------------------------------------------------------

- The matrix A is randomly generated for each test.
- The following scaled residual check will be computed:
      ||Ax-b||_oo / ( eps * ( || x ||_oo * || A ||_oo + || b ||_oo ) * N )
- The relative machine precision (eps) is taken to be               1.110223e-16
- Computational tests pass if scaled residuals are less than                16.0

--------------------------------------------------------------------------
Primary job  terminated normally, but 1 process returned
a non-zero exit code. Per user-direction, the job has been aborted.
--------------------------------------------------------------------------
--------------------------------------------------------------------------
An MPI communication peer process has unexpectedly disconnected.  This
usually indicates a failure in the peer process (e.g., a crash or
otherwise exiting without calling MPI_FINALIZE first).

Although this local MPI process will likely now behave unpredictably
(it may even hang or crash), the root cause of this problem is the
failure of the peer -- that is what you need to investigate.  For
example, there may be a core file that you can examine.  More
generally: such peer hangups are frequently caused by application bugs
or other external events.

  Local host: teller
  Local PID:  2253
  Peer host:  kemeny
--------------------------------------------------------------------------
--------------------------------------------------------------------------
mpirun noticed that process rank 23 with PID 2448 on node kemeny exited on signal 4 (Illegal instructio
n).
--------------------------------------------------------------------------</code></pre></div>

<p>We see above that the MPI rank(s) fail on host kemeny, which happens to be the CM3 module (not CM3+). E
ven though I compiled HPL natively on kemeny this issue persists. So ultimately, the HPL run was limite
d to the 5 remaining CM3+ nodes (i.e. 20 cores).</p>

<p>Next, we submit HPL requesting 20 cores (all on CM3+ modules). Core affinity is specified, and we reque
st specifically the model type &ldquo;CM3plus&rdquo;. The job was submitted interactively and the output follows:</p>

<div class="highlight"><pre><code class="language-plaintext">lsfadmin@turingpi:/opt/HPC/hpl-2.3/bin/cm3_3 $ bsub -n 20 -Is -R "select[model==CM3plus] affinity[core(
1)]" mpirun --mca btl_tcp_if_exclude lo,docker0 ./xhpl
Job &lt;41865&gt; is submitted to default queue &lt;interactive&gt;.
&lt;&lt;Waiting for dispatch ...&gt;&gt;
&lt;&lt;Starting on vonkarman&gt;&gt;
================================================================================
HPLinpack 2.3  --  High-Performance Linpack benchmark  --   December 2, 2018
Written by A. Petitet and R. Clint Whaley,  Innovative Computing Laboratory, UTK
Modified by Piotr Luszczek, Innovative Computing Laboratory, UTK
Modified by Julien Langou, University of Colorado Denver
================================================================================

An explanation of the input/output parameters follows:
T/V    : Wall time / encoded variant.
N      : The order of the coefficient matrix A.
NB     : The partitioning blocking factor.
P      : The number of process rows.
Q      : The number of process columns.
Time   : Time in seconds to solve the linear system.
Gflops : Rate of execution for solving the linear system.

The following parameter values will be used:

N      :   15968 
NB     :      48       96      192 
PMAP   : Row-major process mapping
P      :       4        5 
Q      :       5        4 
PFACT  :   Right 
NBMIN  :       4 
NDIV   :       2 
RFACT  :   Crout 
BCAST  :  1ringM 
DEPTH  :       1 
SWAP   : Mix (threshold = 64)
L1     : transposed form
U      : transposed form
EQUIL  : yes
ALIGN  : 8 double precision words

--------------------------------------------------------------------------------

- The matrix A is randomly generated for each test.
- The following scaled residual check will be computed:
      ||Ax-b||_oo / ( eps * ( || x ||_oo * || A ||_oo + || b ||_oo ) * N )
- The relative machine precision (eps) is taken to be               1.110223e-16
- Computational tests pass if scaled residuals are less than                16.0

================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       15968    48     4     5             327.96             8.2776e+00
HPL_pdgesv() start time Sun Mar  3 20:29:45 2024

HPL_pdgesv() end time   Sun Mar  3 20:35:13 2024

--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   2.74851526e-03 ...... PASSED
================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       15968    96     4     5             315.71             8.5987e+00
HPL_pdgesv() start time Sun Mar  3 20:35:18 2024

HPL_pdgesv() end time   Sun Mar  3 20:40:34 2024

--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   3.82600703e-03 ...... PASSED
================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       15968   192     4     5             319.93             8.4854e+00
HPL_pdgesv() start time Sun Mar  3 20:40:38 2024

HPL_pdgesv() end time   Sun Mar  3 20:45:58 2024

--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   2.56990081e-03 ...... PASSED
================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       15968    48     5     4             342.36             7.9293e+00
HPL_pdgesv() start time Sun Mar  3 20:46:03 2024

HPL_pdgesv() end time   Sun Mar  3 20:51:45 2024

--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   2.89956630e-03 ...... PASSED
================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       15968    96     5     4             313.72             8.6531e+00
HPL_pdgesv() start time Sun Mar  3 20:51:50 2024

HPL_pdgesv() end time   Sun Mar  3 20:57:04 2024

--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   3.04113830e-03 ...... PASSED
================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       15968   192     5     4             312.48             8.6877e+00
HPL_pdgesv() start time Sun Mar  3 20:57:08 2024

HPL_pdgesv() end time   Sun Mar  3 21:02:21 2024

--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   3.30812017e-03 ...... PASSED
================================================================================

Finished      6 tests with the following results:
              6 tests completed and passed residual checks,
              0 tests completed and failed residual checks,
              0 tests skipped because of illegal input values.
--------------------------------------------------------------------------------

End of Tests.
================================================================================</code></pre></div>

<p>We oberved during the HPL run that the CPU temperatures exceeded 80 degrees Celsius:</p>

<div class="highlight"><pre><code class="language-plaintext">root@turingpi:/home/lsfadmin# parallel-ssh -h /opt/workers -i "/opt/tools/cputemp.sh"
[1] 20:47:30 [SUCCESS] kemeny
Current CPU temperature is 61.22 degrees Celsius.
[2] 20:47:30 [SUCCESS] teller
Current CPU temperature is 82.21 degrees Celsius.
[3] 20:47:30 [SUCCESS] wigner
Current CPU temperature is 82.74 degrees Celsius.
[4] 20:47:31 [SUCCESS] szilard
Current CPU temperature is 82.21 degrees Celsius.
[5] 20:47:31 [SUCCESS] neumann
Current CPU temperature is 82.74 degrees Celsius.
[6] 20:47:31 [SUCCESS] vonkarman
Current CPU temperature is 83.28 degrees Celsius.

root@turingpi:/home/lsfadmin# parallel-ssh -h /opt/workers -i "/usr/bin/vcgencmd measure_clock arm" 
[1] 20:47:42 [SUCCESS] kemeny
frequency(48)=1199998000
[2] 20:47:43 [SUCCESS] szilard
frequency(48)=1034000000
[3] 20:47:43 [SUCCESS] teller
frequency(48)=980000000
[4] 20:47:44 [SUCCESS] wigner
frequency(48)=926000000
[5] 20:47:44 [SUCCESS] neumann
frequency(48)=818000000
[6] 20:47:44 [SUCCESS] vonkarman
frequency(48)=872000000</code></pre></div>

<p>And of course with high temperatures come CPU throttling. Clearly, with this thermal situation the run
of HPL was not going to be optimal.</p>

<p><strong>Giant Tiger to the rescue</strong></p>

<p>Even for those in Canada this may see like a very strange reference. <a href="https://www.gianttiger.com/">Giant Tiger</a> is a discount store ch
ain which sell everything from A through Z. Unfortunately the local &ldquo;GT Boutique&rdquo; as call it closed dow
n this past January. I happened to purchase on a whim a USB powered desktop fan at the GT Boutique abou
t a year ago. The idea was to help keep me cool at my keyboard during the hot summer days. But in this
case, it was just what was needed to provide a bit of active cooling to the Turing Pi system.</p>

<p>Repeating the run of HPL with the &ldquo;highly advanced active cooling&rdquo; measures in place, we were able to u
p the HPL results a tad while helping to preserve the life of the cluster nodes. And the results show g
oing from 8.65 GFlops with passive cooling to 9.5 GFlops with the active cooling.</p>

<div class="highlight"><pre><code class="language-plaintext">lsfadmin@turingpi:/opt/HPC/hpl-2.3/bin/cm3_3 $ bsub -n 20 -Is -R "select[model==CM3plus] affinity[core(
1)]" mpirun --mca btl_tcp_if_exclude lo,docker0 ./xhpl
Job &lt;41866&gt; is submitted to default queue &lt;interactive&gt;.
&lt;&lt;Waiting for dispatch ...&gt;&gt;
&lt;&lt;Starting on teller&gt;&gt;
================================================================================
HPLinpack 2.3  --  High-Performance Linpack benchmark  --   December 2, 2018
Written by A. Petitet and R. Clint Whaley,  Innovative Computing Laboratory, UTK
Modified by Piotr Luszczek, Innovative Computing Laboratory, UTK
Modified by Julien Langou, University of Colorado Denver
================================================================================

An explanation of the input/output parameters follows:
T/V    : Wall time / encoded variant.
N      : The order of the coefficient matrix A.
NB     : The partitioning blocking factor.
P      : The number of process rows.
Q      : The number of process columns.
Time   : Time in seconds to solve the linear system.
Gflops : Rate of execution for solving the linear system.

The following parameter values will be used:

N      :   15968 
NB     :      48       96      192 
PMAP   : Row-major process mapping
P      :       4        5 
Q      :       5        4 
PFACT  :   Right 
NBMIN  :       4 
NDIV   :       2 
RFACT  :   Crout 
BCAST  :  1ringM 
DEPTH  :       1 
SWAP   : Mix (threshold = 64)
L1     : transposed form
U      : transposed form
EQUIL  : yes
ALIGN  : 8 double precision words

--------------------------------------------------------------------------------

- The matrix A is randomly generated for each test.
- The following scaled residual check will be computed:
      ||Ax-b||_oo / ( eps * ( || x ||_oo * || A ||_oo + || b ||_oo ) * N )
- The relative machine precision (eps) is taken to be               1.110223e-16
- Computational tests pass if scaled residuals are less than                16.0

================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       15968    48     4     5             319.43             8.4985e+00
HPL_pdgesv() start time Sun Mar  3 21:15:42 2024

HPL_pdgesv() end time   Sun Mar  3 21:21:01 2024

--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   2.74851526e-03 ...... PASSED
================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       15968    96     4     5             296.94             9.1423e+00
HPL_pdgesv() start time Sun Mar  3 21:21:05 2024

HPL_pdgesv() end time   Sun Mar  3 21:26:02 2024

--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   3.82600703e-03 ...... PASSED
================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       15968   192     4     5             289.03             9.3926e+00
HPL_pdgesv() start time Sun Mar  3 21:26:06 2024

HPL_pdgesv() end time   Sun Mar  3 21:30:55 2024
--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   2.56990081e-03 ...... PASSED
================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       15968    48     5     4             316.20             8.5855e+00
HPL_pdgesv() start time Sun Mar  3 21:30:59 2024

HPL_pdgesv() end time   Sun Mar  3 21:36:15 2024

--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   2.89956630e-03 ...... PASSED
================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       15968    96     5     4             285.87             9.4961e+00
HPL_pdgesv() start time Sun Mar  3 21:36:19 2024

HPL_pdgesv() end time   Sun Mar  3 21:41:05 2024

--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   3.04113830e-03 ...... PASSED
================================================================================
T/V                N    NB     P     Q               Time                 Gflops
--------------------------------------------------------------------------------
WR11C2R4       15968   192     5     4             284.69             9.5355e+00
HPL_pdgesv() start time Sun Mar  3 21:41:09 2024

HPL_pdgesv() end time   Sun Mar  3 21:45:53 2024

--------------------------------------------------------------------------------
||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=   3.30812017e-03 ...... PASSED
================================================================================

Finished      6 tests with the following results:
              6 tests completed and passed residual checks,
              0 tests completed and failed residual checks,
              0 tests skipped because of illegal input values.
--------------------------------------------------------------------------------

End of Tests.
================================================================================</code></pre></div>

<p>And during the runtime, we see that no throttling occurrred and the CPU temperatures hovered in the hig
h 50&rsquo;s to low 60 degree Celsius range.</p>

<div class="highlight"><pre><code class="language-plaintext">root@turingpi:/home/lsfadmin# parallel-ssh -h /opt/workers -i "/opt/tools/cputemp.sh"
[1] 21:41:25 [SUCCESS] kemeny
Current CPU temperature is 36.48 degrees Celsius.
[2] 21:41:25 [SUCCESS] teller
Current CPU temperature is 58.53 degrees Celsius.
[3] 21:41:25 [SUCCESS] vonkarman
Current CPU temperature is 58.00 degrees Celsius.
[4] 21:41:25 [SUCCESS] neumann
Current CPU temperature is 55.84 degrees Celsius.
[5] 21:41:25 [SUCCESS] szilard
Current CPU temperature is 61.76 degrees Celsius.
[6] 21:41:25 [SUCCESS] wigner
Current CPU temperature is 55.31 degrees Celsius.

root@turingpi:/home/lsfadmin# parallel-ssh -h /opt/workers -i "/usr/bin/vcgencmd measure_clock arm" 
[1] 21:41:29 [SUCCESS] kemeny
frequency(48)=1200000000
[2] 21:41:29 [SUCCESS] teller
frequency(48)=1200000000
[3] 21:41:29 [SUCCESS] vonkarman
frequency(48)=1200000000
[4] 21:41:29 [SUCCESS] wigner
frequency(48)=1200000000
[5] 21:41:29 [SUCCESS] neumann
frequency(48)=1200000000
[6] 21:41:29 [SUCCESS] szilard
frequency(48)=1200002000</code></pre></div>

<p><strong>Wrap up</strong></p>

<p>I always liked the idea of a small cluster that you could easily take with you. That&rsquo;s why I&rsquo;m strongly
considering the Turing Pi V2.5, which can work with the much more powerful CM4 omdules, among other ve
ry capable modules. Budget allowing, I hope to purchase a Turing Pi V2.5 sometime in 2024. As always st
ay tuned for more exciting high performance computing tales. And at the end of the day, a compute clust
er in a mini ITX format isn&rsquo;t a pie in the sky idea. For me, it&rsquo;s a great tool for learning!</p>