---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2024-08-01 13:08:20'
layout: post
original_url: https://www.gaborsamu.com/blog/lsf_rc_ibmcloud_part3/
slug: advanced-lsf-resource-connector-configuration-on-ibm-cloud-part-iii
title: Advanced LSF resource connector configuration on IBM Cloud - part III
---

<p><strong>Overview</strong></p>

<p>This is the third instalment in a series of blogs covering advanced configuration topics for LSF resource connector. The earlier parts in the series can be found here: <a href="https://community.ibm.com/community/user/cloud/blogs/gbor-samu/2023/11/09/advanced-resource-connector-configuration-on-ibm-c">part I</a>, <a href="https://community.ibm.com/community/user/cloud/blogs/gbor-samu/2024/03/20/advanced-lsf-resource-connector-configuration-on-i">part II</a>.</p>

<p>As hinted in the closing of part II, this instalment will cover running Docker workloads on cloud instances which are dynamically managed by the LSF resource connector. The cloud environment in this example is <a href="https://cloud.ibm.com/catalog/content/terraform-1623200063-71606cab-c6e1-4f95-a47a-2ce541dcbed8-global">IBM Cloud</a>. To understand more about LSF resource connector, please read the earlier parts in the blog series.</p>

<p><a href="https://www.ibm.com/products/hpc-workload-management">LSF</a> provides a framework for the management and execution of containerized workloads. It supports the following container runtimes: Docker, NVIDIA Docker, Shifter, Singularity, Podman and Enroot. The LSF <a href="https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=lsf-configuring-containers">documentation</a> provides configuration steps for the supported container runtimes. Once configured, this capability is effectively transparent from the end user perspective.</p>

<p><strong>Enable Docker support</strong></p>

<p>First we need to enable support in LSF to run Docker containers. This is covered in detail in the LSF <a href="https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=containers-lsf-docker">documentation</a> and also something which I wrote about previously in the blog post <a href="https://medium.com/ibm-data-ai/jack-of-all-containers-e0d7fd0633b3">Jack of all containers</a>. The following steps will assume that the configuration steps have been completed.</p>

<p>LSF uses a Boolean resource named <em>docker</em> to identify hosts where the Docker runtime is available. This Boolean resource needs to be set on the compute nodes which are dynamically started by LSF resource connector.</p>

<p>In our example, an insecure Docker repository (using http) has been setup on the LSF manager host in the cluster with hostname <em>lsf-mgmt-host</em>. This will serve as the repository to host an OpenFOAM Docker container which has been prepared according to the procedures documented <a href="https://community.ibm.com/community/user/cloud/blogs/john-welch/2020/02/12/building-an-openfoam-ready-container-for-lsf">here</a>. This blog will not go into detail on the creation of the insecure Docker registry. On the LSF management node, below is the output showing the available images. We see the OpenFoam image is available both locally and via http on port 5000.</p>

<div class="highlight"><pre><code class="language-plaintext"># docker image ls
REPOSITORY                   TAG           IMAGE ID      CREATED        SIZE
localhost/openfoam/openfoam  v1912_update  bce4eb059f36  11 days ago    6.71 GB
localhost:5000/openfoam      v1912_update  bce4eb059f36  11 days ago    6.71 GB
docker.io/library/registry   2             6a3edb1d5eb6  10 months ago  26 MB</code></pre></div>

<p><strong>Note</strong> An insecure Docker registry was used in this example for simplicity and is not recommended in production.</p>

<p>As was the case in part II of the blog series, the <em>user_data.sh</em> script will be used for multiple purposes here:</p>

<ul>
<li>Set <em>docker</em> Boolean variable on dynamic compute nodes</li>
<li>Install Docker CE runtime and relevant support packages</li>
<li>Add user(s) to the docker group (<em>/etc/group</em>)</li>
<li>Configuration to point to insecure Docker registry on LSF management host
<em>lsf-mgmt-host</em></li>
</ul>
<p>The following updates were made to the <em>user_data.sh</em> script. See comments inline for details.</p>

