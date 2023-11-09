---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2023-11-08 02:21:04'
layout: post
original_url: https://www.gaborsamu.com/blog/lsf_rc_ibmcloud_part1/
slug: advanced-lsf-resource-connector-configuration-on-ibm-cloud-part-i
title: Advanced LSF resource connector configuration on IBM Cloud - part I
---

<p><strong>Overview</strong></p>

<p>This is the first in a series of blogs that discusses some advanced
configuration of the IBM LSF resource connector. LSF resource connector enables
LSF clusters to borrow resources from supported resource providers in the
cloud. LSF includes resource connectors for the following resource providers:</p>

<ul>
<li>IBM Cloud</li>
<li>AWS</li>
<li>Google Cloud Platform</li>
<li>Microsoft Azure</li>
<li>Microsoft Azure CycleCloud</li>
<li>Red Hat OpenShift</li>
<li>OpenStack</li>
</ul>
<p>The resource connector plug-ins for LSF are available under an open source
license (the Apache License 2.0) on the public IBM Spectrum Computing github
<a href="https://github.com/IBMSpectrumComputing/cloud-provider-plugins">here</a>.</p>

<p>LSF resource connector works in conjunction with the LSF multicluster
capability to create a flexible and dynamic hybrid HPC cloud. LSF multicluster
enables organizations to have multiple LSF clusters connect with one another
and to define queues which can forward to remote clusters and receive jobs
from remote clusters. Historically LSF multicluster was used by clients who
have multiple, geographically dispersed compute centres and it allowed them to
connect these environments and have work forwarded between them. Naturally
this can also be used to setup an LSF cluster in the cloud and tie it in to
your existing on-premises LSF cluster. According to a recent Hyperion Research
whitepaper, the most widely adopted framework for leveraging HPC resources in
the cloud is in a hybrid environment where a user runs their HPC workloads
both on-premises and in the cloud.<sup id="fnref:1"><a class="footnote-ref" href="https://www.gaborsamu.com/blog/index.xml#fn:1">1</a></sup></p>

<p><strong>Cloud templates</strong></p>

<p>As part of the configuration of the LSF resource connector, it’s necessary to
define templates which are used to specify a specific cloud instance type. In
other words, the template is used to define a set of hosts with common
attributes including memory and number of cores and operating system image.
These templates are used by LSF when requesting instances from a particular
cloud to satisfy the workload demands.</p>

<p>In this post, we’ll take a closer look at configuring multiple LSF resource
connector templates for IBM Cloud as the resource provider. By default, when
multiple templates are configured, LSF will sort the candidate template
servers alphabetically by template name. However, administrators may wish to
sort the templates according to a specific priority. For example, an
organization may want to assign a high priority to a template corresponding to
a less costly instance type. When priorities are specified for templates, LSF
will use high priority templates first.</p>

<p>The environment used in this example was deployed using the IBM Cloud
automation for LSF. Using Terraform/IBM Cloud Schematics, it’s possible to
automatically deploy a fully functioning LSF cluster in about 10 minutes time.
This includes a login node, NFS storage node, LSF manager node(s) and
LSF Application Center. The automation also configures the LSF resource
connector for the instance type specified at deployment time. More detailed
information can be found about deploying LSF on IBM Cloud <a href="https://cloud.ibm.com/docs/ibm-spectrum-lsf?topic=ibm-spectrum-lsf-getting-started-tutorial">here</a>.</p>

<p>The following steps assume that LSF has been deployed on IBM Cloud. Note that
this is a single LSF cluster and not a hybrid cloud that has been configured.
Now, we’ll look at the following configuration examples:</p>

<ul>
<li>Specifying multiple templates for different cloud instance types with different priorities</li>
<li>Update LSF resource connector user script to load required OS packages on compute nodes</li>
</ul>
<p><strong>Multiple templates</strong></p>

