---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2024-03-19 20:40:52'
layout: post
original_url: https://www.gaborsamu.com/blog/lsf_rc_ibmcloud_part2/
slug: advanced-lsf-resource-connector-configuration-on-ibm-cloud-part-ii
title: Advanced LSF resource connector configuration on IBM Cloud - part II
---

<p><strong>Overview</strong></p>

<p>Back in November 2023 I authored a blog titled <a href="https://www.gaborsamu.com/blog/lsf_rc_ibmcloud_part1/">Advanced LSF resource connector configuration on IBM Cloud - part I</a>. As I signed off in that post, I mentioned that there would be a follow-on post to cover some more advanced configuration topics on LSF resource connector.  And that’s the topic of this article today.</p>

<p>To recap, the <a href="https://www.ibm.com/products/hpc-workload-management">IBM LSF</a> resource connector functionality enables LSF clusters to dynamically spin-up cloud instances from supported resource providers in the cloud based on workload demand, and to destroy those instances when no longer required.</p>

<p>The LSF resource connector intelligently choses the most appropriate cloud instance type for a given job from the templates that have been defined by the administrator. This is done automatically and is transparent from the end user perspective. What if the job requires to run on a very specific instance type? In this post we’ll show you how this is feasible by defining an LSF string resource, along with the necessary configuration of LSF resource connector and a supporting script. This will allow users to submit jobs to LSF with a resource requirement string specifying the cloud instance type desired.</p>

<p>For the example below, we’ll be using an IBM LSF environment which has been deployed on IBM Cloud using the extensive deployment automation that is available via the IBM Cloud catalog for <a href="https://cloud.ibm.com/catalog/content/terraform-1623200063-71606cab-c6e1-4f95-a47a-2ce541dcbed8-global">IBM LSF</a>. Using this automation, you can deploy an LSF cluster in about 10 minutes time including the creation of the virtual private cloud (VPC), networking, security, bastion node, NFS server node, LSF management nodes and optionally LSF Application Center.</p>

<p><strong>What is user_data.sh?</strong></p>

<p>We’ll start with a brief description of the LSF resource connector <em>user_data.sh</em> script. This script will play an important part in the configuration of the compute servers as we’ll see. The <em>user_data.sh</em> script is used to start-up the LSF daemons on the compute instances launched by LSF resource connector. It also crucially enables admins to configure settings, including LSF settings, which is what we’ll be using the in example below.</p>

<p><strong>Specifying the cloud instance type</strong></p>

<p>By default, the LSF resource connector intelligently chooses the cloud instance profile type based upon the job submission parameters. For example, it considers things like the number of processor requested, the memory requested just to name of few. And it will startup the compute instance or instances from the available configured templates which most closely matches the job requirement.</p>

<p>What if you need to request a very specific compute instance type for the work that you’ve submitted based upon other, site-specific needs?  Here we will show exactly how you can achieve this.</p>

<p><strong>Let the configuration begin!</strong></p>

<p>We begin with updating the LSF configuration to create a new string resource called profile. In the configuration file <em>$LSF_ENVDIR/lsf.shared</em>, define the new string resource profile in the <em>Resource</em> section.</p>

<div class="highlight"><pre><code class="language-plaintext">….
….
Begin Resource
RESOURCENAME	TYPE	    INTERVAL	INCREASING	DESCRIPTION        # Keywords
profile 	  String   ()       ()             (IBM Cloud Gen2 profile type)
End Resource
….
….</code></pre></div>

<p>To make the change take effect, reconfigure the LSF cluster with the LSF command <em>lsadmin reconfig</em>.</p>

<div class="highlight"><pre><code class="language-plaintext"># lsadmin reconfig -v

Checking configuration files ...


EGO 3.4.0 build 1599999, Jan 04 2023
Copyright International Business Machines Corp. 1992, 2016.
US Government Users Restricted Rights - Use, duplication or disclosure restricted by GSA ADP Schedule Contract with IBM Corp.

  binary type: linux3.10-glibc2.17-x86_64
