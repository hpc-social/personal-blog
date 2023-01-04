---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2019-12-19 02:21:37'
layout: post
original_url: https://www.gaborsamu.com/blog/intelligent_hpc/
slug: intelligent-hpc-keeping-hard-work-at-bay-es-
title: Intelligent HPC - Keeping Hard Work at Bay(es)
---

<p>Since the dawn of time, humans have looked for ways to make their lives easier. Over the centuries human ingenuity has given us inventions such as the wheel and simple machines – which help greatly with tasks that would otherwise be extremely laborious. Over the time, we’ve learned there are often alternatives to brute force ways of doing things. It’s this human reasoning that has driven the advancement we find in our world today.</p>

<p>Fast forward to this century where computer driven simulations have been developed as the third branch of scientific method supplementing theory and experimentation. For decades, simulation and modelling have delivered unprecedented capabilities to drive innovation for the betterment of the world. The need to run more simulations faster, has spurred the development of ever faster processors, networking and storage. The approach to speeding up simulations has been one of brute force. Faster computing to deliver faster results. But the insatiable desire to perform simulations faster has very real implications in today’s world – such as managing the power requirements of future supercomputers. It’s time in high performance computing to revisit the brute force approaches to achieve the next level of performance.</p>

<p><strong>Lessons from the past</strong></p>

<p>We sometimes forget that it’s important to look at lessons from the past, in order to create a better future. HPC simulations today are computationally intensive – and as the fidelity of models increases, so does the number of calculations and time to solution. Rethinking this laborious method for simulations, are there ways that we can cut down on the number of calculations performed? A calculation avoided, is time saved. Our lesson takes us back to 1763 when Thomas Bayes authored <em>“An Essay towards solving a Problem in the Doctrine of Changes”</em>, from which Bayes’ Theorem was developed.</p>

<p>In simple terms, Bayes’ theorem can be used to predict the probability of an outcome, based upon prior knowledge or information. What if Bayes’ theorem could be applied to computational simulations to determine the likelihood of a given iteration of a simulation to provide a useful outcome, and to discard those iterations where there is not likely a useful outcome? A calculation avoided, is time saved. As it turns out, applying Bayesian methods to HPC design can dramatically reduce the time to optimal product specification.</p>

<p><strong>Bayesian optimization at work</strong></p>

<p>To put Bayesian methods to the test, the engineers of the IBM Systems High Speed Bus Signal Integrity (HSB-SI) Team used software based upon the principles of Bayesian statistics called IBM Bayesian Optimization (IBO) developed by IBM Research. IBO was designed to accelerate computational workflows through the application of sophisticated algorithms. The HSB-SI team’s challenge is to minimize the time needed design validation simulation analysis of high-speed interfaces for the purpose of choosing an optimal configuration point, while maintaining or increasing the fidelity of the solution. In testing IBO, they wanted to reduce the number of simulations needed to reach the optimal configuration point for chip-to-chip communication.</p>

<blockquote>
<p><em>“Our team is taking advantage of state-of-the-art machine learning to design computer systems of the future.”</em>
<strong>Dale Becker, Ph.D., Chief Engineer Electrical Packaging Integration, IBM</strong></p>

</blockquote>
<p><a href="https://tci.taborcommunications.com/l/21812/2019-12-18/6kxfd4">The results were dramatic</a>. They achieved a 140x faster time to solution with higher accuracy than their legacy method. They used 99% less cores to arrive at a higher confidence solution with less than a 1% error rate using IBO.</p>

<p>With time to solution being a critical element of competitive advantage, the adoption of sophisticated statistical methods and machine learning to accelerate simulation workflows is destined to grow quickly. In our next article about innovations in HPC we will highlight multiple use cases where Bayesian optimized workflows are transforming HPC simulation-driven innovation.</p>

<p><strong>Originally published on HPCwire IBM Solution Channel on December 18, 2019</strong></p>