<div class="highlight"><pre><code class="language-plaintext">$ diff -u4 ./user_data.sh ./user_data_sh.org
--- ./user_data.sh	2024-07-29 18:44:24.483146000 +0000
+++ ./user_data_sh.org	2024-07-11 14:34:47.688341000 +0000
@@ -29,25 +29,8 @@
 
 #!/bin/bash
 # shellcheck disable=all
 
-# 
-# The following steps will add the Docker CE repo, install the latest Docker CE
-# version along with supporting packages. It will create a Docker Linux group
-# and add the lsfadmin user to that group. Furthermore, it will create
-# the /etc/docker/daemon.json file pointing to the insecure Docker registry
-# which has been configured on the LSF management host. Finally it will
-# start Docker. Note that the hostname lsf-mgmt-host for the insecure-registries
-# configuration of Docker needs to be updated accordingly. 
-# 
-yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo -y 
-dnf install htop hwloc hwloc-libs libevent stress stress-ng python36 docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y  &gt;&gt; $logfile 2&gt;&amp;1
-ln -s /usr/bin/python3 /usr/bin/python
-groupadd docker &gt;&gt; $logfile 2&gt;&amp;1
-usermod -aG docker lsfadmin  &gt;&gt; $logfile 2&gt;&amp;1 
-echo -e "{\n \"insecure-registries\" : [ \”lsf-mgmt-host:5000\" ]\n }" &gt;&gt; /etc/docker/daemon.json 
-systemctl start docker &gt;&gt; $logfile 2&gt;&amp;1  
-
 if [ "$compute_user_data_vars_ok" != "1" ]; then
   echo 2&gt;&amp;1 "fatal: vars block is missing"
   exit 1
 fi
@@ -225,15 +208,8 @@
 else
   echo "Can not get instance ID" &gt;&gt; $logfile
 fi
 
-# 
-# Add the docker Boolean variable to the LSF_LOCAL_RESOURCES variable in
-# the lsf.conf file on the compute hosts. This will ensure that the host
-# is tagged with the docker variable. 
-# 
-sed -i "s/\(LSF_LOCAL_RESOURCES=.*\)\"/\1 [resource docker]\"/" $LSF_CONF_FILE &gt;&gt; $logfile 2&gt;&amp;1 
-
 #Update LSF Tuning on dynamic hosts
 LSF_TUNABLES="etc/sysctl.conf"
 echo 'vm.overcommit_memory=1' &gt;&gt; $LSF_TUNABLES
 echo 'net.core.rmem_max=26214400' &gt;&gt; $LSF_TUNABLES</code></pre></div>

<p><strong>Application profile configuration</strong></p>

<p>Next, we configure the LSF application profile for the OpenFOAM Docker container which has been loaded into the insecure Docker registry on the LSF management host. LSF application profiles can be used to define common job parameters for the same job type. This includes the container and container runtime definition. Learn more about LSF application profiles <a href="https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=lsf-application-profiles">here</a>.</p>

<p>On the LSF management node, the following application profile is defined in <em>$LSF_ENVDIR/lsbatch/&lt;clustername&gt;/configdir/lsb.applications</em>. Note that the hostname <em>lsf-mgmt-host</em> must point to the hostname where the insecure Docker repository has been setup in your environment. Additionally the volume specification <em>-v /mnt/vpcstorage/data</em> is specific to this environment and can be adjusted or removed as needed.</p>

<div class="highlight"><pre><code class="language-plaintext">….
….
Begin Application
NAME = openfoam
DESCRIPTION = Example OpenFOAM application
CONTAINER = docker[image(lsf-mgmt-host:5000/openfoam:v1912_update) \
   options(--rm --net=host --ipc=host \
   --cap-add=SYS_PTRACE \
   -v /etc/passwd:/etc/passwd \
   -v /etc/group:/etc/group \
   -v /mnt/vpcstorage/data:/mnt/vpcstorage/data \ 
   ) starter(root)]   