Reading configuration from /opt/ibm/lsf/conf/lsf.conf
Mar 13 16:57:25 2024 1478621 6 3.4.0 Lim starting...
Mar 13 16:57:25 2024 1478621 6 3.4.0 LIM is running in advanced workload execution mode.
Mar 13 16:57:25 2024 1478621 6 3.4.0 Master LIM is not running in EGO_DISABLE_UNRESOLVABLE_HOST mode.
Mar 13 16:57:25 2024 1478621 5 3.4.0 /opt/ibm/lsf/10.1/linux3.10-glibc2.17-x86_64/etc/lim -C
Mar 13 16:57:26 2024 1478621 6 3.4.0 LIM is running as IBM Spectrum LSF Standard Edition.
Mar 13 16:57:26 2024 1478621 6 3.4.0 reCheckClass: numhosts 1 so reset exchIntvl to 15.00
Mar 13 16:57:26 2024 1478621 6 3.4.0 Checking Done.
---------------------------------------------------------
No errors found.

Restart only the master candidate hosts? [y/n] n
Do you really want to restart LIMs on all hosts? [y/n] y
Restart LIM on &lt;icgen2host-AAA-BBB-CCC-DDD&gt; ...... done</code></pre></div>

<p>Now, check that the profile variable has been setup properly. This can be done using the LSF <em>lsinfo</em> command.</p>

<div class="highlight"><pre><code class="language-plaintext"># lsinfo |grep profile
profile        String   N/A   IBM Cloud Gen2 profile type</code></pre></div>

<p>Now we’re ready to update the LSF resource connector templates to add the <em>profile</em> string variable. For this example, there are two templates defined for IBM Cloud profile types <em>bx2-4x16</em> and <em>mx2-16x128</em> in the configuration file <em>$LSF_ENVDIR/resource_connector/ibmcloudgen2/conf</em>. Within the template definition, the <em>profile</em> string variable is defined, and a value is set for each respective profile type. Note that the “-“ character cannot be used in the LSF string variables, and in place of that the “_” character is used. The specified profile string for each respective template is defined and used as the selection criteria by LSF resource connector. Then the <em>userData</em> field is used to ensure that this value gets passed and set in the compute instance that is started by the LSF resource connector when the <em>user_data.sh</em> script is run.</p>

<table>
<thead>
<tr>
<th style="text-align: left;">Instance type</th>
<th>LSF <em>profile</em> variable string value</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: left;">bx2-4x16</td>
<td>bx2_4x16</td>
</tr>
<tr>
<td style="text-align: left;">mx2-16x128</td>
<td>mx2_16x128</td>
</tr>
</tbody>
</table>
<hr />
<p><strong><em>ibmcloudgen2_templates.json</em>, with <em>profile</em> configured (obfuscated)</strong></p>

<div class="highlight"><pre><code class="language-plaintext">{
    "templates": [
        {
            "templateId": "Template-1",
            "maxNumber": 2,
            "attributes": {
                "type": ["String", "X86_64"],
                "ncores": ["Numeric", "2"],
                "ncpus": ["Numeric", "4"],
                "mem": ["Numeric", "16384"],
                "icgen2host": ["Boolean", "1"], 
                "profile":["String","bx2_4x16"]
            },
            "imageId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "subnetId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "vpcId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "vmType": "bx2-4x16",
            "userData":"profile=bx2_4x16",
            "securityGroupIds": ["aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff"],
            "resourceGroupId": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
            "sshkey_id": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "priority": "10", 
            "region": "us-east",
            "zone": "us-east-1" 
        },

       {
            "templateId": "Template-2",
            "maxNumber": 2,
            "attributes": {
                "type": ["String", "X86_64"],
                "ncores": ["Numeric", "8"],
                "ncpus": ["Numeric", "16"],
                "mem": ["Numeric", "131072"],
                "icgen2host": ["Boolean", "1"],
		       "profile":["String","mx2_16x128"]
            },
            "imageId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "subnetId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "vpcId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "vmType": "mx2-16x128",
            "userData":"profile=mx2_16x128",
            "securityGroupIds": ["aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff"],
            "resourceGroupId": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
            "sshkey_id": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "priority": "5",
            "region": "us-east",
            "zone": "us-east-1"
        }

    ]
}</code></pre></div>

<p>Now the <em>user_data.sh</em> script is required to be updated in order to set the value of the <em>profile</em> variable in the LSF resourcemap based upon what was requested by the user. This will be added to the LSF configuration during the bootup of the dynamic cloud instances. For more information about the LSF resourcemap read <a href="https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=resources-configure-lsfclustercluster-name-resourcemap-section">here</a>.</p>