<p>The LSF deployment automation on IBM Cloud configures a single template for
the compute VSI type specified at deployment time. In the example below, we
see a single template configured for VSI profile type <em>bx2-4x16</em>. A complete
list of available IBM Cloud VPC VSI types can be found <a href="https://cloud.ibm.com/docs/vpc?topic=vpc-profiles&amp;interface=ui">here</a>.</p>

<p><strong>ibmcloudgen2_templates.json (obfuscated)</strong>
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
                "icgen2host": ["Boolean", "1"]
            },
            "imageId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "subnetId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "vpcId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "vmType": "bx2-4x16",
            "securityGroupIds": ["aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff"],
            "resourceGroupId": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
            "sshkey_id": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "region": "us-east",
            "zone": "us-east-1"
        }
    ]
}</code></pre></div>
</p>

<p><strong>Defining priorities</strong></p>

<p>You’ll note that there is no priority specified for the template. When no
priority is defined, LSF sorts the templates according to template name. Next,
we’ll define a second template in the configuration for the VSI instance type
<em>mx2-16x128</em>. At the same time, we’ll introduce the priority parameter and
specify a priority of 10 (higher) for <em>bx2-4x16</em>, and 5 (lower) for
<em>mx2-16x128</em>. More details regarding the parameters can be found <a href="https://www.ibm.com/docs/en/spectrum-lsf/10.1.0?topic=reference-ibmcloudgen2-templatesjson">here</a>. With this configuration, LSF will favour and use the higher priority
template first, which in this case will be for VSI instance type <em>bx2-4x16</em>.</p>

<p><strong>ibmcloudgen2_templates.json (obfuscated, with priorities configured)</strong></p>

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
                "icgen2host": ["Boolean", "1"] 
            },
            "imageId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "subnetId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "vpcId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "vmType": "bx2-4x16",
            "securityGroupIds": ["aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff"],
            "resourceGroupId": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
            "sshkey_id": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            **"priority": "10",** 
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
                "icgen2host": ["Boolean", "1"]
            },
            "imageId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "subnetId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "vpcId": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            "vmType": "mx2-16x128",
            "userData":"profile=mx2_16x128",
            ""securityGroupIds": ["aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff"],
            "resourceGroupId": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
            "sshkey_id": "aaaa-bbbbbbbb-cccc-dddd-eeee-ffffffffffff",
            **"priority": "5",**
            "region": "us-east",
            "zone": "us-east-1"
        }

    ]
}</code></pre></div>

<p><strong>Modifying compute server options</strong></p>

<p>Next, we’ll submit some example jobs to the LSF cluster and observe how the
LSF resource connector template priority influences the startup of resources
by the LSF resource connector. The example job we wish to run is the OS
supplied <em>stress</em> command. <em>stress</em> is not installed in the default compute
images and can be added as part of the startup of the compute servers via the
LSF resource connector user_data.sh script. This script is used to set
environment variables, and control the startup of LSF on the compute servers.
It can also be used to perform customization of the environment, and pass LSF
resources to the compute servers. The user_data.sh script is located in:
<em>/opt/ibm/lsf/conf/resource_connector/ibmcloudgen2</em>.</p>

<p>Modifying the <em>user_data.sh</em> script, I’ve inserted the line to install the OS
<em>stress</em> package before the LSF daemons startup.</p>

<p><strong>user_data.sh</strong></p>

<div class="highlight"><pre><code class="language-plaintext">...
...
**# Install stress utility**
**dnf install stress -y**

cat $LSF_CONF_FILE  &gt;&gt; $logfile

sleep 5
lsf_daemons start &amp;
sleep 5
lsf_daemons status &gt;&gt; $logfile
echo END `date '+%Y-%m-%d %H:%M:%S'` &gt;&gt; $logfile
# Allow login as lsfadmin
nfs_mount_dir="data"
mkdir -p /home/lsfadmin/.ssh
cp /mnt/data/ssh/authorized_keys /home/lsfadmin/.ssh/authorized_keys
cat /mnt/data/ssh/id_rsa.pub &gt;&gt; /root/.ssh/authorized_keys
chmod 600 /home/lsfadmin/.ssh/authorized_keys
chmod 700 /home/lsfadmin/.ssh
chown -R lsfadmin:lsfadmin /home/lsfadmin/.ssh
echo "MTU=9000" &gt;&gt; "/etc/sysconfig/network-scripts/ifcfg-eth0"
systemctl restart NetworkManager
...
...</code></pre></div>