EXEC_DRIVER = context[user(lsfadmin)] \
   starter[/opt/ibm/lsf_worker/10.1/linux3.10-glibc2.17-x86_64/etc/docker-starter.py] \
   controller[/opt/ibm/lsf_worker/10.1/linux3.10-glibc2.17-x86_64/etc/docker-control.py] \
   monitor[/opt/ibm/lsf_worker/10.1/linux3.10-glibc2.17-x86_64/etc/docker-monitor.py]
End Application
….
….</code></pre></div>

<p>In order to make the above change take effect, run the <em>badmin reconfig</em> command as the defined LSF administrator. The LSF <em>bapp</em> command can be used to check the newly defined configuration for LSF application profile <em>openfoam</em>.</p>

<div class="highlight"><pre><code class="language-plaintext">$ badmin reconfig

Checking configuration files ...

No errors found.

Reconfiguration initiated

$ bapp -l openfoam

APPLICATION NAME: openfoam
 -- Example OpenFOAM application

STATISTICS:
   NJOBS     PEND      RUN    SSUSP    USUSP      RSV 
       0        0        0        0        0        0

PARAMETERS:

CONTAINER: docker[image(lsf-mgmt-host:5000/openfoam:v1912_update)    options(--rm --net=host --ipc=host    --cap-add=SYS_PTRACE    -v /etc/passwd:/etc/passwd    -v /etc/group:/etc/group    -v /mnt/vpcstorage/data:/mnt/vpcstorage/data    ) starter(root)]
EXEC_DRIVER: 
    context[user(lsfadmin)]
    starter[/opt/ibm/lsf_worker/10.1/linux3.10-glibc2.17-x86_64/etc/docker-starter.py]
    controller[/opt/ibm/lsf_worker/10.1/linux3.10-glibc2.17-x86_64/etc/docker-control.py]
    monitor[/opt/ibm/lsf_worker/10.1/linux3.10-glibc2.17-x86_64/etc/docker-monitor.py]</code></pre></div>

<p><strong>Submitting workload</strong></p>

<p>With all of the configuration in place, it’s now time to submit an OpenFOAM workload. For this, LSF Application Center is used. The OpenFOAM application template is available on the Spectrum Computing github <a href="https://github.com/IBMSpectrumComputing/lsf-integrations">here</a>. The OpenFOAM application template is configured to use the <em>openfoam</em> application profile. An example job is submitted and it runs to completion successfully. In the screenshot below, we see that the openfoam Docker container is executed.</p>

<figure><img src="https://www.gaborsamu.com/images/openfoam_job.jpg" />
</figure>

<p>The LSF <em>bjobs</em> and <em>bhist</em> output from the job follows below:</p>

<div class="highlight"><pre><code class="language-plaintext">$ bjobs -l 2613

Job &lt;2613&gt;, Job Name &lt;myOpenFoam_run_motorBike&gt;, User &lt;lsfadmin&gt;, Project &lt;defa
                     ult&gt;, Application &lt;openfoam&gt;, Status &lt;RUN&gt;, Queue &lt;normal&gt;
                     , Command &lt;/mnt/lsf/repository-path/lsfadmin/myOpenFoam_ru
                     n_1722358195200AuWDY/motorBike/bsub.myOpenFoam_run&gt;, Share
                      group charged &lt;/lsfadmin&gt;
