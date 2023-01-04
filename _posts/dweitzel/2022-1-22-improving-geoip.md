---
author: Derek Weitzel's Blog
author_tag: dweitzel
blog_subtitle: Thoughts from Derek
blog_title: Dereks Web
blog_url: https://derekweitzel.com/
category: dweitzel
date: '2022-01-22 05:00:00'
layout: post
original_url: https://derekweitzel.com/2022/01/22/improving-geoip/
slug: improving-the-open-science-data-federation-s-cache-selection
title: Improving the Open Science Data Federation’s Cache Selection
---

<p>Optimizing data transfers requires tuning many parameters.  High latency between the client and a server can decrease data transfer throughput. The Open Science Data Federation (OSDF) attempts to optimize the latency between a client and cache by using GeoIP to locate the nearest cache to the client.  But, using GeoIP alone has many flaws.  In this post, we utilize <a href="https://workers.cloudflare.com/">Cloudflare Workers</a> to provide GeoIP information during cache selection.  During the evaluation, we found that location accuracy grew from <strong>86%</strong> accurate with the original GeoIP service to <strong>95%</strong> accurate with Cloudflare Workers.</p>


<figure class="">
  <img alt="Map of U.S. OSDF" src="https://derekweitzel.com/images/posts/CloudflareWorkers/CacheMap.png" /><figcaption>
      Map of OSDF locations

    </figcaption></figure>

<p>GeoIP has many flaws, first, the nearest physical cache may not be the nearest in the network topology.  Determining the nearest cache in the network would require probing the network topology between the client and every cache, a intensive task to perform for each client startup, and may be impossible with some network configurations, such as blocked network protocols.</p>


<p>Second, the GeoIP database is not perfect.  It does not have every IP address, and the addresses may not have accurate location information.  When GeoIP is unable to determine a location, it will default to “guessing” the location is a lake in Kansas (<a href="https://arstechnica.com/tech-policy/2016/08/kansas-couple-sues-ip-mapping-firm-for-turning-their-life-into-a-digital-hell/">a well known issue</a>).</p>


<p>Following a review of the Open Science Data Federation (OSDF), we found that we could improve effeciency by improving the geo locating of clients.  In the review, several sites where detected to not be using the nearest cache.</p>


<h2 id="implementation">Implementation</h2>

<p>StashCP queries the <a href="https://cernvm.cern.ch/fs/">CVMFS</a> geo location service which relies on the <a href="https://www.maxmind.com/en/home">MaxMind GeoIP database</a>.</p>


<p><a href="https://workers.cloudflare.com/">Cloudflare Workers</a> are designed to run at Cloudflare’s many colocation facilities near the client.  Cloudflare directs a client’s request to a nearby data center using DNS.  Each request is annotaed with an approximate location of the client, as well as the colocation center that received the request.  Cloudflare uses a GeoIP database much like MaxMind, but it also falls back to the colocation site that the request was serviced.</p>


<p>I wrote a Cloudflare worker, <a href="https://github.com/djw8605/cache-locator"><code class="language-plaintext highlighter-rouge">cache-locator</code></a>, which calculates the nearest cache to the client.  It uses the GeoIP location of the client to calculate the ordered list of nearest caches.  If the GeoIP fails for a location, the incoming request to the worker will not be annotated with the location but will include the <code class="language-plaintext highlighter-rouge">IATA</code> airport code of the colocation center that received the client request.  We then return the ordered list of nearest caches to the airport.</p>


<p>We imported a <a href="https://www.partow.net/miscellaneous/airportdatabase/">database of airport codes</a> to locations that is pubically available.  The database is stored in the <a href="https://developers.cloudflare.com/workers/learning/how-kv-works">Cloudflare Key-Value</a>, keyed by the <code class="language-plaintext highlighter-rouge">IATA</code> code of the airport.</p>


<h2 id="evaluation">Evaluation</h2>

<p>To evaluate the location, I submitted test jobs to each site available in the OSG OSPool, 43 different sites at the time of evaluation.  The test jobs:</p>


