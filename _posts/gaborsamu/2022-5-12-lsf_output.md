---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2022-05-12 13:16:02'
layout: post
original_url: https://www.gaborsamu.com/blog/lsf_output/
slug: customizing-command-output-in-ibm-spectrum-lsf
title: Customizing command output in IBM Spectrum LSF
---

<p><a href="https://www.ibm.com/products/hpc-workload-management">IBM Spectrum LSF</a> provides many ways to query the LSF cluster for information about workloads. As a user, once you’ve submitted a job to LSF, it’s logical to want to understand what has happened to your job.  Has the job started yet?  Is the job pending? If so, why is it pending? And the all important, “Is my job done yet?”.  Of course, LSF provides a very rich CLI which has been developed and refined - over the past three decades. It’s also possible to get JSON-formatted output from various LSF query commands. This is useful for users and administrators alike as JSON-formatted output is easy to parse, and scripting can be used to extract values from the JSON output.</p>

<p>This is not meant to be a definitive guide on how to query information in LSF, but rather provides some examples of the various ways that users can query job related information using the LSF CLI.  This will include a look at the <em>-json</em> and <em>-o <!-- raw HTML omitted --></em> options which have been introduced during the lifecycle of LSF v10.1.0 family. The <em>-json</em> option can be used to provide JSON-formatted output from various LSF query commands and the <em>-o <!-- raw HTML omitted --></em> can be used to customize the fields in the output to only those desired.</p>

<p>We’ll start with a simple job submission.  Here we submit a test workload as a non-root user in the LSF cluster.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bsub -o $HOME/output.%J -e $HOME/error.%J ./testjob.sh
Job &lt;24520&gt; is submitted to default queue &lt;normal&gt;.</code></pre></div>

<p>With the unique jobID number 24520, we can now query LSF for information about the job:</p>

<div class="highlight"><pre><code class="language-plaintext">$ bjobs 24520
JOBID   USER    STAT  QUEUE      FROM_HOST   EXEC_HOST   JOB_NAME   SUBMIT_TIME
24520   gsamu   RUN   normal     kilenc      kilenc      *estjob.sh May 10 21:09</code></pre></div>

<p>Adding the <em>-l</em> option to <em>bjobs</em> provides the long output (more details).</p>

<div class="highlight"><pre><code class="language-plaintext">$ bjobs -l 24520

Job &lt;24520&gt;, User &lt;gsamu&gt;, Project &lt;default&gt;, Status &lt;RUN&gt;, Queue &lt;normal&gt;, Com
                     mand &lt;./testjob.sh&gt;, Share group charged &lt;/gsamu&gt;
Tue May 10 21:09:22: Submitted from host &lt;kilenc&gt;, CWD &lt;$HOME&gt;, Output File &lt;/h
                     ome/gsamu/output.24520&gt;, Error File &lt;/home/gsamu/error.245
                     20&gt;;
Tue May 10 21:09:23: Started 1 Task(s) on Host(s) &lt;kilenc&gt;, Allocated 1 Slot(s)
                      on Host(s) &lt;kilenc&gt;, Execution Home &lt;/home/gsamu&gt;, Execut
                     ion CWD &lt;/home/gsamu&gt;;
Tue May 10 21:10:01: Resource usage collected.
                     MEM: 12 Mbytes;  SWAP: 0 Mbytes;  NTHREAD: 5
                     PGID: 313588;  PIDs: 313588 313589 313591 313592 


 MEMORY USAGE:
 MAX MEM: 12 Mbytes;  AVG MEM: 10 Mbytes

 SCHEDULING PARAMETERS:
           r15s   r1m  r15m   ut      pg    io   ls    it    tmp    swp    mem
 loadSched   -     -     -     -       -     -    -     -     -      -      -  
 loadStop    -     -     -     -       -     -    -     -     -      -      -  

 RESOURCE REQUIREMENT DETAILS:
 Combined: select[type == local] order[r15s:pg]
 Effective: select[type == local] order[r15s:pg] </code></pre></div>

<p>It is possible to customize the output format of the <em>bjobs</em> command using the <em>-o <!-- raw HTML omitted --></em> option. In this case, we want to show only some specific details about the job in the output of bjobs. We’ve selected to view: jobID, job status, project name, memory consumed, output and error files. A full list of the available fields for the custom format can be found <a href="https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=options-o">here</a>.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bjobs -o "jobid stat: queue:- project:10  mem:12:G output_file error_file" 24520
JOBID STAT       QUEUE PROJ_NAME  MEM          OUTPUT_FILE ERROR_FILE
24520 RUN       normal default    0.01 G       /home/gsamu/output.24520 /home/gsamu/error.24520</code></pre></div>