Tue Jul 30 16:49:55: Submitted from host &lt;gsamu-hpc-demo-mgmt-1-a844-001&gt;, CWD 
                     &lt;/mnt/lsf/repository-path/lsfadmin/myOpenFoam_run_17223581
                     95200AuWDY/motorBike&gt;, Specified CWD &lt;/mnt/lsf/repository-
                     path/lsfadmin/myOpenFoam_run_1722358195200AuWDY/motorBike&gt;
                     , Output File &lt;/mnt/lsf/repository-path/lsfadmin/myOpenFoa
                     m_run_1722358195200AuWDY/motorBike/output.lsfadmin.txt&gt;, E
                     rror File &lt;/mnt/lsf/repository-path/lsfadmin/myOpenFoam_ru
                     n_1722358195200AuWDY/motorBike/error.lsfadmin.txt&gt;, Notify
                      when job begins/ends, 6 Task(s), Requested Resources &lt;spa
                     n[hosts=1]&gt;;
Tue Jul 30 16:55:23: Started 6 Task(s) on Host(s) &lt;gsamu-hpc-demo-10-241-0-137&gt;
                     &lt;gsamu-hpc-demo-10-241-0-137&gt; &lt;gsamu-hpc-demo-10-241-0-137
                     &gt; &lt;gsamu-hpc-demo-10-241-0-137&gt; &lt;gsamu-hpc-demo-10-241-0-1
                     37&gt; &lt;gsamu-hpc-demo-10-241-0-137&gt;, Allocated 6 Slot(s) on 
                     Host(s) &lt;gsamu-hpc-demo-10-241-0-137&gt; &lt;gsamu-hpc-demo-10-2
                     41-0-137&gt; &lt;gsamu-hpc-demo-10-241-0-137&gt; &lt;gsamu-hpc-demo-10
                     -241-0-137&gt; &lt;gsamu-hpc-demo-10-241-0-137&gt; &lt;gsamu-hpc-demo-
                     10-241-0-137&gt;, Execution Home &lt;/home/lsfadmin&gt;, Execution 
                     CWD &lt;/mnt/lsf/repository-path/lsfadmin/myOpenFoam_run_1722
                     358195200AuWDY/motorBike&gt;;
Tue Jul 30 17:03:33: Resource usage collected.
                     The CPU time used is 1411 seconds.
                     MEM: 928 Mbytes;  SWAP: 0 Mbytes;  NTHREAD: 41
                     PGID: 18426;  PIDs: 18426 18427 18428 20088 
                     PGID: 20374;  PIDs: 20374 20388 20800 21385 
                     PGID: 21389;  PIDs: 21389 
                     PGID: 21390;  PIDs: 21390 
                     PGID: 21391;  PIDs: 21391 
                     PGID: 21392;  PIDs: 21392 
                     PGID: 21393;  PIDs: 21393 
                     PGID: 21394;  PIDs: 21394 


 MEMORY USAGE:
 MAX MEM: 982 Mbytes;  AVG MEM: 422 Mbytes; MEM Efficiency: 0.00%

 CPU USAGE:
 CPU PEAK: 5.89 ;  CPU PEAK DURATION: 63 second(s)
 CPU AVERAGE EFFICIENCY: 42.81% ;  CPU PEAK EFFICIENCY: 98.15%

 SCHEDULING PARAMETERS:
           r15s   r1m  r15m   ut      pg    io   ls    it    tmp    swp    mem
 loadSched   -     -     -     -       -     -    -     -     -      -      -  
 loadStop    -     -     -     -       -     -    -     -     -      -      -  

 RESOURCE REQUIREMENT DETAILS:
 Combined: select[(docker) &amp;&amp; (type == any)] order[r15s:pg] span[hosts=1]
 Effective: select[(docker) &amp;&amp; (type == any)] order[r15s:pg] span[hosts=1] </code></pre></div>

<div class="highlight"><pre><code class="language-plaintext">$ bhist -l 2613

Job &lt;2613&gt;, Job Name &lt;myOpenFoam_run_motorBike&gt;, User &lt;lsfadmin&gt;, Project &lt;defa
                     ult&gt;, Application &lt;openfoam&gt;, Command &lt;/mnt/lsf/repository
                     -path/lsfadmin/myOpenFoam_run_1722358195200AuWDY/motorBike
                     /bsub.myOpenFoam_run&gt;
