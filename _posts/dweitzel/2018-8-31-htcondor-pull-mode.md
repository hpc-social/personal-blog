---
author: Derek Weitzel's Blog
author_tag: dweitzel
blog_subtitle: Thoughts from Derek
blog_title: Dereks Web
blog_url: https://derekweitzel.com/
category: dweitzel
date: '2018-08-31 18:28:42'
layout: post
original_url: https://derekweitzel.com/2018/08/31/htcondor-pull-mode/
slug: htcondor-pull-mode
title: HTCondor Pull Mode
---

<p>For a recent project to utilize HPC clusters for HTC workflows, I had to add the ability to transfer the input and output sandboxes to and from HTCondor.  HTCondor already has the ability to spool input files to a SchedD, and pull the output sandbox. These functions are intended to stage jobs to an HTCondor pool.  But, HTCondor did not have the ability to pull jobs from an HTCondor pool.</p>


<p>The anticipated steps for a job pulled from an HTCondor pool:</p>


<ol>
  <li>Download the <strong>input</strong> sandbox</li>
  <li>Submit the job to the local scheduler</li>
  <li>Watch the job status of the job</li>
  <li>Once completed, transfer the <strong>output</strong> sandbox to the origin SchedD</li>
</ol>

<p>The sandboxes are:</p>


<ul>
  <li><strong>Input</strong>:
    <ul>
      <li>Input files</li>
      <li>Executable</li>
      <li>Credentials</li>
    </ul>
  </li>
  <li><strong>Output</strong>:
    <ul>
      <li>Stdout / Stderr from job</li>
      <li>Output files or any files that may have changed while the job ran</li>
    </ul>
  </li>
</ul>

<h2 id="api-additions">API Additions</h2>

<p>In order to transfer the input sandbox and output sandbox, 2 new commands where added to the SchedD, as well as a new client function and python bindings to use them.</p>


<p>The function for transferring input files is:</p>


<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>transferInputSandbox(constraint, destination)
</code></pre></div>
</div>


<p><code class="language-plaintext highlighter-rouge">jobs</code> is a HTCondor constraint selecting the jobs whose input files should be transferred.  <code class="language-plaintext highlighter-rouge">destination</code> is a directory to put the sandboxes.  The sandboxes will be placed in directories named <code class="language-plaintext highlighter-rouge">destination/&lt;ClusterId&gt;/&lt;ProcId&gt;/</code>.</p>


<p>For transferring output files, the function is:</p>


<div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code>transferOutputSandbox( jobs )
</code></pre></div>
</div>


<p>Where <code class="language-plaintext highlighter-rouge">jobs</code> is a list of tuples.  The structure of the tuple is <code class="language-plaintext highlighter-rouge">( classad, sandboxdir )</code>.  <code class="language-plaintext highlighter-rouge">classad</code> is the full classad of the original job, and <code class="language-plaintext highlighter-rouge">sandboxdir</code> is the location of the output sandbox to send.</p>


<h2 id="current-status">Current Status</h2>

<p>I have created a <a href="https://github.com/djw8605/htcondor-pull">repo</a> for an example that uses these functions in order to pull a job from a remote SchedD.</p>


<p>Also, my changes to <a href="https://github.com/djw8605/htcondor/tree/add_sandbox_transfers">HTCondor</a> are in my repo, and I have begun the discussion about merging in my changes.</p>