<p>Adding the <em>-json</em> option, it’s possible to get this customized job output in JSON format.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bjobs -o "jobid stat: queue:- project:10  mem:12:G output_file error_file" -json 24520
{
  "COMMAND":"bjobs",
  "JOBS":1,
  "RECORDS":[
    {
      "JOBID":"24520",
      "STAT":"RUN",
      "QUEUE":"normal",
      "PROJ_NAME":"default",
      "MEM":"0.01 G",
      "OUTPUT_FILE":"\/home\/gsamu\/output.24520",
      "ERROR_FILE":"\/home\/gsamu\/error.24520"
    }
  ]
}</code></pre></div>

<p>Next, let’s look at the <em>bhist</em> command. This can be used to view historical data about jobs.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bhist 24520
Summary of time in seconds spent in various states:
JOBID   USER    JOB_NAME  PEND    PSUSP   RUN     USUSP   SSUSP   UNKWN   TOTAL
24520   gsamu   *tjob.sh  1       0       457     0       0       0       458       </code></pre></div>

<p>We see that the job command has been truncated. Let’s now run <em>bhist</em> again with the <em>-w</em> option to produce a wide output.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bhist -w 24520
Summary of time in seconds spent in various states:
JOBID   USER    JOB_NAME  PEND    PSUSP   RUN     USUSP   SSUSP   UNKWN   TOTAL
24520   gsamu   ./testjob.sh 1       0       462     0       0       0       463       </code></pre></div>

<p>And finally, with the <em>-l</em> option to produce a long, detailed output.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bhist -l 24520

Job &lt;24520&gt;, User &lt;gsamu&gt;, Project &lt;default&gt;, Command &lt;./testjob.sh&gt;
Tue May 10 21:09:22: Submitted from host &lt;kilenc&gt;, to Queue &lt;normal&gt;, CWD &lt;$HOM
                     E&gt;, Output File &lt;/home/gsamu/output.%J&gt;, Error File &lt;/home
                     /gsamu/error.%J&gt;;
Tue May 10 21:09:23: Dispatched 1 Task(s) on Host(s) &lt;kilenc&gt;, Allocated 1 Slot
                     (s) on Host(s) &lt;kilenc&gt;, Effective RES_REQ &lt;select[type ==
                      local] order[r15s:pg] &gt;;
Tue May 10 21:09:25: Starting (Pid 313588);
Tue May 10 21:09:25: Running with execution home &lt;/home/gsamu&gt;, Execution CWD &lt;
                     /home/gsamu&gt;, Execution Pid &lt;313588&gt;;


Summary of time in seconds spent in various states by  Tue May 10 21:17:26
  PEND     PSUSP    RUN      USUSP    SSUSP    UNKWN    TOTAL
  1        0        483      0        0        0        484         </code></pre></div>

<p>When the job is done, the <em>bacct</em> command can be used to get detailed accounting information for jobs.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bacct 24520

Accounting information about jobs that are: 
  - submitted by all users.
  - accounted on all projects.
  - completed normally or exited
  - executed on all hosts.
  - submitted to all queues.
  - accounted on all service classes.
  - accounted to all RC accounts.
------------------------------------------------------------------------------

SUMMARY:      ( time unit: second ) 
 Total number of done jobs:       1      Total number of exited jobs:     0
 Total CPU time consumed:       3.4      Average CPU time consumed:     3.4
 Maximum CPU time of a job:     3.4      Minimum CPU time of a job:     3.4
 Total wait time in queues:     1.0
 Average wait time in queue:    1.0
 Maximum wait time in queue:    1.0      Minimum wait time in queue:    1.0
 Average turnaround time:       669 (seconds/job)
 Maximum turnaround time:       669      Minimum turnaround time:       669
 Average hog factor of a job:  0.01 ( cpu time / turnaround time )
 Maximum hog factor of a job:  0.01      Minimum hog factor of a job:  0.01
 Average expansion factor of a job:  1.00 ( turnaround time / run time )
 Maximum expansion factor of a job:  1.00
 Minimum expansion factor of a job:  1.00
 Total Run time consumed:       668      Average Run time consumed:     668
 Maximum Run time of a job:     668      Minimum Run time of a job:     668
 Scheduler Efficiency for 1 jobs
 Slot Utilization:          100.00%  Memory Utilization:            100.00% </code></pre></div>

