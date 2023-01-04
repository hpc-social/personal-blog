---
author: Derek Weitzel's Blog
author_tag: dweitzel
blog_subtitle: Thoughts from Derek
blog_title: Dereks Web
blog_url: https://derekweitzel.com/
category: dweitzel
date: '2022-09-14 06:00:00'
layout: post
original_url: https://derekweitzel.com/2022/09/14/dashboards/
slug: dashboards-for-learning-data-visualizations
title: Dashboards for Learning Data Visualizations
---

<p>Creating dashboards and data visualizations are a favorite past time of mine.  Also, I jump at any chance to learn a new technology.  That is why I have spent the last couple of months building dashboards and data visualizations for various projects while learning several web technologies.</p>


<p>Through these dashboards, I have learned many new technologies:</p>

<ul>
  <li><a href="https://reactjs.org/">React</a> and <a href="https://nextjs.org/">NextJS</a></li>
  <li>Mapping libraries such as <a href="https://leafletjs.com/">Leaflet</a> and <a href="https://www.mapbox.com/">Mapbox</a></li>
  <li>CSS libraries such as <a href="https://derekweitzel.com/2022/09/14/dashboards/TailwindCSS">TailwindCSS</a></li>
  <li>Data access JS clients for <a href="https://derekweitzel.com/2022/09/14/dashboards/Elasticsearch">Elasticsearch</a> and <a href="https://derekweitzel.com/2022/09/14/dashboards/Prometheus">Prometheus</a></li>
  <li>Website hosting service <a href="https://derekweitzel.com/2022/09/14/dashboards/Vercel">Vercel</a></li>
  <li>Data Visualization library <a href="https://derekweitzel.com/2022/09/14/dashboards/D3.js">D3.js</a></li>
</ul>

<h2 id="gp-argo-dashboard"><a href="https://gp-argo.greatplains.net/">GP-ARGO Dashboard</a></h2>

<p><a href="https://gp-argo.greatplains.net/">The Great Plains Augmented Regional Gateway to the Open Science Grid</a> (GP-ARGO) is a regional collaboration of 16 campuses hosting computing that is made available to the OSG.  My goal with the GP-ARGO dashboard was to show who is using the resources, as well as give high level overview of the region and sites hosting GP-ARGO resources.</p>


<p>The metrics are gathered from OSG’s <a href="https://gracc.opensciencegrid.org/">GRACC Elasticsearch</a>.  The list of projects are also from GRACC, and the bar graph in the bottom right are from OSG is simply an iframe to a grafana panel from GRACC.</p>


<p>Technologies used: <a href="https://reactjs.org/">React</a>, <a href="https://nextjs.org/">NextJS</a>, <a href="https://leafletjs.com/">Leaflet</a>, <a href="https://github.com/elastic/elasticsearch-js">Elasticsearch</a></p>


<p><strong>Repo:</strong> <a href="https://github.com/djw8605/gp-argo-map">GP-ARGO Map</a></p>


<p><a href="https://gp-argo.greatplains.net/"><img alt="GP-ARGO" src="https://derekweitzel.com/images/posts/Dashboards/gp-argo-screenshot.png" /></a></p>


<h2 id="osdf-website"><a href="https://osdf.osg-htc.org/">OSDF Website</a></h2>

<p>My next website was the <a href="https://osdf.osg-htc.org/">Open Science Data Federation</a> landing page.  I was more bold in the design of the OSDF page.  I took heavy inspiration from other technology websites such as the <a href="https://www.mapbox.com/">Mapbox</a> website and the <a href="https://k8slens.dev/">Lens</a> website.  The theme is darker and it was also my first experience with the TailwindCSS library.  Additionally, I learned the CSS <a href="https://en.wikipedia.org/wiki/CSS_Flexible_Box_Layout">flexbox</a> layout techniques.</p>


<p>The spinning globe is using the <a href="https://globe.gl/">Globe.gl</a> library.  The library is great to create visualizations to show distribution throughout the world.  On the globe I added “transfers” between the OSDF origins and caches.  Each origin sends transfers to every cache in the visualization, though it’s all just animation.  There is no data behind the transfers, it’s only for visual effect.  Also, on the globe, each cache location is labeled.  The globe can be rotated and zoomed with your mouse.</p>