<p><strong>user_data.sh script portion</strong></p>

<div class="highlight"><pre><code class="language-plaintext">….
….
# Set value of profile variable in the LSF resourcemap. This is based
# on the profile value requested at job submission time. 
if [ -n "$profile" ]; then
sed -i "s/\(LSF_LOCAL_RESOURCES=.*\)\"/\1 [resourcemap $profile*profile]\"/" $LSF_CONF_FILE
echo "update LSF_LOCAL_RESOURCES in $LSF_CONF_FILE successfully, add [resourcemap ${profile}*profile]" &gt;&gt; $logfile
else
echo "profile doesn't exist in environment variable" &gt;&gt; $logfile
fi
….
….</code></pre></div>

<p>With all of the configuration in place, it’s now time to test things out. Initially, a stress job is submitted requesting 4 cores is submitted without requesting a specific compute profile. In this case, the LSF resource connector will chose the most appropriate instance type from the configured templates. In our configuration the templates for instance types <em>bx2-4x16</em> and <em>mx2-16x128</em> are configured. Given this, we expect the LSF resource connector to startup a <em>bx2-4x16</em> instance to satisfy the requirements for this example job.</p>

<div class="highlight"><pre><code class="language-plaintext">$ bsub -n 4 -q normal -o /mnt/data/%J.out /usr/bin/stress --cpu 4 --vm-bytes 8192MB --timeout 60s 
Job &lt;3811&gt; is submitted to queue &lt;normal&gt;.</code></pre></div>

<p>After a few moments, we see that a new host, <em>icgen2host-XXX-YYY-ZZZ-44</em> joins the LSF cluster and the job enters run state. We note that this is a host with the characteristics of 4 cores, and 16GB RAM, which matches the type <em>bx2-4x16</em>.</p>

<div class="highlight"><pre><code class="language-plaintext">$ lsload -w
HOST_NAME               status  r15s   r1m  r15m   ut    pg  ls    it   tmp   swp   mem
icgen2host-XXX-YYY-ZZZ-37      ok   0.4   0.1   0.1   2%   0.0   1    16   40G    0
icgen2host-XXX-YYY-ZZZ-44      ok   0.9   0.2   0.1  19%   0.0   0     0   88G    0M 14.9G

$ lshosts -w
HOST_NAME                       type       model  cpuf ncpus maxmem maxswp server RESOURCES
icgen2host-XXX-YYY-ZZZ-37        X86_64    Intel_E5  12.5     4  15.4G      -    Yes (mg docker ParaView)
icgen2host-XXX-YYY-ZZZ-44		 X86_64    Intel_E5  12.5     4  15.5G      -    Dyn (icgen2host docker)

$ bjobs -l -r

Job &lt;3811&gt;, User &lt;lsfadmin&gt;, Project &lt;default&gt;, Status &lt;RUN&gt;, Queue &lt;normal&gt;, C
                     ommand &lt;/usr/bin/stress --cpu 4 --vm-bytes 8192MB --timeou
                     t 60s&gt;, Share group charged &lt;/lsfadmin&gt;
Mon Mar 18 19:42:22: Submitted from host &lt;icgen2host-XXX-YYY-ZZZ-37&gt;, CWD &lt;$HOME&gt;,
                      Output File &lt;/mnt/data/3811.out&gt;, 4 Task(s);
Mon Mar 18 19:45:11: Started 4 Task(s) on Host(s) &lt;icgen2host-XXX-YYY-ZZZ-44&gt; &lt;icg
                     en2host-XXX-YYY-ZZZ-44&gt; &lt;icgen2host-XXX-YYY-ZZZ-44&gt; &lt;icgen2host-
                     XXX-YYY-ZZZ-44&gt;, Allocated 4 Slot(s) on Host(s) &lt;icgen2host-X
                     XX-YYY-ZZZ-44&gt; &lt;icgen2host-XXX-YYY-ZZZ-44&gt; &lt;icgen2host-XXX-YYY-ZZZ-
                     44&gt; &lt;icgen2host-XXX-YYY-ZZZ-44&gt;, Execution Home &lt;/home/lsfadm
                     in&gt;, Execution CWD &lt;/home/lsfadmin&gt;;
