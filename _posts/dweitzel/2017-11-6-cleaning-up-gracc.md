---
author: Derek Weitzel's Blog
author_tag: dweitzel
blog_subtitle: Thoughts from Derek
blog_title: Dereks Web
blog_url: https://derekweitzel.com/
category: dweitzel
date: '2017-11-06 19:09:23'
layout: post
original_url: https://derekweitzel.com/2017/11/06/cleaning-up-gracc/
slug: cleaning-up-gracc
title: Cleaning Up GRACC
---

<p>The <a href="https://opensciencegrid.github.io/gracc/">GRid ACcounting Collector</a> (GRACC) is the OSG’s new version of accounting software, replacing Gratia.  It has been running in production since March 2017.  Last week, on Friday November 3rd, we held a GRACC Focus Day.  Our goal was to clean up data that is presented in GRACC.  My changes where:</p>


<ul>
  <li>Update the GRACC-Collector to version <a href="https://github.com/opensciencegrid/gracc-collector/tree/v1.1.8">1.1.8</a>.  The primary change in this release is setting the messages sent to RabbitMQ to be “persistent”.  The persistent messages are then saved to disk in order to survive a RabbitMQ reboot.</li>
  <li>Use case-insenstive comparisons to determine the <a href="https://oim.grid.iu.edu/oim/home">Open Science Grid Information Management system</a> (OIM) information.  This was an issue with GPGrid (Fermilab), which was registered as <strong>GPGRID</strong>.</li>
  <li>Set the <code class="language-plaintext highlighter-rouge">OIM_Site</code> equal to the <code class="language-plaintext highlighter-rouge">Host_description</code> attribute if the OIM logic is unable to determine the registered OIM site.  This is especially useful for the LIGO collaboration, which uses sites in Europe that are not registered in OIM.  Now, instead of a lot of Unknown sites listed on the LIGO site listing, it shows the somewhat reported site name of where the job ran.</li>
</ul>

<figure class="">
  <img alt="GRACC Projects Page" src="https://derekweitzel.com/images/posts/GRACC-Cleanup/GRACC_Projects_Ligo.png" /><figcaption>
      GRACC Projects Page for LIGO

    </figcaption></figure>

<h2 id="regular-expression-corrections"><a id="regex"></a>Regular Expression Corrections</h2>

<p>One of the common problems we have in GRACC is poor data coming from the various probes installed at hundreds of sites.  We don’t control the data coming into GRACC, so occasionally we must make corrections to the data for clarity or correctness.  One of these corrections is misreporting the “site” that the jobs ran on.</p>


<p>In many instances, the probe is unable to determine the site and simply lists the hostname of the worker node where the job ran.  This can cause the cardinality of sites listed in GRACC to increase dramatically as we get new hostnames inserted into the sites listing.  If the hostnames are predictable, a regular expression matching algorithm can match a worker node hostname to a proper site name.</p>


<p>The largest change for GRACC was the regular expression corrections.  With this new feature, GRACC administrators can set corrections to match on attributes using regular expression patterns.  For example, consider the following correction configuration.</p>


<div class="language-toml highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nn">[[Corrections]]</span>
<span class="py">index</span> <span class="p">=</span> <span class="s">'gracc.corrections'</span>
<span class="py">doc_type</span> <span class="p">=</span> <span class="s">'host_description_regex'</span>
<span class="py">match_fields</span> <span class="p">=</span> <span class="nn">['Host_description']</span>
<span class="py">source_field</span> <span class="p">=</span> <span class="s">'Corrected_OIM_Site'</span>
<span class="py">dest_field</span> <span class="p">=</span> <span class="s">'OIM_Site'</span>
<span class="py">regex</span> <span class="p">=</span> <span class="kc">true</span>
</code></pre></div>
</div>


<p>This configuration means:</p>


<blockquote>
  <p>Match the <code class="language-plaintext highlighter-rouge">Host_description</code> field in the incoming job record with the regular expression <code class="language-plaintext highlighter-rouge">Host_description</code> field in the corrections table.  If they are a match, take the value in the <code class="language-plaintext highlighter-rouge">Corrected_OIM_Site</code> field in the corrections table and place it into the <code class="language-plaintext highlighter-rouge">OIM_Site</code> field in the job record.</p>

</blockquote>

<p>And the correction document would look like:</p>


<div class="language-json highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">{</span><span class="w">
  </span><span class="nl">"_index"</span><span class="p">:</span><span class="w"> </span><span class="s2">"gracc.corrections-0"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"_type"</span><span class="p">:</span><span class="w"> </span><span class="s2">"host_description_regex"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"_id"</span><span class="p">:</span><span class="w"> </span><span class="s2">"asldkfj;alksjdf"</span><span class="p">,</span><span class="w">
  </span><span class="nl">"_score"</span><span class="p">:</span><span class="w"> </span><span class="mi">1</span><span class="p">,</span><span class="w">
  </span><span class="nl">"_source"</span><span class="p">:</span><span class="w"> </span><span class="p">{</span><span class="w">
    </span><span class="nl">"Host_description"</span><span class="p">:</span><span class="w"> </span><span class="s2">".*</span><span class="se">\.</span><span class="s2">bridges</span><span class="se">\.</span><span class="s2">psc</span><span class="se">\.</span><span class="s2">edu"</span><span class="p">,</span><span class="w">
    </span><span class="nl">"Corrected_OIM_Site"</span><span class="p">:</span><span class="w"> </span><span class="s2">"PSC Bridges"</span><span class="p">,</span><span class="w">
  </span><span class="p">}</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre></div>
</div>

<p>The regular expression is in the <code class="language-plaintext highlighter-rouge">Host_description</code> FIELD.</p>


<p>So, if the incoming job record is similar to :</p>


<div class="language-json highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">{</span><span class="w">
</span><span class="err">...</span><span class="w">
</span><span class="nl">"Host_description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"l006.pvt.bridges.psc.edu"</span><span class="w">
</span><span class="err">...</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre></div>
</div>


<p>Then the correction would modify or create values such that the final record would approximate:</p>


<div class="language-json highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="p">{</span><span class="w">
</span><span class="err">...</span><span class="w">
</span><span class="nl">"Host_description"</span><span class="p">:</span><span class="w"> </span><span class="s2">"l006.pvt.bridges.psc.edu"</span><span class="p">,</span><span class="w">
</span><span class="nl">"OIM_Site"</span><span class="p">:</span><span class="w"> </span><span class="s2">"PSC Bridges"</span><span class="p">,</span><span class="w">
</span><span class="nl">"RawOIM_Site"</span><span class="p">:</span><span class="w"> </span><span class="s2">""</span><span class="w">
</span><span class="err">...</span><span class="w">
</span><span class="p">}</span><span class="w">
</span></code></pre></div>
</div>


<p>Note that the <code class="language-plaintext highlighter-rouge">Host_description</code> field stays the same.  We must keep it the same because it is used in record duplicate detection.  If we modified the field and resummarized previous records, then it would cause multiple records to represent the same job.</p>