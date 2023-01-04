---
author: Derek Weitzel's Blog
author_tag: dweitzel
blog_subtitle: Thoughts from Derek
blog_title: Dereks Web
blog_url: https://derekweitzel.com/
category: dweitzel
date: '2020-10-11 06:00:00'
layout: post
original_url: https://derekweitzel.com/2020/10/11/xrootd-client-manager/
slug: xrootd-client-manager
title: XRootD Client Manager
---

<p>The validation project for XRootD Monitoring is moving to phase 2, scale
testing.  Phase 1 focused on correctness of single server monitoring.  <a href="https://doi.org/10.5281/zenodo.3981359">The
report</a> is available.</p>


<p>We are still forming the testing plan for the scale test of XRootD, but a
component of the testing will be multiple clients downloading from multiple
servers.  In addition, we must record exactly how much data each client reads
from each server in order to validate the monitoring with the client’s real behavior.</p>


<p>This level of testing will require detailed coordination and recording of client
actions.  I am not aware of a testing framework that can coordinate and record
accesses of multiple clients and servers, therefore I spent the weekend
developing a simple framework for coordinating these tests.</p>


<p>Some requirements for the application are:</p>


<ul>
  <li>Easy to use interface</li>
  <li>Easy to add clients and servers</li>
  <li>Authenticated access for clients, servers, and interface</li>
  <li>Storage of tests and results</li>
</ul>

<p>I chose <a href="https://heroku.com">Heroku</a> for prototyping this application.</p>


<h2 id="interface">Interface</h2>

<p>The web interface is available at https://xrootd-client-manager.herokuapp.com/.
I chose to host it on heroku as it is my go to for pet projects.  I will likely
move this over to OSG’s production kubernetes installation soon.  The entire
application is only the web interface and a back-end <a href="https://redis.io/">Redis</a>
data store.</p>


<figure class="">
  <img alt="Screenshot of web interface" src="https://derekweitzel.com/images/posts/XRootDClientManager/Interface.png" /><figcaption>
      Screenshot of simple web interface

    </figcaption></figure>

<p>The web interface shows the connected clients and servers.  The web interface
also connects to the web server with an persistent connection to update the list
of connected clients.</p>


<h2 id="client-communication">Client Communication</h2>

<p>Client communcation is handled through a Socket.IO connection.  Socket.IO is a
library that will at create a bi-directional event based communcation between
the client and the server.  The communcation is over websockets if possible, but
will fall back to HTTP long polling.  A good discussion of long polling vs.
websockets is available from
<a href="https://www.ably.io/blog/websockets-vs-long-polling/">Ably</a>.  The Socket.IO
connection is established between each worker, server, and web client and the
web server.</p>


<p>The difficult part is authenticating the Socket.IO connections.  We discuss this
in the security session.</p>


<h2 id="security">Security</h2>
<p>Securing the commands and web interface is required since the web interface is
sending commands to the connected worker nodes and servers.</p>


<h3 id="socketio-connections">Socket.IO Connections</h3>

<p>The Socket.IO connection is secured with a shared key.  The communication flow
for a non-web client (worker/server):</p>


<ol>
  <li>A JWT is created from the secret key.  The secret key is communicated through
a separate secure channel.  In most cases, it will be through the command
line arguments of the client.  The JWT has a limited lifetime and a scope.</li>
  <li>The client registers with the web server, with an Authentication bearer token
in the headers.  The registration includes details about the client.  It
returns a special (secret) <code class="language-plaintext highlighter-rouge">client_id</code> that will be used to authenticate the
Socket.IO connection.  The registration is valid for 30
seconds before the <code class="language-plaintext highlighter-rouge">client_id</code> is no longer valid.</li>
  <li>The client creates a Socket.IO connection with the <code class="language-plaintext highlighter-rouge">client_id</code> in the request
arguments.</li>
</ol>

<h3 id="web-interface">Web Interface</h3>

<p>The web interface is secured with an OAuth login from GitHub.  There is a whitelist
of allowed GitHub users that can access the interface.</p>


<p>The flow for web clients connecting with Socket.IO is much easier since they are already authenticated
with OAuth from GitHub.</p>


<ol>
  <li>The user authenticates with GitHub</li>
  <li>The Socket.IO connection includes cookies such as the session, which is a
signed by a secret key on the server.  The session’s github key is compared to the
whitelist of allowed users.</li>
</ol>

<h2 id="storage-of-tests-and-results">Storage of tests and results</h2>

<p>Storage of the tests and results are still being designed.  Most likely, the
tests and results will be stored in a database such as Postgres.</p>


<h1 id="conclusions">Conclusions</h1>

<p><a href="https://heroku.com">Heroku</a> provides a great playing ground to prototype these
web applications. I hope that I can find an alternative eventually that will run on
OSG’s production kubernetes installation.</p>


<p>The web application is still be developed, and there is much to be done before
it can be fully utilized for the scale validation.  But, many of the difficult
components are completed, including the communcation and eventing, secure web
interface, and clients.</p>


<p>The GitHub repos are available at:</p>


<ul>
  <li><a href="https://github.com/djw8605/xrootd-client-manager">XRootD Client Manager</a></li>
  <li><a href="https://github.com/djw8605/xrootd-ws-client">XRootD Client</a></li>
</ul>