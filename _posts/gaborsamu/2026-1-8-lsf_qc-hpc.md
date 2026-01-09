---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2026-01-08 14:22:59'
layout: post
original_url: https://www.gaborsamu.com/blog/lsf_qc-hpc/
slug: orchestrating-hybrid-quantum-classical-workflows-with-ibm-lsf-inside-the-sqd-workflow-demo-at-sc25
title: Orchestrating Hybrid Quantum–Classical Workflows with IBM LSF- Inside the SQD
  Workflow Demo at SC25
---

<p>As we enter 2026, it seems that SC25 is far off in our rearview mirror. But it&rsquo;s only been a bit over a month since the HPC world converged on St. Louis, Missouri for the annual <a href="https://sc25.supercomputing.org/">Supercomputing 2025</a> (SC25) event. SC25 signaled one emerging trend: the exploration of hybrid workflows combining quantum and classical computing, offering a look at how these technologies can work synergistically over time. This was indeed the main topic of the 1st Annual Workshop on Large-Scale Quantum-Classical Computing, a workshop which I found to be very insightful.</p>

<p>At the IBM booth, we showcased how <a href="https://www.ibm.com/products/hpc-workload-management">IBM LSF</a> can schedule and orchestrate a hybrid quantum–classical workflow across IBM Quantum systems and classical x86 compute.  The demo featured the Sample-based Quantum Diagonalization (SQD) workflow, to estimate the ground-state energy of a Hamiltonian representing a molecular system. SQD is part of the <a href="https://quantum.cloud.ibm.com/docs/en/guides/qiskit-addons-sqd">IBM Qiskit add-ons</a>.</p>

<p>Before diving into the details on what was demonstrated at SC25, and how LSF was used to manage the workflow, I would like to acknowledge that this work was supported by the Hartree Center for Digital Innovation, a collaboration between UKRI-STFC and IBM. The demonstration was created in close collaboration with Vadim Elisseev and Ritesh Krishna from IBM Research, alongside Gábor Samu and Michael Spriggs from IBM. Additionally, this post does not aim to provide an in-depth look at SQD itself. Rather the focus is on how LSF can manage hybrid quantum-classical workflows across a heterogeneous environment comprised of both quantum and classical resources.</p>

<p><strong>Hybrid workflows are not new</strong></p>

<p>For three decades, we have seen the use of accelerators in HPC to drive performance—from GPUs to FPGAs and other specialized architectures. Effective scheduling of tasks in these heterogeneous environments has always been a key consideration for efficiency, scalability—and to maximize the ROI in commercial HPC environments. As resource topologies grow more complex, scheduling must account for characteristics such as connectivity, latency, and dependency constraints across increasingly diverse infrastructures. Quantum Processors (QPUs) are now making their appearance as complementary resources within HPC workflows, aim at challenges such as specific optimization problems, many-body physics and quantum chemistry.</p>

<p><strong>Demo details</strong></p>

<p>The IBM LSF cluster was deployed on IBM Cloud using the LSF Deployable Architecture, which rapidly deploys and configures a ready-to-use HPC environment. IBM Research provided integration components for LSF in the form of esub and jobstarter scripts. These scripts enable LSF to query the cloud-based IBM Quantum Platform to determine which QPUs are available for a given user account and meet the qubit requirements specified at job submission. The list of eligible QPUs is then sorted by queue length, and the system with the shortest queue is selected as the target for the quantum circuit. These integration scripts (esub and jobstarter) are intended to be made open source at a later time.</p>

<p>The LSF environment was deployed on IBM Cloud using the <a href="https://cloud.ibm.com/catalog/architecture/deploy-arch-ibm-hpc-lsf-1444e20a-af22-40d1-af98-c880918849cb-global">LSF Deployable Architecture</a> v3.1.0:</p>

<ul>
<li>LSF 10.1.0.15</li>
<li>RHEL 8.10</li>
<li>IBM Cloud profile bx2-16x64 (compute hosts)</li>
</ul>
<p>The IBM Qiskit package versions used:</p>

<ul>
<li>qiskit v2.2.1</li>
<li>qiskit-addon-sqd v0.12.0</li>
<li>qiskit-ibm-runtime v0.43.0</li>
</ul>
<p>The SQD Python program is available as part of the IBM Qiskit Add-ons (see details here). For this demonstration, the original monolithic SQD script was refactored into four smaller Python programs—each representing a distinct step in the workflow. These steps map directly to LSF jobs, enabling orchestration of the workflow across the quantum and classical HPC resources as shown in the architecture diagram (Figure 1):</p>

<ul>
<li><strong>Stage 1</strong> map the inputs to a quantum problem.</li>
<li><strong>Stage 2</strong> optimizes the problem for quantum hardware execution—this is where the circuit is transpiled and optimized for the target QPU</li>
<li><strong>Stage 3</strong> executes the circuit on the QPU using Qiskit primitives</li>
<li><strong>Stage 4</strong> performs post-processing and returns the result in the desired classical format</li>
</ul>
<p><figure><img src="https://www.gaborsamu.com/images/figure1_lsfqc.png" />
</figure>

