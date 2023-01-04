---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2019-06-05 03:21:37'
layout: post
original_url: https://www.gaborsamu.com/blog/beyond_simulation_isc/
slug: beyond-simulation-harnessing-ai-for-next-generation-hpc-at-isc
title: Beyond Simulation – Harnessing AI for Next-Generation HPC at ISC
---

<p>Computer simulation has become a staple technique in many disciplines – so much so that it often described as the “third pillar” of the scientific method. Alongside theory and experimentation, simulation is used in everything from automotive design to computational chemistry to forecasting weather and market movements.</p>

<p>Simulation helps us solve problems that are too difficult, time-consuming, or expensive to solve empirically – for example, what is the optimal design and material for an impeller in a centrifugal pump? Or what failure states might exist in a semiconductor design from which a device can’t recover?</p>

<p>By devising accurate mathematical models and approximating those numerically in software, we can predict the behavior of real-world systems based on various parameters and a set of initial conditions. The better the model, the quality of the input data, and the more computing power that can be brought to bear, the better the prediction.</p>

<p><strong>Simulation vs. analytics</strong></p>

<p>High-performance data analytics (HPDA) and computer simulation are increasingly joined at the hip. Analytic techniques are sometimes used to improve simulation – providing better quality datasets to feed a simulation model, for example. Other times, simulation helps improve analytics – back-testing the performance of a financial or weather model over past data, for example, to gain confidence in a model’s predictive quality.</p>

<p>While simulation has served us well, it has limits. The quality of a predictive model is only as good as our ability to identify features useful in making accurate predictions. For some problems, such as are structural mechanics, the features required to build a predictive model are relatively well known. For other problems, such as financial markets or weather models, the number of potential parameters is vast, and their effects are sometimes poorly understood, significantly affecting the quality of the result.</p>

<p><strong>A fourth pillar in the scientific method</strong></p>

<p>AI is rapidly emerging as a “fourth pillar” in the scientific method complementing theory, experimentation, and simulation techniques. Inference allows computers to make educated guesses about future results without the need to go through a full-blown simulation.</p>

<p>In fact, the AI development process can be modeled as automation of the scientific method where the steps are:</p>

<ol>
<li>Observe</li>
<li>Hypothesize</li>
<li>Test Hypothesis</li>
<li>(return to #1)</li>
</ol>
<p><strong>The power of &ldquo;better guesses&rdquo;</strong></p>

<p>Humans often infer things based on prior knowledge intuitively. For example, back to our impeller design, if a centrifugal pump needs to handle a viscous or corrosive liquid, the human engineer might know intuitively that a strong, non-reactive material like stainless steel is a good choice. By making educated guesses on materials and other parameters, the problem-space to be simulated is reduced dramatically.</p>

<p>When dealing with complex problems, however, our human ability to make such inferences breaks down. Even for subject matter experts, problems like modeling chemical reactions or predicting how a semiconductor will behave, are beyond our experience. The systems we need to model are too complex and involve too many parameters.</p>

<p><strong>Intelligent Simulation</strong></p>

<p>Fortunately, computers are very good at sifting through vast amounts of data and detecting patterns not obvious to humans. The best way to boost simulation performance is often to avoid simulations that will be irrelevant and not useful. By applying machine learning and other AI techniques to make informed guesses about what parameters and simulations will be most useful in solving a problem we can:</p>

<ul>
<li>Reduce the number of simulations required</li>
<li>Provide higher resolution simulations and more trustworthy models</li>
<li>Reduce costs and cycle times wherever computer simulation is used</li>
</ul>
<p>Intelligent simulation helps us more effectively explore a problem space by predicting what regions, data, and exploratory techniques are most likely to be useful and omitting the rest.</p>

<p><strong>Bayesian Optimization</strong></p>

<p>In probability theory, Bayes’ theorem describes the probability of an event, based on prior knowledge of conditions that might be related to the event. It turns out that Bayesian analysis is a particularly effective way to capture common sense information from data, to help make better predictions, thus reducing the amount of computer simulation required. IBM has developed a Bayesian optimization accelerator that can function as an HPC advisory engine.</p>

<p>Powered by Bayesian optimization libraries, the system helps scientists exploit these state-of-the-art techniques to computer simulation in multiple industries without the need for deep AI expertise. Bayesian optimization has demonstrated that it can reduce simulation requirements by half with no disruption to the existing HPC infrastructure, dramatically improving HPC productivity.</p>

<p><strong>Harnessing AI for Next-Generation HPC @ ISC 2019</strong></p>

<p>At this year’s ISC conference in Frankfurt, Germany, you can learn more about IBM solutions for AI and HPC –</p>

<ul>
<li>Learn how accelerating simulations with <a href="https://www.ibm.com/blogs/think/2018/11/fueling-the-hpc-transformation-with-ai/">Bayesian optimization</a> has the potential to help you perform simulations in half the time</li>
<li>Learn how IBM Q researchers are putting machine learning on the path to quantum advantage</li>
<li>Try out <a href="https://rxn.res.ibm.com/">IBM RXN for Chemistry</a> and learn how AI techniques are helping automated discovery for organic chemistry by predicting chemical reactions</li>
<li>Finally, learn how a CPPM PCIe40 data acquisition adapter in an IBM POWER9 based system can help advance state-of-the-art research in high-energy physics and other applications</li>
</ul>
<p>Stop by the IBM booth (D-1140 in the exhibit hall) to see demos Power Systems and Spectrum Storage to Spectrum Computing and Watson Machine Learning Accelerator.</p>