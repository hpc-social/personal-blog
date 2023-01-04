---
author: Derek Weitzel's Blog
author_tag: dweitzel
blog_subtitle: Thoughts from Derek
blog_title: Dereks Web
blog_url: https://derekweitzel.com/
category: dweitzel
date: '2017-09-07 19:20:04'
layout: post
original_url: https://derekweitzel.com/2017/09/07/installing-scitokens-on-a-mac/
slug: installing-scitokens-on-a-mac
title: Installing SciTokens on a Mac
---

<p>In case I ever have to install <a href="https://scitokens.org/">SciTokens</a> again, the steps I took to make it work on my mac.  The most difficult part of this is installing openssl headers for the jwt python library.  I followed the advice on this <a href="https://solitum.net/openssl-os-x-el-capitan-and-brew/">blog post</a>.</p>


<ol>
  <li>Install <a href="https://brew.sh/">Homebrew</a></li>
  <li>
    <p>Install openssl:</p>


    <div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code> brew install openssl
</code></pre></div>
    </div>

  </li>
  <li>
    <p>Download the SciTokens library:</p>


    <div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code> git clone https://github.com/scitokens/scitokens.git
 cd scitokens
</code></pre></div>
    </div>

  </li>
  <li>
    <p>Create the virtualenv to install the <a href="https://jwt.io/">jwt</a> library</p>


    <div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code> virtualenv jwt
 . jwt/bin/activate
</code></pre></div>
    </div>

  </li>
  <li>
    <p>Install jwt pointing to the Homebrew installed openssl headers:</p>


    <div class="language-plaintext highlighter-rouge"><div class="highlight"><pre class="highlight"><code> env LDFLAGS="-L$(brew --prefix openssl)/lib" CFLAGS="-I$(brew --prefix openssl)/include" pip install cryptography PyJWT
</code></pre></div>
    </div>

  </li>
</ol>