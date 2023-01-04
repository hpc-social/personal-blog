---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2015-05-29 03:49:58'
layout: post
original_url: https://www.gaborsamu.com/blog/lsf_docker_whale/
slug: ibm-platform-lsf-and-docker-a-whale-of-a-time-
title: IBM Platform LSF and Docker- A Whale of a time!
---

<p>Containers are useful. Whether you&rsquo;re shipping things across the blue seas or
encapsulating applications on a computer system, they provide numerous benefits. HPC Administrators will know that applications today can depend upon multiple
packages, libraries and environments. <a href="https://www.docker.com/">Docker</a>, a container technology for Linux, based on well proven technologies brings together ease of setup, use and
efficiency to application management. Leveraging Docker in High-Performance
Computing is one approach to address application &ldquo;dependency hell&rdquo;, as well
as easing transition to the cloud.</p>

<p>Workload managers are commonly used in High Performance Computing environments
to drive effective use of compute resources and ensure alignment of resources
with business priorities. IBM Platform LSF, a leading workload management
family of products provides support for workloads to run within user-specified
Docker containers by way of an integration package available as an open beta
on Service Management Connect.</p>

<p>By leveraging the rich Platform LSF plugin framework, the Docker integration
works seamlessly and allows users to specify a defined Docker image as a
submission option. All resource constraints, environment variables are
automatically passed to the container thanks to the integration and Platform
LSF job lifecycle management functions including monitoring resource usage as
well as control actions (i.e. suspend, resume and terminate) are also supported
for Docker containers.</p>

<p>Ease the burden of administration and ensure consistency with IBM Platform LSF
and Docker! - and have a whale of a time!</p>