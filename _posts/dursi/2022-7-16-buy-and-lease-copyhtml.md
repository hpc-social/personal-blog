---
author: Jonathan Dursi's Blog
author_tag: dursi
blog_subtitle: R&amp;D computing at scale
blog_title: Jonathan Dursi
blog_url: http://www.dursi.ca
category: dursi
date: '2022-07-16 00:00:00'
layout: post
original_url: http://www.dursi.ca/post/buy-and-lease-copy.html
slug: buy-and-lease-not-cloud-vs-on-prem
title: Buy and Lease, not Cloud vs On-Prem
---

<p>(Note: This post is adapted from <a href="https://www.researchcomputingteams.org/newsletter_issues/0130">#130</a> of the <a href="https://www.researchcomputingteams.org">Research Computing Teams Newsletter</a>)</p>


<p>I’d like us to move past the “cloud-vs-on-prem” debate.  Right now, AWS or GCP will deliver their cloud hardware into your data centres to run there, if you want.  Various commercial software can be subscribed to to manage infrastructure control.  Hardware can be leased, bought, sold back.  If your data centre is a co-lo, so the premises aren’t yours, is it really on premises?  And…</p>


<p>There’s a whole spectrum of options available today, and our community is still debating “on-prem vs cloud” like it’s 2012.  It would do us and our researchers well to have more sophisticated discussions.  The question isn’t “on-prem vs cloud”, it’s what should be bought outright and what should be leased for a given workload mix.</p>


<p>Here are some real options for moderately-sized teams charged with delivering the capabilities of a system to users right now:</p>


<ul>
  <li>Buy the hardware outright, install all open-source software,</li>
  <li>Buy the hardware outright, install a lot of “rented” commercial software (or commercial support of open source software)</li>
  <li>Buy some hardware outright and install something like Azure Stack, renting a lot of the infrastructure software and control plane</li>
  <li>Lease the hardware, with any of the software options above</li>
  <li>Host any of the hardware options above in a datacenter you own, “renting” the data centre operations staff’s time (cooling, power, internet, physical security..)</li>
  <li>Any of the hardware above in a leased space where you control and have to manage the utilities, renting the same staff time</li>
  <li>Any of the hardware above in an an institutional or commercially-managed space where you are now “renting” the space and their data centre operations staff</li>
  <li>Rent medium-term reserved hardware at a cloud provider, combining data centre ops, some systems ops, and the hardware for moderate periods of time</li>
  <li>Mix of the above plus short-term “renting” of on-demand or shorter-term contract nodes/services</li>
  <li>All short-term</li>
  <li>“Hybrid”, which just means a mix of any two or more of any of the combinations above</li>
</ul>

<p>It’s not “A vs B” any more, and it hasn’t been for ages.  We have a widely diverse research community with its huge range of use cases to support, and we need to think in more sophisticated ways than obsolescent binaries.</p>


<p>(And <strong>here’s</strong> a controversial take:  You know who in our institutions are <em>really, really</em> smart about leasing versus buying for big capital needs?  Who have taken, like, whole postsecondary courses on the topic?  The folks in the finance department).</p>