<p>With the updates made to the user_data.sh script, we’re now ready to submit
jobs to LSF. We submit two <em>stress</em> jobs as follows:</p>

<div class="highlight"><pre><code class="language-plaintext">[lsfadmin@icgen2host-10-241-0-37 ~]$ bsub -q normal /usr/bin/stress --cpu 1 --vm-bytes 8192MB --timeout 60s
Job &lt;2836&gt; is submitted to queue &lt;normal&gt;.
[lsfadmin@icgen2host-10-241-0-37 ~]$ bsub -q normal /usr/bin/stress --cpu 1 --vm-bytes 8192MB --timeout 60s
Job &lt;2837&gt; is submitted to queue &lt;normal&gt;.</code></pre></div>

<p>After a few moments, the LSF resource connector automatically starts up a
single compute server on the IBM Cloud with hostname <em>icgen2host-10-241-0-42</em>
to satisfy the pending workload requirements. Note that the hostname prefix
icgen2host stands for IBM Cloud Generation 2 VPC. The numeric portion of the
hostname represents the IP address of the server that was automatically
started by the LSF resource connector. Therefore, this hostname may differ
in your environment. Compute server <em>icgen2host-10-241-0-42</em> is equipped with
4 cores and 16 GB RAM, matching the higher priority template <em>bx2-4x16</em>.</p>

<div class="highlight"><pre><code class="language-plaintext">[lsfadmin@icgen2host-10-241-0-37 ~]$ lshosts -w
HOST_NAME                       type       model  cpuf ncpus maxmem maxswp server RESOURCES
icgen2host-10-241-0-37        X86_64    Intel_E5  12.5     4  15.4G      -    Yes (mg)
icgen2host-10-241-0-42        X86_64    Intel_E5  12.5     4  15.5G      -    Dyn (icgen2host)


[lsfadmin@icgen2host-10-241-0-37 ~]$ bhosts -rc -w
HOST_NAME          STATUS          JL/U    MAX  NJOBS    RUN  SSUSP  USUSP    RSV RC_STATUS             PROV_STATUS           UPDATED_AT             INSTANCE_ID               
icgen2host-10-241-0-37 closed_Full     -      0      0      0      0      0      0           -                     -                     -                      -               
icgen2host-10-241-0-42 ok              -      4      2      2      0      0      0 Allocated             running               2023-11-06T23:02:26UTC 0757_4f5295bd-a265-4fda-840c-6f89e326ca1f </code></pre></div>

<p>As there is no other work that has been submitted to the LSF cluster, once the
<em>stress</em> jobs have completed, the LSF resource connector will automatically
shut down the compute servers according to the <strong>LSB_RC_EXTERNAL_HOST_IDLE_TIME</strong>
parameter in <em>lsf.conf</em>. This defines the time interval after which the LSF
resource connector will relinquish the cloud instances if no jobs are running.</p>

<p><strong>Conclusion</strong></p>

<p>We’ve just scratched the surface in terms of the configuration possibilities
with LSF resource connector. In the next article we’ll look at how LSF
resources can be assigned to servers which are dynamically started by the LSF
resource connector, as well as configuring Docker to point to a local
repository.</p>

<section class="footnotes">
<hr />
<ol>
<li id="fn:1">
<p><a href="https://www.ibm.com/downloads/cas/RGKYOOKB">The Evolution of HPC Includes Strong Use of Hybrid Cloud</a>&#160;<a class="footnote-backref" href="https://www.gaborsamu.com/blog/index.xml#fnref:1">&#x21a9;&#xfe0e;</a></p>

</li>
</ol>
</section>