<p>And now the long, detailed output from bacct using the <em>-l</em> parameter.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bacct -l 24520

Accounting information about jobs that are: 
  - submitted by all users.
  - accounted on all projects.
  - completed normally or exited
  - executed on all hosts.
  - submitted to all queues.
  - accounted on all service classes.
  - accounted to all RC accounts.
------------------------------------------------------------------------------

Job &lt;24520&gt;, User &lt;gsamu&gt;, Project &lt;default&gt;, Status &lt;DONE&gt;, Queue &lt;normal&gt;, Co
                     mmand &lt;./testjob.sh&gt;, Share group charged &lt;/gsamu&gt;
Tue May 10 21:09:22: Submitted from host &lt;kilenc&gt;, CWD &lt;$HOME&gt;, Output File &lt;/h
                     ome/gsamu/output.%J&gt;, Error File &lt;/home/gsamu/error.%J&gt;;
Tue May 10 21:09:23: Dispatched 1 Task(s) on Host(s) &lt;kilenc&gt;, Allocated 1 Slot
                     (s) on Host(s) &lt;kilenc&gt;, Effective RES_REQ &lt;select[type ==
                      local] order[r15s:pg] &gt;;
Tue May 10 21:20:31: Completed &lt;done&gt;.

Accounting information about this job:
     Share group charged &lt;/gsamu&gt;
     CPU_T     WAIT     TURNAROUND   STATUS     HOG_FACTOR    MEM    SWAP
      3.37        1            669     done         0.0050    12M      0M
------------------------------------------------------------------------------

SUMMARY:      ( time unit: second ) 
 Total number of done jobs:       1      Total number of exited jobs:     0
 Total CPU time consumed:       3.4      Average CPU time consumed:     3.4
 Maximum CPU time of a job:     3.4      Minimum CPU time of a job:     3.4
 Total wait time in queues:     1.0
 Average wait time in queue:    1.0
 Maximum wait time in queue:    1.0      Minimum wait time in queue:    1.0
 Average turnaround time:       669 (seconds/job)
 Maximum turnaround time:       669      Minimum turnaround time:       669
 Average hog factor of a job:  0.01 ( cpu time / turnaround time )
 Maximum hog factor of a job:  0.01      Minimum hog factor of a job:  0.01
 Average expansion factor of a job:  1.00 ( turnaround time / run time )
 Maximum expansion factor of a job:  1.00
 Minimum expansion factor of a job:  1.00
 Total Run time consumed:       668      Average Run time consumed:     668
 Maximum Run time of a job:     668      Minimum Run time of a job:     668
 Scheduler Efficiency for 1 jobs
 Slot Utilization:          100.00%  Memory Utilization:            100.00% </code></pre></div>

<p><strong>From jobs to queues</strong></p>

<p>We’ve looked briefly at querying LSF for job related information.  Let’s now take a closer look at querying LSF for information regarding the queue configuration.  Batch queues are where users submit jobs to. Queues can have a very wide array of attributes and settings.  Below we see a listing of the default queues configured in LSF Suite for HPC. The <em>bqueues</em> command is used to query LSF for the queue configuration.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bqueues
QUEUE_NAME      PRIO STATUS          MAX JL/U JL/P JL/H NJOBS  PEND   RUN  SUSP 
admin            50  Open:Active       -    -    -    -     0     0     0     0
owners           43  Open:Active       -    -    -    -     0     0     0     0
priority         43  Open:Active       -    -    -    -     0     0     0     0
night            40  Open:Active       -    -    -    -     0     0     0     0
short            35  Open:Active       -    -    -    -     0     0     0     0
dataq            33  Open:Active       -    -    -    -     0     0     0     0
normal           30  Open:Active       -    -    -    -     0     0     0     0
interactive      30  Open:Active       -    -    -    -     0     0     0     0
idle             20  Open:Active       -    -    -    -     0     0     0     0</code></pre></div>

<p>The <em>-l</em> option of <em>bqueues</em> can be used to get a more details view about the queues. Here, we look at the long output for the queue <em>normal</em>.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bqueues -l normal

QUEUE: normal
  -- For normal low priority jobs, running only if hosts are lightly loaded.  This is the default queue.

PARAMETERS/STATISTICS
PRIO NICE STATUS          MAX JL/U JL/P JL/H NJOBS  PEND   RUN SSUSP USUSP  RSV PJOBS 
 30    0  Open:Active       -    -    -    -     0     0     0     0     0    0     0
