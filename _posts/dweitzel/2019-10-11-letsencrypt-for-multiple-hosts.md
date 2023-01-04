---
author: Derek Weitzel's Blog
author_tag: dweitzel
blog_subtitle: Thoughts from Derek
blog_title: Dereks Web
blog_url: https://derekweitzel.com/
category: dweitzel
date: '2019-10-11 20:38:14'
layout: post
original_url: https://derekweitzel.com/2019/10/11/letsencrypt-for-multiple-hosts/
slug: letsencrypt-for-multiple-hosts
title: LetsEncrypt for Multiple Hosts
---

<p>Using <a href="https://letsencrypt.org/">LetsEncrypt</a> for certificate creation and management has made secure communications much easier.  Instead of contacting the IT department of your university to request a certificate, you can skip the middle man and generate your own certificate which it trusted around the world.</p>


<p>A common use case of certificates is to secure data transfers.  Data transfers that use the GridFTP, XRootD, or HTTPS transfer protocols can load balance between multiple servers to increase throughput.  <a href="https://www.keepalived.org/">keepalived</a> is used to load balance between multiple transfer servers.  The certificate provided to the clients need to have the virtual host address of the load balancer, as well as the hostname of each of the worker nodes.</p>


<ol>
  <li>Create a shared directory between the data transfer nodes</li>
  <li>Install httpd on each of the data transfer nodes</li>
  <li>Configure httpd to use the shared directory as the “webroot”</li>
  <li>Configure <code class="language-plaintext highlighter-rouge">keepalived</code> to use virtualize port 80 to at least 1 of your data transfer nodes.</li>
  <li>Run certbot with the webroot option, as well as the multiple hostnames of the data transfer nodes.</li>
</ol>

<p>Create a NFS share that each of the data transfer nodes can read.  The steps in creating a NFS shared directory is outside the scope of this guide.  In this guide, the shared directory will be referred as <code class="language-plaintext highlighter-rouge">/mnt/nfsshare</code> . Next, install httpd on each of the data transfer nodes:</p>


<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>root@host $ yum install httpd
</code></pre></div>
</div>


<p>Create a webroot directory within the shared directory on one of the nodes:</p>


<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>root@host $ mkdir /mnt/nfsshare/webroot
</code></pre></div>
</div>


<p>Configure httpd to export the same webroot on each of the data transfer nodes:</p>


<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>&lt;VirtualHost *:80&gt;
    DocumentRoot "/mnt/nfsshare/webroot"
    &lt;Directory "/mnt/nfsshare/webroot"&gt;
        Require all granted
    &lt;/Directory&gt;
&lt;/VirtualHost&gt;
</code></pre></div>
</div>


<p>Configure <code class="language-plaintext highlighter-rouge">keepalived</code> to virtualize port 80 to at least one of your data transfer nodes.
Add to your configuration:</p>


<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>virtual_server &lt;VIRTUAL-IP-ADDRESS&gt; 80 {
    delay_loop 10
    lb_algo wlc
    lb_kind DR
    protocol tcp

    real_server &lt;GRIDFTP-SERVER-#1-IP ADDRESS&gt; {
        TCP_CHECK {
            connect_timeout 3
            connect_port 80
        }
    }
}
</code></pre></div>
</div>


<p>Run <code class="language-plaintext highlighter-rouge">certbot</code> with the webroot options on only 1 of the data nodes.  The first domain in the command line should be the virtual hostname:</p>


<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>root@host $ certbot certonly -w /mnt/nfsshare/webroot -d &lt;VIRTUAL_HOSTNAME&gt; -d &lt;DATANODE_1&gt; -d &lt;DATANODE_N&gt;...
</code></pre></div>
</div>