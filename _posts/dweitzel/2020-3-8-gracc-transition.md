---
author: Derek Weitzel's Blog
author_tag: dweitzel
blog_subtitle: Thoughts from Derek
blog_title: Dereks Web
blog_url: https://derekweitzel.com/
category: dweitzel
date: '2020-03-08 06:00:00'
layout: post
original_url: https://derekweitzel.com/2020/03/08/gracc-transition/
slug: gracc-transition-visualization
title: GRACC Transition Visualization
---

<p>The OSG is in the progress of transitioning from an older ElasticSearch (ES) cluster to a new version.  Part of this process is reindexing (copying) data from the old to the new.  Unfortunately, it’s not easy to capture a status of this transition.  For this, I have created the <a href="https://gracc-transition.herokuapp.com/">GRACC Transition page</a>.</p>


<p>The goal is to transition when both the old and new ES have the same data.  A simple measure of this is if they share the same number of documents in all of the indexes.</p>


<p>Source for this app is available on github: <a href="https://github.com/djw8605/gracc-transition">GRACC Transition</a></p>


<h2 id="data-collection">Data Collection</h2>

<p>Data collection is performed by a probe on each the new and old ElasticSearch clusters.  Upload is performed with a POST to the gracc transition website.  Authorization is performed with a shared random token between the probe and the website.</p>


<p>The probe is very simple.  It queries ES for all indexes, as well as the number of documents and data size inside the index.</p>


<p>There are also many indexes that the OSG is not transitioning to the new ES.  In order to ignore these indexes, a set of regular expressions is used to remove the indexes from consideration.  Those regular expressions are:</p>


<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>/^osg.*/,           // Start with osg.*
/^ps_.*/,           // Start with ps_*
/^shrink\-ps_.*/,   // Start with shrink-ps_*
/^glidein.*/,       // Start with glidein*
/^\..*/,            // Start with .
/^ps\-itb.*/        // Start with ps-itb*
</code></pre></div>
</div>


<h2 id="the-website">The Website</h2>

<p><img alt="GRACC Transition Website" src="https://derekweitzel.com/images/posts/gracc-transition/gracc-transition-website.png" /></p>


<p>The gracc transition app is hosted on the <a href="https://www.heroku.com/">Heroku</a>.  I choose Heroku because it provides a simple hosting platform with a database for free.</p>


<p>The website pushes alot of the data processing to the client.  The data is stored in the database as JSON and is sent to the client without any transformation.  The client pulls the data from the website for both the new and old ES and begins to process the data within javascript.</p>


<p>The website breaks the statistics into three visualizations:</p>


<ol>
  <li><strong>Progress Bars</strong>: Comparing the total documents and total data size of the old and new.  The progress is defined as new / old.  The bars provide a very good visualization of the progress of the transition as they need to reach 100% before we are able to fully transition.</li>
  <li><strong>Summary Statistics</strong>: The summary statistics show the raw number of either missing or mismatched indexes.  If an index is in the old ES but is not in the new ES, it is counted as <strong>missing</strong>.  If the index is a different size in the old vs. the new, it is counted as <strong>mismatched</strong>.</li>
  <li><strong>Table of Indices</strong>: Finally, a table of indices is shown with the number of documents that are missing, or simply <strong>Missing</strong> if the index is missing in the new ES.</li>
</ol>

<p>In addition to the table, I also provide a button to download the list of indexes that are missing or mismatched.  This can be useful for an administrator to make sure it matches what they expect or to process with elasticsearch.</p>


<h2 id="improvements-and-future">Improvements and Future</h2>

<p>In the future, I would like to generate a weekly or even daily email to show the progress of the transition.  This would give provide a constant reminder of the state of the transition.</p>