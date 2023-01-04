---
author: Derek Weitzel's Blog
author_tag: dweitzel
blog_subtitle: Thoughts from Derek
blog_title: Dereks Web
blog_url: https://derekweitzel.com/
category: dweitzel
date: '2018-09-26 06:00:00'
layout: post
original_url: https://derekweitzel.com/2018/09/26/stashcache-by-the-numbers/
slug: stashcache-by-the-numbers
title: StashCache By The Numbers
---

<p>The StashCache federation is comprised of 3 components: Origins, Caches, and Clients.  There are additional components that increase the usability of StashCache which I will also mention in this post.</p>


<figure class="">
  <img alt="Diagram of StashCache Infrastructure" src="https://derekweitzel.com/images/posts/StashCache-By-Numbers/StashCache-Diagram.png" /><figcaption>
      Diagram of the StashCache Federation

    </figcaption></figure>

<figure class="">
  <img alt="Cumulative Usage of StashCache" src="https://derekweitzel.com/images/posts/StashCache-By-Numbers/StashCache-Cumulative.png" /><figcaption>
      Cumulative Usage of StashCache over the last 90 days

    </figcaption></figure>

<h2 id="origins">Origins</h2>

<p>A StashCache Origin is the authoritative source of data.  The origin receives data location requests from the central redirectors.  These requests take the form of “Do you have the file X”, to which the origin will respond “Yes” or “No”.  The redirector then returns a list of origins that claim to have the requested file to the client.</p>


<p>An Origin is a simple XRootD server, exporting a directory or set of directories for access.</p>


<table>
  <thead>
    <tr>
      <th>Origin</th>
      <th>Base Directory</th>
      <th>Data Read</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>LIGO Open Data</td>
      <td>/gwdata</td>
      <td>926TB</td>
    </tr>
    <tr>
      <td>OSG Connect</td>
      <td>/user</td>
      <td>246TB</td>
    </tr>
    <tr>
      <td>FNAL</td>
      <td>/pnfs</td>
      <td>166TB</td>
    </tr>
    <tr>
      <td>OSG Connect</td>
      <td>/project</td>
      <td>63TB</td>
    </tr>
  </tbody>
</table>

<p>A list of Origins and their base directories.</p>


<h2 id="clients">Clients</h2>

<p>The clients interact with the StashCache federation on the user’s behalf.  They are responsible for choosing the “best” cache.  The available clients are <a href="https://cernvm.cern.ch/portal/filesystem">CVMFS</a> and <a href="https://github.com/opensciencegrid/StashCache">StashCP</a>.</p>


<figure class="half ">
  
    
      <a href="https://derekweitzel.com/posts/StashCache-By-Numbers/StashCache-CVMFS.png" title="Client Usage By Tool">
          <img alt="Client Usage By Tool" src="https://derekweitzel.com/posts/StashCache-By-Numbers/StashCache-CVMFS.png" />
      </a>
    
  
    
      <a href="https://derekweitzel.com/posts/StashCache-By-Numbers/StashCP-Usage.png" title="StashCP Usage">
          <img alt="StashCP Usage" src="https://derekweitzel.com/posts/StashCache-By-Numbers/StashCP-Usage.png" />
      </a>
    
  
  
    <figcaption>StashCache Client Usage
</figcaption>
  
</figure>

<p>In the pictures above, you can see that most users of StashCache use CVMFS to access the federation.  GeoIP is used by all clients in determining the “best” cache.  GeoIP location services are provided by the CVMFS infrastructure in the U.S.  The geographically nearest cache is used.</p>


<p>The GeoIP service runs on multiple CVMFS Stratum 1s and other servers.  The request to the GeoIP service includes all of the cache hostnames.  The GeoIP service takes the requesting IP address and attempts to locate the requester.  After determining the location of all of the caches, the service returns an ordered list of nearest caches.</p>


<p>The GeoIP service uses the <a href="https://www.maxmind.com/">MaxMind database</a> to determine locations by IP address.</p>


<h3 id="cvmfs">CVMFS</h3>

<p>Most (if not all) origins on are indexed in an <code class="language-plaintext highlighter-rouge">*.osgstorage.org</code> repo.  For example, the OSG Connect origin is indexed in the <code class="language-plaintext highlighter-rouge">stash.osgstorage.org</code> repo.  It uses a special feature of CVMFS where the namespace and data are separated.  The file metadata such as file permissions, directory structure, and checksums are stored within CVMFS.  The file contents are not within CVMFS.</p>


<p>When accessing a file, CVMFS will use the directory structure to form an HTTP request to an external data server.  CVMFS uses GeoIP to determine the nearest cache.</p>


<p>The indexer may also configure a repo to be “authenticated”.  A whitelist of certificate DN’s is stored within the repo metadata and distributed to each client.  The CVMFS client will pull the certificate from the user’s environment.  If the certificate DN matches a DN in the whitelist, it uses the certificate to authenticate with an authenticated cache.</p>


<h3 id="stashcp">StashCP</h3>

<p>StashCP works in the order:</p>


<ol>
  <li>Check if the requested file is available from CVMFS.  If it is, copy the file from CVMFS.</li>
  <li>Determine the nearest cache by sending cache hostnames to the GeoIP service.</li>
  <li>After determining the nearest cache, run the <code class="language-plaintext highlighter-rouge">xrdcp</code> command to copy the data from the nearest cache.</li>
</ol>

<h2 id="caches">Caches</h2>

<figure class="">
  <img alt="Cache Locations" src="https://derekweitzel.com/images/posts/StashCache-By-Numbers/CacheLocations.png" /><figcaption>
      Cache Locations in the U.S.

    </figcaption></figure>

<p>The cache is half XRootD cache and half XRootd client.  When a cache receives a data request from a client, it searches it’s own cache directory for the files.  If the file is not in the cache, it uses the built-in client to retrieve the file from one of the origins.  The cache will request the data location from the central redirector which in turn, asks the origins for the file location.</p>


<p>The cache listens on port 1094 to regular XRootD protocol, and port 8000 for HTTP.</p>


<h3 id="authenticated-caches">Authenticated Caches</h3>

<p>Authenticated caches use GSI certificates to authenticate access to files within the cache.  The client will authenticate with the cache using the client’s certificate.  If the file is not in the cache, the cache will use it’s own certificate to authenticate with the origin to download the file.</p>


<p>Authenticated caches use port 8443 for HTTPS.</p>