<p>The number of bytes read and files read is gathered using the Elasticsearch client querying GRACC, the OSG’s accounting service.  The OSG gathers statistics on every transfer a cache or origin perform.  Additionally, we calculate the rate of data transfers and rate of files being read using GRACC.</p>


<p>One unique feature of the OSDF website is the resiliency of the bytes read and files read metrics.  We wanted to make sure that the metrics would be shown even if a data component has failed.  The metrics are gathered in 3 different ways for resiliency:</p>

<ol>
  <li>If all components are working correctly, the metrics are downloaded from the OSG’s Elasticsearch instance.</li>
  <li>If OSG Elasticsearch has failed, the dashboard pulls saved metrics from NRP’s S3 storage.  The metrics are saved everytime they are succesfully gathered from Elasticsearch, so they should be fairly recent.</li>
  <li>The metrics are gathered and saved on each website build.  The metrics are static and immediatly available upon website load.  If all else fails, these saved static metrics are always available, even if they may be old.</li>
</ol>

<p>Technologies used: <a href="https://reactjs.org/">React</a>, <a href="https://nextjs.org/">NextJS</a>, <a href="https://globe.gl/">Globe.gl</a></p>


<p><strong>Repo:</strong> <a href="https://github.com/djw8605/osdf-website">OSDF Website</a></p>


<p><a href="https://osdf.osg-htc.org/"><img alt="OSDF" src="https://derekweitzel.com/images/posts/Dashboards/osdf-screenshot.png" /></a></p>


<h2 id="nrp-dashboard"><a href="https://dash.nrp-nautilus.io/">NRP Dashboard</a></h2>

<p>The National Research Platform dashboard is largely similar to the <a href="https://derekweitzel.com/2022/09/14/dashboards/#gp-argo-dashboard">GP-ARGO</a> dashboard.  It uses the same basic framework and technologies.  But, the data acquisition is different.</p>


<p>The metrics shown are the number of gpus allocated, number of pod running, and the number of active research groups.  The metrics are gathered from the NRP’s <a href="https://prometheus.io/">prometheus</a> server on-demand.  The graph in the background of the metric is generated with <a href="https://d3js.org/">D3.js</a>.</p>


<p>Technologies used: <a href="https://reactjs.org/">React</a>, <a href="https://nextjs.org/">NextJS</a>, <a href="https://d3js.org/">D3.js</a>, <a href="https://github.com/siimon/prom-client">Prometheus</a>, <a href="https://tailwindcss.com/">TailwindCSS</a></p>


<p><strong>Repo:</strong> <a href="https://github.com/djw8605/nrp-map-app">NRP Map App</a></p>


<p><a href="https://dash.nrp-nautilus.io/"><img alt="NRP Dashboard" src="https://derekweitzel.com/images/posts/Dashboards/nrp-dashboard-screenshot.png" /></a></p>


<h2 id="pnrp-website"><a href="https://nrp-website.vercel.app/">PNRP Website</a></h2>

<p>The <a href="https://www.nsf.gov/awardsearch/showAward?AWD_ID=2112167&amp;HistoricalAwards=false">Prototype National Research Platform</a> is a NSF research platform.  The dashboard is also in prototype stage as the PNRP hardware is not fully delivered and operational yet.</p>


<p>The dashboard is my first experience with a large map from <a href="https://www.mapbox.com/">Mapbox</a>.  I used a <a href="https://visgl.github.io/react-map-gl/">React binding</a> to interface with the <a href="https://www.mapbox.com/">Mapbox</a> service.  Also, when you click on a site, it zooms into the building where the PNRP hardware will be hosted.</p>


<p>The transfer metrics come from the NRP’s prometheus which shows the bytes moving into and out of the node.  The transfer metrics are for cache nodes nearby the sites, but once PNRP hardware becomes operational the transfer metrics will show the site’s cache.</p>


<p>Technologies Used: <a href="https://reactjs.org/">React</a>, <a href="https://nextjs.org/">NextJS</a>, <a href="https://www.mapbox.com/">Mapbox</a>, <a href="https://tailwindcss.com/">TailwindCSS</a>, <a href="https://github.com/siimon/prom-client">Prometheus</a></p>


<p><strong>Repo:</strong> <a href="https://github.com/djw8605/nrp-website">NRP Website</a></p>


<p><a href="https://nrp-website.vercel.app/"><img alt="PNRP Website" src="https://derekweitzel.com/images/posts/Dashboards/nrp-website-screenshot.png" /></a></p>