Interval for a host to accept two jobs is 0 seconds

SCHEDULING PARAMETERS
           r15s   r1m  r15m   ut      pg    io   ls    it    tmp    swp    mem
 loadSched   -     -     -     -       -     -    -     -     -      -      -  
 loadStop    -     -     -     -       -     -    -     -     -      -      -  

SCHEDULING POLICIES:  FAIRSHARE  NO_INTERACTIVE
USER_SHARES:  [default, 1] 

SHARE_INFO_FOR: normal/
 USER/GROUP   SHARES  PRIORITY  STARTED  RESERVED  CPU_TIME  RUN_TIME   ADJUST  GPU_RUN_TIME
gsamu           1       0.333      0        0         0.0        0       0.000             0
elasticsearch     1       0.333      0        0         0.0        0       0.000             0

USERS: all  
HOSTS:  all </code></pre></div>

<p>Custom output formatting can also be used for the <em>bqueues</em> command. Below is an example of the use of custom output formatting using the <em>-o <!-- raw HTML omitted --></em> parameter. For this example, we display queue name, status and the number of jobs (all states). More details about the <em>bqueues -o <!-- raw HTML omitted --></em> parameter can be found <a href="https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=reference-bqueues">here</a>.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bqueues -o "queue_name:12 status:12 njobs"
QUEUE_NAME   STATUS       NJOBS
admin        Open:Active  0
owners       Open:Active  0
priority     Open:Active  0
night        Open:Active  0
short        Open:Active  0
dataq        Open:Active  0
normal       Open:Active  0
interactive  Open:Active  0
idle         Open:Active  0</code></pre></div>

<p>And for JSON-formatted output, we add the <em>-json</em> parameter.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bqueues -json -o "queue_name:12 status:12 njobs"
{
  "COMMAND":"bqueues",
  "QUEUES":9,
  "RECORDS":[
    {
      "QUEUE_NAME":"admin",
      "STATUS":"Open:Active",
      "NJOBS":"0"
    },
    {
      "QUEUE_NAME":"owners",
      "STATUS":"Open:Active",
      "NJOBS":"0"
    },
    {
      "QUEUE_NAME":"priority",
      "STATUS":"Open:Active",
      "NJOBS":"0"
    },
    {
      "QUEUE_NAME":"night",
      "STATUS":"Open:Active",
      "NJOBS":"0"
    },
    {
      "QUEUE_NAME":"short",
      "STATUS":"Open:Active",
      "NJOBS":"0"
    },
    {
      "QUEUE_NAME":"dataq",
      "STATUS":"Open:Active",
      "NJOBS":"0"
    },
    {
      "QUEUE_NAME":"normal",
      "STATUS":"Open:Active",
      "NJOBS":"0"
    },
    {
      "QUEUE_NAME":"interactive",
      "STATUS":"Open:Active",
      "NJOBS":"0"
    },
    {
      "QUEUE_NAME":"idle",
      "STATUS":"Open:Active",
      "NJOBS":"0"
    }
  ]
}</code></pre></div>

<p><strong>From queues to servers</strong></p>

<p>Finally, we’ll look at the LSF <em>bhosts</em> command, which is used to display information about the batch hosts in the LSF cluster.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bhosts
HOST_NAME          STATUS       JL/U    MAX  NJOBS    RUN  SSUSP  USUSP    RSV 
archie             ok              -      2      0      0      0      0      0
kilenc             ok              -     32      0      0      0      0      0</code></pre></div>