Mon Mar 18 19:45:47: Resource usage collected.
                     The CPU time used is 142 seconds.
                     MEM: 3 Mbytes;  SWAP: 0 Mbytes;  NTHREAD: 8
                     PGID: 2169;  PIDs: 2169 2170 2172 2173 2174 2175 2176 


 MEMORY USAGE:
 MAX MEM: 3 Mbytes;  AVG MEM: 3 Mbytes; MEM Efficiency: 0.00%

 CPU USAGE:
 CPU PEAK: 0.00 ;  CPU PEAK DURATION: 0 second(s)
 CPU AVERAGE EFFICIENCY: 0.00% ;  CPU PEAK EFFICIENCY: 0.00%

 SCHEDULING PARAMETERS:
           r15s   r1m  r15m   ut      pg    io   ls    it    tmp    swp    mem
 loadSched   -     -     -     -       -     -    -     -     -      -      -  
 loadStop    -     -     -     -       -     -    -     -     -      -      -  

 RESOURCE REQUIREMENT DETAILS:
 Combined: select[type == local] order[r15s:pg]
 Effective: select[type == local] order[r15s:pg] 

$ bhist -l 3811

Job &lt;3811&gt;, User &lt;lsfadmin&gt;, Project &lt;default&gt;, Command &lt;/usr/bin/stress --cpu 
                     4 --vm-bytes 8192MB --timeout 60s&gt;
Mon Mar 18 19:42:22: Submitted from host &lt;icgen2host-XXX-YYY-ZZZ-37&gt;, to Queue &lt;no
                     rmal&gt;, CWD &lt;$HOME&gt;, Output File &lt;/mnt/data/%J.out&gt;, 4 Task
                     (s);
Mon Mar 18 19:45:11: Dispatched 4 Task(s) on Host(s) &lt;icgen2host-XXX-YYY-ZZZ-44&gt; &lt;
                     icgen2host-XXX-YYY-ZZZ-44&gt; &lt;icgen2host-XXX-YYY-ZZZ-44&gt; &lt;icgen2ho
                     st-XXX-YYY-ZZZ-44&gt;, Allocated 4 Slot(s) on Host(s) &lt;icgen2hos
                     t-XXX-YYY-ZZZ-44&gt; &lt;icgen2host-XXX-YYY-ZZZ-44&gt; &lt;icgen2host-XXX-YYY
                     -ZZZ-44&gt; &lt;icgen2host-XXX-YYY-ZZZ-44&gt;, Effective RES_REQ &lt;select
                     [type == local] order[r15s:pg] &gt;;
Mon Mar 18 19:45:11: Starting (Pid 2169);
Mon Mar 18 19:45:11: Running with execution home &lt;/home/lsfadmin&gt;, Execution CW
                     D &lt;/home/lsfadmin&gt;, Execution Pid &lt;2169&gt;;
Mon Mar 18 19:46:11: Done successfully. The CPU time used is 239.3 seconds;
Mon Mar 18 19:46:12: Post job process done successfully;


MEMORY USAGE:
MAX MEM: 3 Mbytes;  AVG MEM: 2 Mbytes; MEM Efficiency: 0.00%

CPU USAGE:
CPU PEAK: 3.98 ;  CPU PEAK DURATION: 60 second(s)
CPU AVERAGE EFFICIENCY: 99.58% ;  CPU PEAK EFFICIENCY: 99.58%

Summary of time in seconds spent in various states by  Mon Mar 18 19:46:12
  PEND     PSUSP    RUN      USUSP    SSUSP    UNKWN    TOTAL
  169      0        60       0        0        0        229         </code></pre></div>

<p>Next, let’s submit the same job, but explicitly requesting the profile type <em>mx2-16x128</em>. To do this, we add the resource requirement requesting profile to be equal to <em>mx2_16x128</em> as follows:</p>

<div class="highlight"><pre><code class="language-plaintext">$ bsub -q normal -R "profile==mx2_16x128" -n 4 -o /mnt/data/%J.out /usr/bin/stress --cpu 4 --vm-bytes 8192MB --timeout 60s 
Job &lt;3813&gt; is submitted to queue &lt;normal&gt;.</code></pre></div>

<p>After a few moments, a dynamic host with 16 CPUs and 128 GB RAM joins the cluster, which corresponds to instance type <em>mx2-16x128</em>.</p>