<ol>
  <li>
    <p>Run the existing <code class="language-plaintext highlighter-rouge">stashcp</code> to retrieve the closest cache.</p>


    <div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code> stashcp --closest
</code></pre></div>
    </div>

  </li>
  <li>
    <p>Run a custom <a href="https://github.com/djw8605/closest-cache-cloudflare">closest script</a> that will query the Cloudflare worker for the nearest caches and print out the cache.</p>

  </li>
</ol>

<p>After the jobs completed, I compiled the caches decisions to a <a href="https://docs.google.com/spreadsheets/d/1mo1FHYW2vpCyhSeCCd_bwP21rFFzqedv0dZ0z8EY4gg/edit?usp=sharing">spreadsheet</a> and manually evaluated each cache selection decision.  The site names in the spreadsheet are the somewhat arbitrary internal names given to sites.</p>


<p>In the spreadsheet, you can see that the correct cache was choosen <strong>86%</strong> of the time with the old GeoIP service, and <strong>95%</strong> of the time with Cloudflare workers.</p>


<h3 id="notes-during-the-evaluation">Notes during the Evaluation</h3>

<p>Cloudflare was determined to be incorrect at two sites, the first being <code class="language-plaintext highlighter-rouge">UColorado_HEP</code> (University of Colorado in Boulder).  In this case, the Colorado clients failed the primary GeoIP lookup and the cloudflare workers fell back to using the <code class="language-plaintext highlighter-rouge">IATA</code> code from the request.  The requests from Colorado all where recieved by the Cloudflare Dallas colocation site, which is nearest the Houston cache.  The original GeoIP service choose the Kansas City cache, which is the correct decision.  It is unknown if the orignal GeoIP service choose KC cache because it knew the GeoIP location of the clients, or it defaulted to the Kansas default.</p>


<p>The second site where the Cloudflare worker implementation was incorrect was <code class="language-plaintext highlighter-rouge">SIUE-CC-production</code> (Southern Illinois University Edwardsville).  In this case, the original GeoIP service choose Chicago, while the new service choose Kansas.  Edwardsville is almost equal distance from both the KC cache and Chicago.  The difference in the distance to the caches is ~0.6 KM, with Chicago being closer.</p>


<!-- TODO: Find out why KC cache was choosen SIUE -->

<p>An example of a site that did not work with GeoIP was <code class="language-plaintext highlighter-rouge">ASU-DELL_M420</code> (Arizona Statue University).  The original service returned that the KC cache was the nearest.  The Cloudflare service gave the default Lat/Log if GeoIP failed, the middle of Kansas, but the data center serving the request had the airport code of <code class="language-plaintext highlighter-rouge">LAX</code> (Los Angeles).  The nearest cache to <code class="language-plaintext highlighter-rouge">LAX</code> is the UCSD cache, which is the correct cache decision.</p>


<p>During the evaluation, I originally used the Cloudflare worker development DNS address, <a href="https://stash-location.djw8605.workers.dev">stash-location.djw8605.workers.dev</a>.  Purdue University and the American Museum of Natural History sites both blocked the development DNS address.  The block was from an OpenDNS service which reported the domain had been linked to malware and phishing.  Since the DNS hostname was hours old, it’s likely that most <code class="language-plaintext highlighter-rouge">*workers.dev</code> domains were blocked.</p>


<h2 id="conclusion">Conclusion</h2>

<p>Improving the cache selection can improve the download effeciency.  It is left as future work to measure if the nearest geographical cache is the best choice.  While the OSDF is using GeoIP service for cache selection, it is important to select the correct cache.  Using the new Cloudflare service results in <strong>95%</strong> correct cache decision vs. <strong>86%</strong> with the original service.</p>


<p>Cloudflare Workers is also very affordable for the scale that the OSDF would require.  The first 100,000 requests are free, while it is $5/mo for the next 10 Million requests.  The OSPool runs between 100,000 to 230,000 jobs per day, easily fitting within the $5/mo tier.</p>