Tue Jul 30 16:49:55: Submitted from host &lt;gsamu-hpc-demo-mgmt-1-a844-001&gt;, to Q
                     ueue &lt;normal&gt;, CWD &lt;/mnt/lsf/repository-path/lsfadmin/myOp
                     enFoam_run_1722358195200AuWDY/motorBike&gt;, Specified CWD &lt;/
                     mnt/lsf/repository-path/lsfadmin/myOpenFoam_run_1722358195
                     200AuWDY/motorBike&gt;, Output File &lt;/mnt/lsf/repository-path
                     /lsfadmin/myOpenFoam_run_1722358195200AuWDY/motorBike/outp
                     ut.lsfadmin.txt&gt;, Error File &lt;/mnt/lsf/repository-path/lsf
                     admin/myOpenFoam_run_1722358195200AuWDY/motorBike/error.ls
                     fadmin.txt&gt;, Notify when job begins/ends, 6 Task(s), Reque
                     sted Resources &lt;span[hosts=1]&gt;;
Tue Jul 30 16:55:23: Dispatched 6 Task(s) on Host(s) &lt;gsamu-hpc-demo-10-241-0-1
                     37&gt; &lt;gsamu-hpc-demo-10-241-0-137&gt; &lt;gsamu-hpc-demo-10-241-0
                     -137&gt; &lt;gsamu-hpc-demo-10-241-0-137&gt; &lt;gsamu-hpc-demo-10-241
                     -0-137&gt; &lt;gsamu-hpc-demo-10-241-0-137&gt;, Allocated 6 Slot(s)
                      on Host(s) &lt;gsamu-hpc-demo-10-241-0-137&gt; &lt;gsamu-hpc-demo-
                     10-241-0-137&gt; &lt;gsamu-hpc-demo-10-241-0-137&gt; &lt;gsamu-hpc-dem
                     o-10-241-0-137&gt; &lt;gsamu-hpc-demo-10-241-0-137&gt; &lt;gsamu-hpc-d
                     emo-10-241-0-137&gt;, Effective RES_REQ &lt;select[(docker) &amp;&amp; (
                     type == any)] order[r15s:pg] span[hosts=1] &gt;;
Tue Jul 30 16:55:23: Starting (Pid 18426);
Tue Jul 30 16:55:24: Running with execution home &lt;/home/lsfadmin&gt;, Execution CW
                     D &lt;/mnt/lsf/repository-path/lsfadmin/myOpenFoam_run_172235
                     8195200AuWDY/motorBike&gt;, Execution Pid &lt;18426&gt;;
Tue Jul 30 17:04:01: Done successfully. The CPU time used is 1535.1 seconds;
Tue Jul 30 17:04:02: Post job process done successfully;


MEMORY USAGE:
MAX MEM: 982 Mbytes;  AVG MEM: 431 Mbytes; MEM Efficiency: 0.00%

CPU USAGE:
CPU PEAK: 5.92 ;  CPU PEAK DURATION: 63 second(s)
CPU AVERAGE EFFICIENCY: 50.67% ;  CPU PEAK EFFICIENCY: 98.68%

Summary of time in seconds spent in various states by  Tue Jul 30 17:04:02
  PEND     PSUSP    RUN      USUSP    SSUSP    UNKWN    TOTAL
  328      0        518      0        0        0        846         </code></pre></div>

<p><strong>Conclusion</strong></p>

<p>The <em>user_data.sh</em> script of LSF resource connector allows a high degree of customization for cloud compute resources that dynamically join the LSF cluster. We’ve demonstrated how it can be used to tag cloud compute resources with a specific LSF Boolean resource in addition to the ability to install specific packages and do configuration customization. This is a simplified example, but illustrates this point.</p>