<div class="highlight"><pre><code class="language-plaintext">$ lshosts -w
HOST_NAME                       type       model  cpuf ncpus maxmem maxswp server RESOURCES
icgen2host-XXX-YYY-ZZZ-37        X86_64    Intel_E5  12.5     4  15.4G      -    Yes (mg docker ParaView)
icgen2host-XXX-YYY-ZZZ-46        X86_64    Intel_E5  12.5    16 125.7G      -    Dyn (icgen2host docker)

$ lsload -w
HOST_NAME               status  r15s   r1m  r15m   ut    pg  ls    it   tmp   swp   mem
icgen2host-XXX-YYY-ZZZ-37      ok   0.0   0.1   0.1   3%   0.0   1     2   40G    0M 12.2G
icgen2host-XXX-YYY-ZZZ-46      ok   1.6   0.4   0.1  17%   0.0   0     0   88G    0M 123.8G

$ bjobs -l -r

Job &lt;3813&gt;, User &lt;lsfadmin&gt;, Project &lt;default&gt;, Status &lt;RUN&gt;, Queue &lt;normal&gt;, C
                     ommand &lt;/usr/bin/stress --cpu 4 --vm-bytes 8192MB --timeou
                     t 60s&gt;, Share group charged &lt;/lsfadmin&gt;
Mon Mar 18 20:27:52: Submitted from host &lt;icgen2host-XXX-YYY-ZZZ-37&gt;, CWD &lt;$HOME&gt;,
                      Output File &lt;/mnt/data/3813.out&gt;, 4 Task(s), Requested Re
                     sources &lt;profile==mx2_16x128&gt;;
Mon Mar 18 20:30:01: Started 4 Task(s) on Host(s) &lt;icgen2host-XXX-YYY-ZZZ-46&gt; &lt;icg
                     en2host-XXX-YYY-ZZZ-46&gt; &lt;icgen2host-XXX-YYY-ZZZ-46&gt; &lt;icgen2host-
                     XXX-YYY-ZZZ-46&gt;, Allocated 4 Slot(s) on Host(s) &lt;icgen2host                     -XXX-YYY-ZZZ-46&gt; &lt;icgen2host-XXX-YYY-ZZZ-46&gt; &lt;icgen2host-XXX-Y                     YY-ZZZ-46&gt; &lt;icgen2host-XXX-YYY-ZZZ-46&gt;, Execution Home &lt;/home/lsfadm
                     in&gt;, Execution CWD &lt;/home/lsfadmin&gt;;

 MEMORY USAGE:
 MEM Efficiency: 0.00%

 CPU USAGE:
 CPU PEAK: 0.00 ;  CPU PEAK DURATION: 0 second(s)
 CPU AVERAGE EFFICIENCY: 0.00% ;  CPU PEAK EFFICIENCY: 0.00%

 SCHEDULING PARAMETERS:
           r15s   r1m  r15m   ut      pg    io   ls    it    tmp    swp    mem
 loadSched   -     -     -     -       -     -    -     -     -      -      -  
 loadStop    -     -     -     -       -     -    -     -     -      -      -  

 RESOURCE REQUIREMENT DETAILS:
 Combined: select[(profile == mx2_16x128 ) &amp;&amp; (type == any)] order[r15s:pg]
 Effective: select[(profile == mx2_16x128 ) &amp;&amp; (type == any)] order[r15s:pg] </code></pre></div>

<p>Additionally, using the LSF lshosts command with the -s option, we can view information about the resources in the environment. We see in particular the profile resource is set to the value <em>mx2_16x128</em> for the dynamic host <em>icgen2host-XXX-YYY-ZZZ-46</em>.</p>

<div class="highlight"><pre><code class="language-plaintext">$ lshosts -s
RESOURCE                                VALUE       LOCATION
rc_account                            default       icgen2host-XXX-YYY-ZZZ-46 
profile                            mx2_16x128       icgen2host-XXX-YYY-ZZZ-46 
instanceID               0757_95e39240-22e7-4734-8fd5-9988ab247801  
                                                    icgen2host-XXX-YYY-ZZZ-46 </code></pre></div>

<p>We have shown the great flexibility that LSF provides for configuring the resource connector capability. Generally speaking, LSF provides many open interfaces which allow site specific configuration or customization to be realized. In the next blog in this series, we’ll take a closer look at running Docker jobs under LSF on dynamic cloud resources.</p>