<em>Figure 1 LSF hybrid quantum-classical workflow demo (Vadim Elisseev, IBM Research)</em></p>

<p>For this demonstration, we used IBM LSF Application Center—a web-based interface for job submission and management. LSF Application Center supports application templates, which simplify job submission by providing predefined forms. Templates were created for both the SQD workflow and the Jupyter Notebook application, which is used to visualize the workflow results.</p>

<p><strong>Demo execution steps</strong></p>

<ul>
<li>We start by using the SQD template to submit an instance of the SQD workflow (Figure 2) which is used to calculate an approximate ground-energy state of the nitrogen molecule (N2). The submission form is customized to let users specify the script for each step of the workflow and specify the desired number of qubits required on the QPU for the quantum circuit. This parameter is used by LSF to select the appropriate quantum system from the available resources. Note that jobs are submitted to LSF with a done dependency condition, ensuring that each stage runs only after the previous one completes successfully. Stage 2 begins after Stage 1, Stage 3 follows Stage 2, and Stage 4 executes once Stage 3 has finished</li>
</ul>
<p><figure><img src="https://www.gaborsamu.com/images/figure2a_lsfqc.png" />
</figure>

<em>Figure 2 LSF Application Center SQD submission form</em></p>

<ul>
<li>Next, we submit an instance of the Jupyter Notebook to monitor the workflow initiated in Step 1. This notebook is designed for this demonstration to visualize the status of each workflow step, displaying results as they successfully complete. Figure 3 shows the Jupyter submission form.</li>
</ul>
<p><figure><img src="https://www.gaborsamu.com/images/figure3a_lsfqc.png" />
</figure>

<em>Figure 3 LSF Application Center Jupyter Notebook submission form</em></p>

<ul>
<li>The Workload view in the LSF Application Center can be used to monitor the progress of each job within the workflow. Additionally, the Jupyter Notebook instance can be accessed here via the provided hyperlink. Figure 4 shows the workload view in LSF Application Center. This shows a list of jobs in the LSF system.</li>
</ul>
<p><figure><img src="https://www.gaborsamu.com/images/figure4a_lsfqc.png" />
</figure>

<em>Figure 4 LSF Application Center workload view</em></p>

<ul>
<li>As each stage of the SQD workflow completes, the Jupyter Notebook displays the corresponding output in new browser tabs. This includes qubit coupling maps for the QPUs available on the IBM Quantum Platform for the specific account, a diagram of the circuit mapped to the selected QPU, readings from the QPU, and a plot of the estimated ground-state energy of the N2 molecule.</li>
</ul>
<p><figure><img src="https://www.gaborsamu.com/images/figure5_lsfqc.png" />
</figure>

<em>Figure 5 Output from each step of the SQD workflow (Vadim Elisseev, IBM Research)</em></p>

<ul>
<li>Given that demo environment was built using the LSF Deployable Architecture, IBM Cloud Monitoring is automatically configured. It provides a dashboard for the underlying cloud infrastructure, including detailed hardware metrics. In addition, an LSF Dashboard is available through IBM Cloud Monitoring, showing overall cluster metrics such as total jobs, job status, and queue distribution, along with scheduler performance trends over time. IBM Cloud Monitoring infrastructure view and LSF dashboard are shown in Figure 5.</li>
</ul>
<p><figure><img src="https://www.gaborsamu.com/images/figure6_lsfqc.png" />
</figure>

<em>Figure 6 IBM Cloud Monitoring: Infrastructure view, and LSF dashboard</em></p>

<p>A video recording of the end-to-end demonstration can be found <a href="https://community.ibm.com/community/user/viewdocument/demonstration-of-managing-hybrid-qu?CommunityKey=74d589b7-7276-4d70-acf5-0fc26430c6c0&amp;tab=librarydocuments">here</a>.</p>

<p><strong>Conclusions</strong></p>

<p>This demo marked a milestone by demonstrating that IBM Spectrum LSF can seamlessly orchestrate quantum and classical compute resources for a unified workflow. This example demonstrates a practical approach to integrating quantum capabilities into an existing HPC environment running IBM LSF.</p>

<p>This capability lays the foundation for hybrid computing pipelines that integrate emerging quantum hardware into established HPC environments. As organizations adopt these architectures and tools mature, we can expect production-grade workflows tackling complex problems across domains. The future of HPC is not a choice between classical or quantum—it is their convergence, working together to unlock new computational possibilities.</p>

<p>The topic of scheduling for hybrid quantum-classical environments will be the subject of an upcoming paper &ldquo;On Topological Aspects of Workflows Scheduling on Hybrid Quantum - High Performance Computing Systems&rdquo; by Vadim Elisseev, Ritesh Krishna, Vasileios Kalantzis, M. Emre Sahin and Gábor Samu.</p>