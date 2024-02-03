---
author: Derek Weitzel's Blog
author_tag: dweitzel
blog_subtitle: Thoughts from Derek
blog_title: Dereks Web
blog_url: https://derekweitzel.com/
category: dweitzel
date: '2024-01-31 05:00:00'
layout: post
original_url: https://derekweitzel.com/2024/01/31/profiling-xrootd-collector/
slug: profiling-the-xrootd-monitoring-collector
title: Profiling the XRootD Monitoring Collector
---

<p>The <a href="https://github.com/opensciencegrid/xrootd-monitoring-collector">XRootD Monitoring Collector</a> (collector) receives file transfer accounting messages from <a href="https://xrootd.slac.stanford.edu/">XRootD</a> servers.
This transfer information is parsed by the collector and sent to the GRACC accounting database for visualization.
Each transfer will generate multiple messages:</p>


<ol>
  <li>Connection message with client information</li>
  <li>Token information</li>
  <li>File open with file name</li>
  <li>Transfer updates (potentially multiple)</li>
  <li>File close with statistics about bytes read and written</li>
  <li>Disconnection</li>
</ol>

<p>We can see 1000+ messages a second from XRootD servers across the OSG.  But, recently the collector has not been able to keep up.  Below is the traffic of messages to the collector from the OSG’s Message Bus:</p>


<figure class="">
  <img alt="this is a placeholder image" src="https://derekweitzel.com/images/posts/profiling-xrootd-collector/before-optimization-mq.png" /><figcaption>
      Message bus traffic before optimization

    </figcaption></figure>

<p>The graph is from the message bus’s perspective, so publish is incoming to the message bus, and deliver is sending to consumers (the Collector).  We are receiving (Publish) ~1550 messages a second, while the collector is only able to process (Deliver) ~500 messages a second.  1550 messages a second is higher than our average, but we need to be able to process data as fast as it comes.  Messages that are not processed will wait on the queue.  If the queue gets too large (maximum is set to 1 Million messages) then the messages will be deleted, losing valuable transfer accounting data.  At a defecit 1000 messages a second, it would only take ~16 minutes to fill the queue.  It is clear that we missed data for a significant amount of time.</p>


<h2 id="profiling">Profiling</h2>

<p>The first step to optimizing the XRootD Monitoring Collector is to profile the current process.  Profiling is the process of measuring the performance of the collector to identify bottlenecks and areas for improvement.</p>


<p>For profiling, I created a development environment on the <a href="https://nationalresearchplatform.org/">National Research Platform (NRP)</a> to host the collector.  I started a <a href="https://docs.nationalresearchplatform.org/userdocs/jupyter/jupyterhub-service/">jupyter notebook on the NRP</a>, and used VSCode to edit the collector code and a Jupyter notebook to process the data.  I used the <a href="https://docs.python.org/3/library/profile.html">cProfile</a> package built into python to perform the profiling.
I modified the collector to output a profile update every 10 seconds so I could see the progress of the collector.</p>


<p>After profiling, I used <a href="https://jiffyclub.github.io/snakeviz/">snakeviz</a> to visualize the profile.  Below is a visualization of the profile before any optimization.  The largest consumer of processing time was DNS resoluiton, highlighted in the below image in purple.</p>


<figure class="">
  <img alt="this is a placeholder image" src="https://derekweitzel.com/images/posts/profiling-xrootd-collector/before-optimization-profile.png" /><figcaption>
      Snakeviz profile.  Purple is the DNS resolution function

    </figcaption></figure>

<p>The collector uses DNS to resolve the hostnames for all IPs it receives in order to provide a human friendly name for clients and servers.  Significant DNS resolution is expected as the collector is receiving messages from many different hosts.  However, the DNS resolution is taking up a significant amount of time and is a bottleneck for the collector.</p>


<h2 id="improvement">Improvement</h2>

<p>After reviewing the profile, <a href="https://github.com/opensciencegrid/xrootd-monitoring-collector/pull/43">I added a cache to the DNS resolution</a> so that the collecotr only needs to resolve the host once every 24 hours.  When I profiled after making the change, I saw a significant improvement in DNS resolution time.  Below is another visualization of the profile after the DNS caching, purple is the DNS resolution.</p>


<figure class="">
  <img alt="this is a placeholder image" src="https://derekweitzel.com/images/posts/profiling-xrootd-collector/after-optimization-profile.png" /><figcaption>
      Snakeviz profile.  Purple is the DNS resolution function

    </figcaption></figure>

<p>Notice that the DNS resolution is a much smaller portion of the overall running time when compared to the previous profile.</p>


<p>In the following graph, I show the time spent on DNS resolution over time for both before and after the optimization.  I would expect DNS resolution to increase for both, but as you can see, the increase after adding DNS caching is much slower.</p>


<figure class="">
  <img alt="this is a placeholder image" src="https://derekweitzel.com/images/posts/profiling-xrootd-collector/dns-resolution.png" /><figcaption>
      Growth of DNS resolution time

    </figcaption></figure>

<h2 id="production">Production</h2>

<p>When we applied the changes into production, we saw a significant improvement in the collector’s ability to process messages.  Below is the graph of the OSG’s Message Bus after the change:</p>


<figure class="">
  <img alt="this is a placeholder image" src="https://derekweitzel.com/images/posts/profiling-xrootd-collector/edited-production-mq.png" /><figcaption>
      RabbitMQ Message Parsing

    </figcaption></figure>

<p>The incoming messages decreased, but the collector is now able to process messages as fast as they are received.  This is a significant improvement over the previous state.  I suspect that the decrease in incoming messages is due to server load of sending more outgoing messages to the improved collector.  The message bus can slow down the incoming messages under heavier load.</p>


<h2 id="conclusions-and-future-work">Conclusions and Future Work</h2>

<p>Since we implemented the cache for DNS resolution, the collector has been able to keep up with the incoming messages.  This is a significant improvement over the previous state.  Over time, we expect the DNS cache to capture nearly all of the hosts, and the DNS resolution time to decrease even further.</p>


<p>We continue to look for optimizations to the collector.  When looking at the output from the most recent profile, we noticed the collector is spending a significant amount of time in the logging functions.  By default, we have debug logging turned on.  We will look at turning off debug logging in the future.</p>


<p>Additionally, the collector is spending a lot of time polling for messages.  In fact, the message bus is receiving ~1500 messages a second, which is increasing the load on the message bus.  After reading through optimizations for RabbitMQ, it appears that less but larger messages are better for the message bus.  We will look at batching messages in the future.</p>