<p>To view detailed information about a batch host, the <em>-l</em> parameter can be specified for <em>bhosts</em>.  Here we query for information on host <em>archie</em>.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bhosts -l archie
HOST  archie
STATUS           CPUF  JL/U    MAX  NJOBS    RUN  SSUSP  USUSP    RSV DISPATCH_WINDOW
ok               6.00     -      2      0      0      0      0      0      -

 CURRENT LOAD USED FOR SCHEDULING:
                r15s   r1m  r15m    ut    pg    io   ls    it   tmp   swp   mem  slots  ngpus
 Total           0.0   0.0   0.0    0%   0.0     1    1   437 3456M    0M  1.7G      2    0.0
 Reserved        0.0   0.0   0.0    0%   0.0     0    0     0    0M    0M    0M      -     - 

               ngpus_physical gpu_shared_avg_ut gpu_shared_avg_mut gpu_mode0
 Total                    0.0               0.0                0.0       0.0
 Reserved                  -                 -                  -         - 

               gpu_mode1 gpu_mode2 gpu_mode3 gpu_mode4 gpu_mode5 gpu_mode6
 Total               0.0       0.0       0.0       0.0       0.0       0.0
 Reserved             -         -         -         -         -         - 

               gpu_mode7 gpu_temp0 gpu_temp1 gpu_temp2 gpu_temp3 gpu_temp4
 Total               0.0       0.0       0.0       0.0       0.0       0.0
 Reserved             -         -         -         -         -         - 

               gpu_temp5 gpu_temp6 gpu_temp7 gpu_ecc0 gpu_ecc1 gpu_ecc2 gpu_ecc3
 Total               0.0       0.0       0.0      0.0      0.0      0.0      0.0
 Reserved             -         -         -        -        -        -        - 

               gpu_ecc4 gpu_ecc5 gpu_ecc6 gpu_ecc7 gpu_ut0 gpu_ut1 gpu_ut2 gpu_ut3
 Total              0.0      0.0      0.0      0.0     0.0     0.0     0.0     0.0
 Reserved            -        -        -        -       -       -       -       - 

               gpu_ut4 gpu_ut5 gpu_ut6 gpu_ut7 gpu_mut0 gpu_mut1 gpu_mut2 gpu_mut3
 Total             0.0     0.0     0.0     0.0      0.0      0.0      0.0      0.0
 Reserved           -       -       -       -        -        -        -        - 

               gpu_mut4 gpu_mut5 gpu_mut6 gpu_mut7 gpu_mtotal0 gpu_mtotal1
 Total              0.0      0.0      0.0      0.0         0.0         0.0
 Reserved            -        -        -        -           -           - 

               gpu_mtotal2 gpu_mtotal3 gpu_mtotal4 gpu_mtotal5 gpu_mtotal6
 Total                 0.0         0.0         0.0         0.0         0.0
 Reserved               -           -           -           -           - 

               gpu_mtotal7 gpu_mused0 gpu_mused1 gpu_mused2 gpu_mused3 gpu_mused4
 Total                 0.0        0.0        0.0        0.0        0.0        0.0
 Reserved               -          -          -          -          -          - 

               gpu_mused5 gpu_mused6 gpu_mused7 gpu_maxfactor
 Total                0.0        0.0        0.0           0.0
 Reserved              -          -          -             - 


 LOAD THRESHOLD USED FOR SCHEDULING:
           r15s   r1m  r15m   ut      pg    io   ls    it    tmp    swp    mem
 loadSched   -     -     -     -       -     -    -     -     -      -      -  
 loadStop    -     -     -     -       -     -    -     -     -      -      -  

 
 CONFIGURED AFFINITY CPU LIST: all</code></pre></div>

<p>Similar to <em>bjobs</em> and <em>bqueues</em>, the <em>-o <!-- raw HTML omitted --></em> parameter can be used for custom formatting of output of bhosts. Below is an example of the use of custom output formatting using the <em>-o <!-- raw HTML omitted --></em> parameter. For this example, we display host name, status and the number of jobs (all states). More details about the bqueues <em>-o <!-- raw HTML omitted --></em> parameter can be found <a href="https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=reference-bhosts">here</a>.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bhosts -o "host_name:12 status:12 njobs" 
HOST_NAME    STATUS       NJOBS
archie       ok           0
kilenc       ok           0</code></pre></div>

<p>And adding the <em>-json</em> parameter for JSON-formatted output.</p>

<div class="highlight"><pre><code class="language-text">$ bhosts -json -o "host_name:12 status:12 njobs" 
{
  "COMMAND":"bhosts",
  "HOSTS":2,
  "RECORDS":[
    {
      "HOST_NAME":"archie",
      "STATUS":"ok",
      "NJOBS":"0"
    },
    {
      "HOST_NAME":"kilenc",
      "STATUS":"ok",
      "NJOBS":"0"
    }
  ]
}</code></pre></div>

<p>That concludes our brief look at LSF query commands. We’ve only scratched the surface here in terms of capabilities and query commands for LSF. The LSF command line interface is powerful and flexible including ways to customize the command outputs and to output in JSON-format.  For more details, the complete set of IBM Spectrum LSF documentation can be found online at IBM Documentation <a href="https://www.ibm.com/docs/en/spectrum-lsf/10.1.0">here</a>.</p>