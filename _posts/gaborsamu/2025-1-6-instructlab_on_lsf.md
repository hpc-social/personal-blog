---
author: Ramblings of a supercomputing enthusiast.
author_tag: gaborsamu
blog_subtitle: Recent content in Blogs on Technical Computing Goulash
blog_title: Blogs on Technical Computing Goulash
blog_url: https://www.gaborsamu.com/blog/
category: gaborsamu
date: '2025-01-06 19:36:24'
layout: post
original_url: https://www.gaborsamu.com/blog/instructlab_on_lsf/
slug: fine-tuning-ai-models-with-instructlab-under-ibm-lsf
title: Fine tuning AI models with InstructLab under IBM LSF
---

<p><strong>Overview</strong></p>

<p>All the best for 2025! This blog looks back on a demo which I created for <a href="https://sc24.supercomputing.org">SC24</a>
last November to demonstrate InstructLab workflows running on an <a href="https://www.ibm.com/products/hpc-workload-management">IBM LSF</a>
cluster. Let’s begin with a bit of background. I’d like to thank Michael
Spriggs, STSM, IBM LSF for his contributions to this blog.</p>

<p>When I think of tuning, what immediately comes to my mind are visions of an
expert mechanic trying to extract the most from an engine. This blog is
focused on an entirely different type of tuning, AI model tuning. Like tuning
an engine, AI model tuning can be used to ensure a better fit for a given AI
model for your business.</p>

<p>Released by IBM and Red Hat in May 2024, <a href="https://research.ibm.com/blog/instruct-lab">InstructLab</a> is an open-source project
which provides the ability to fine-tune LLMs by adding skills and knowledge,
without having to retrain the model from scratch. InstructLab can run on
resource-constrained systems such as laptops, but also supports GPUs. Much has
been written about InstructLab and this blog is not intended to provide an
in-depth look at InstructLab. Rather, the objective here is to demonstrate how
InstructLab workloads can be distributed and managed in a high-performance
computing cluster with GPUs using the IBM LSF workload scheduler. Recently, IBM
published a paper describing the infrastructure used to train the Granite family
of AI foundation models. The paper describes the Vela and Blue Vela environments
in detail. In particular, the Blue Vela environment is built on a software stack
using Red Hat Enterprise Linux, IBM LSF and Storage Scale. Learn more in the
detailed paper <a href="https://arxiv.org/abs/2407.05467">here</a>.</p>

<p>The demo workflow consists of two LSF jobs. The first job generates synthetic
data, which is used to teach the LLM new skills or knowledge. The second job,
which depends upon the successful completion of the first, is the training job,
where the new skills or knowledge are incorporated into an existing base model.
A simple LSF job dependency is used to ensure the training job only runs after
the successful completion of the synthetic data generation step.</p>

<p>The environment used is equipped with Nvidia GPUs.  InstructLab jobs will be
run with the options for GPU support, and the jobs will be submitted to LSF
with the appropriate GPU scheduling directives. Furthermore, it is assumed that
the users' $HOME directory is available on all hosts in the cluster. Note that I
require neither root access, nor a user account that is an LSF administrator, to
install and use InstructLab on the LSF cluster.</p>

<p><strong>Configuration</strong></p>

<p>The HPC cluster is configured as follows:</p>

<ul>
<li>Red Hat Enterprise Linux v8.8</li>
<li>IBM LSF v10.0.1.15</li>
<li>InstructLab v0.19.4</li>
<li>Miniforge v3 (24.9.0-0)</li>
<li>NVIDIA CUDA v12.6</li>
<li>Compute nodes are equipped with 8 x Nvidia H100 GPUs</li>
</ul>
<p><strong>Install InstructLab</strong></p>

<ol>
<li>Log in to a compute node in the LSF cluster equipped with GPUs. If ssh access
is disabled to compute nodes, then submit an interactive LSF batch job. This job
requests 8 GPUs on a single system and will set them to exclusive execution
mode.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ bsub -Is -R "span[hosts=1]" -gpu "num=8:j_exclusive=yes" bash</code></pre></div>

<ol start="2">
<li>Install and set up a Conda environment. This will enable you to install a
self-contained Conda environment for your user account with the necessary
Python version needed for InstructLab. Miniforge is installed in the default
location and the option to update the users shell profile to start the Conda
environment are selected. We assume here a shared $HOME directory.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ cd $HOME
$ curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
$ bash Miniforge3-$(uname)-$(uname -m).sh</code></pre></div>

<ol start="3">
<li>Before proceeding, you must logout and log back in to activate the
environment. Next, a Conda environment is created with name <em>my_env</em>. Here we’ll
specify Python v3.11, which is a requirement for InstructLab.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">conda create --name my_env -c anaconda python=3.11
conda activate my_env</code></pre></div>

<ol start="4">
<li>Next, install InstructLab. Here, version 0.19.4 of InstructLab is specified.
This was the version of InstructLab available in the timeframe preceding the
SC24 event. Follow the installation steps in the official InstructLab
documentation <a href="https://github.com/instructlab/instructlab?tab=readme-ov-file#-installing-ilab">here</a>.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ pip install instructlab==0.19.4</code></pre></div>

<ol start="5">
<li>Next, perform the installation of InstructLab with Nvidia CUDA support. This
is required for InstructLab to utilize the GPUs. Without this step, InstructLab
will run on the CPUs. Note that CUDA v12.6 is installed on the system and the
variables set below reflect this.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ export CMAKE_ARGS="-DLLAMA_CUBLAS=on -DCUDA_PATH=/usr/local/cuda-12.6 -DCUDAToolkit_ROOT=/usr/local/cuda-12.6 -DCUDAToolkit_INCLUDE_DIR=/usr/local/cuda-12/include -DCUDAToolkit_LIBRARY_DIR=/usr/local/cuda-12.6/lib64"
$ export PATH=/usr/local/cuda-12.6/bin:$PATH
$ pip cache remove llama_cpp_python
$ CMAKE_ARGS="-DLLAMA_CUDA=on -DLLAMA_NATIVE=off" pip install 'instructlab[cuda]'
$ pip install vllm@git+https://github.com/opendatahub-io/vllm@v0.6.2</code></pre></div>

<p><strong>Configure InstructLab</strong></p>

<ol>
<li>With the installation of InstructLab complete, the next step is to run the
initialization. This will setup paths to models, taxonomy repo as well as the
GPU configuration.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ ilab config init</code></pre></div>

<ol start="2">
<li>By default InstructLab stores models, training checkpoints and other files
within <em>~/.cache</em> and <em>~/.local/share/instructlab</em>. If you have limited storage
capacity available in $HOME, then you may opt to disable training checkpoint
files. This can be done by setting the following option in <em>~/.config/instructlab/config.yaml</em> as follows.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">train:

  checkpoint_at_epoch: false</code></pre></div>

<ol start="3">
<li>Next, we download the required models. The ilab model list command can be
used to list the models which are available. Note that a <a href="https://huggingface.co">HuggingFace</a> token is
required to download certain models. Please set HF_TOKEN in the environment
with the appropriate token.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ export HF_TOKEN=&lt;HuggingFace token&gt;
$ ilab model download
$ ilab model download --repository=instructlab/granite-7b-lab
$ ilab model list

+--------------------------------------+---------------------+---------+

| Model Name                           | Last Modified       | Size    |

+--------------------------------------+---------------------+---------+
| instructlab/granite-7b-lab           | 2024-12-27 20:37:29 | 12.6 GB |
| mistral-7b-instruct-v0.2.Q4_K_M.gguf | 2024-12-27 16:55:46 | 4.1 GB  |
| merlinite-7b-lab-Q4_K_M.gguf         | 2024-12-27 16:48:39 | 4.1 GB  |
+--------------------------------------+---------------------+---------+</code></pre></div>

<p><strong>Generate synthetic data &amp; AI model training</strong></p>

<p>Next, is the synthetic data generation step, which will be executed on GPUs.
This step is a prerequisite to teaching the LLM new skills/knowledge via
training.</p>

<ol>
<li>
<p>Here we use example knowledge from the InstructLab github about Taylor Swift
fans, who are known as “Swifties”. This is timely because Taylor Swift recently
wrapped up 6 concerts in Toronto, Canada, where I happen to be based. Copy
attribution.txt and qna.yaml from the following <a href="https://github.com/mairin/taxonomy/tree/swifties/knowledge/arts/music/fandom/swifties">location</a>.</p>

</li>
<li>
<p>By default, the InstructLab taxonomy is found in <em>~/.local/share/instructlab/taxonomy</em>. Here we create the directories fandom/swifties under <em>~/.local/share/instructlab/taxonomy/knowledge/arts/fandom</em> and copy the files from step 1 into
this location.</p>

</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ mkdir -p ~/.local/share/instructlab/taxonomy/knowledge/arts/fandom/swifties
$ cp &lt;path_to&gt;/attribution.txt ~/.local/share/instructlab/taxonomy/knowledge/arts/fandom/swifties
$ cp &lt;path_to&gt;/qna.yaml ~/.local/share/instructlab/taxonomy/knowledge/arts/fandom/swifties</code></pre></div>

<ol start="3">
<li>With the Swifties taxonomy in place, check for any syntax errors with the
command <em>ilab taxonomy diff</em>. It should report that the taxonomy is valid if
there are no syntax errors.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ ilab taxonomy diff
knowledge/arts/fandom/swifties/qna.yaml
Taxonomy in /u/gsamu/.local/share/instructlab/taxonomy is valid :)</code></pre></div>

<ol start="4">
<li>With the taxonomy in place and having confirmed that the syntax is valid,
it’s now time to run the synthetic data generation job through LSF. Here we will
request 8 GPUs on a single server in exclusive execution mode. For the
InstructLab ilab command, specify the <em>&ndash;gpus 8 and &ndash;pipeline full</em> options.
Standard output is written to the $HOME/job-output with filename specification
&lt;LSF_JOBID&gt;.out. The $HOME/job-output directory must already exist.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ mkdir -p $HOME/job-output
$ bsub -o $HOME/job-output/%J.out -R "span[hosts=1]" -gpu "num=8:j_exclusive=yes" ilab data generate --pipeline full --gpus 8
Job &lt;1131&gt; is submitted to default queue &lt;normal&gt;.</code></pre></div>

<ol start="5">
<li>During job execution, the LSF <em>bpeek</em> command can be used to monitor the job
standard output.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ bpeek -f 1131 
&lt;&lt; output from stdout &gt;&gt;
INFO 2025-01-02 09:51:29,503 numexpr.utils:146: Note: detected 96 virtual cores but NumExpr set to maximum of 64, check "NUMEXPR_MAX_THREADS" environment variable.
INFO 2025-01-02 09:51:29,504 numexpr.utils:149: Note: NumExpr detected 96 cores but "NUMEXPR_MAX_THREADS" not set, so enforcing safe limit of 16.
INFO 2025-01-02 09:51:29,504 numexpr.utils:162: NumExpr defaulting to 16 threads.
INFO 2025-01-02 09:51:30,038 datasets:59: PyTorch version 2.3.1 available.
INFO 2025-01-02 09:51:31,226 instructlab.model.backends.llama_cpp💯 Trying to connect to model server at http://127.0.0.1:8000/v1
WARNING 2025-01-02 09:51:56,356 instructlab.data.generate:270: Disabling SDG batching - unsupported with llama.cpp serving
Generating synthetic data using 'full' pipeline, '/u/gsamu/.cache/instructlab/models/mistral-7b-instruct-v0.2.Q4_K_M.gguf' model, '/u/gsamu/.local/share/instructlab/taxonomy' taxonomy, against http://127.0.0.1:55779/v1 server
INFO 2025-01-02 09:51:56,861 instructlab.sdg.generate_data:356: Synthesizing new instructions. If you aren't satisfied with the generated instructions, interrupt training (Ctrl-C) and try adjusting your YAML files. Adding more examples may help.
INFO 2025-01-02 09:51:56,872 instructlab.sdg.pipeline:153: Running pipeline single-threaded
INFO 2025-01-02 09:51:56,872 instructlab.sdg.pipeline:197: Running block: duplicate_document_col
INFO 2025-01-02 09:51:56,872 instructlab.sdg.pipeline:198: Dataset({
    features: ['icl_document', 'document', 'document_outline', 'domain', 'icl_query_1', 'icl_query_2', 'icl_query_3', 'icl_response_1', 'icl_response_2', 'icl_response_3'],
    num_rows: 35
})
INFO 2025-01-02 09:51:58,286 instructlab.sdg.llmblock:51: LLM server supports batched inputs: False
INFO 2025-01-02 09:51:58,286 instructlab.sdg.pipeline:197: Running block: gen_spellcheck
INFO 2025-01-02 09:51:58,286 instructlab.sdg.pipeline:198: Dataset({
    features: ['icl_document', 'document', 'document_outline', 'domain', 'icl_query_1', 'icl_query_2', 'icl_query_3', 'icl_response_1', 'icl_response_2', 'icl_response_3', 'base_document'],
    num_rows: 35
})
/u/gsamu/miniforge3/envs/my_env/lib/python3.11/site-packages/llama_cpp/llama.py:1054: RuntimeWarning: Detected duplicate leading "&lt;s&gt;" in prompt, this will likely reduce response quality, consider removing it...
  warnings.warn(
INFO 2025-01-02 09:57:42,264 instructlab.sdg.pipeline:197: Running block: flatten_auxiliary_columns
INFO 2025-01-02 09:57:42,264 instructlab.sdg.pipeline:198: Dataset({
    features: ['icl_document', 'document', 'document_outline', 'domain', 'icl_query_1', 'icl_query_2', 'icl_query_3', 'icl_response_1', 'icl_response_2', 'icl_response_3', 'base_document', 'spellcheck'],
    num_rows: 35
})
INFO 2025-01-02 09:57:42,279 instructlab.sdg.pipeline:197: Running block: rename_to_document_column
INFO 2025-01-02 09:57:42,279 instructlab.sdg.pipeline:198: Dataset({
    features: ['icl_document', 'document', 'document_outline', 'domain', 'icl_query_1', 'icl_query_2', 'icl_query_3', 'icl_response_1', 'icl_response_2', 'icl_response_3', 'dataset_type', 'corrected_document'],
    num_rows: 70
})
INFO 2025-01-02 09:57:42,282 instructlab.sdg.pipeline:197: Running block: gen_knowledge
INFO 2025-01-02 09:57:42,282 instructlab.sdg.pipeline:198: Dataset({
    features: ['icl_document', 'raw_document', 'document_outline', 'domain', 'icl_query_1', 'icl_query_2', 'icl_query_3', 'icl_response_1', 'icl_response_2', 'icl_response_3', 'dataset_type', 'document'],
    num_rows: 70
})
…
…</code></pre></div>

<ol start="6">
<li>During the runtime of the job, it’s possible to view GPU related metrics
using the LSF <em>lsload</em> and <em>bhosts</em> commands. First, we need to identify the host
where the job has been dispatched to using the LSF bjobs command. In this case
the job was dispatched to host <em>p1-r01-n4</em>. Note that details GPU accounting
metrics are available once the job runs to completion.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ bjobs -w
JOBID   USER    STAT  QUEUE      FROM_HOST   EXEC_HOST   JOB_NAME   SUBMIT_TIME
1131    gsamu   RUN   normal     rmf-login-1 p1-r01-n4   ilab data generate --pipeline full --gpus 8 Jan  2 14:51
$ lsload -w -gpu p1-r01-n4
HOST_NAME                 status ngpus gpu_shared_avg_mut gpu_shared_avg_ut ngpus_physical
p1-r01-n4                     ok     8                 2%                7%              8
$ bhosts -w -gpu p1-r01-n4
HOST_NAME            GPU_ID                MODEL     MUSED      MRSV  NJOBS    RUN   SUSP    RSV 
p1-r01-n4                 0   NVIDIAH10080GBHBM3        2G        0G      1      1      0      0
                          1   NVIDIAH10080GBHBM3        2G        0G      1      1      0      0
                          2   NVIDIAH10080GBHBM3        2G        0G      1      1      0      0
                          3   NVIDIAH10080GBHBM3        2G        0G      1      1      0      0
                          4   NVIDIAH10080GBHBM3        2G        0G      1      1      0      0
                          5   NVIDIAH10080GBHBM3        2G        0G      1      1      0      0
                          6   NVIDIAH10080GBHBM3        2G        0G      1      1      0      0
                          7   NVIDIAH10080GBHBM3        2G        0G      1      1      0      0</code></pre></div>

<ol start="7">
<li>After job completion, it’s possible to view details about the job including
GPU utilization which LSF collects by leveraging NVIDIA DCGM. These metrics are
available upon job completion using both the LSF <em>bhist</em> and <em>bjobs</em> commands.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ bhist -l -gpu 1131

Job &lt;1131&gt;, User &lt;gsamu&gt;, Project &lt;default&gt;, Command &lt;ilab data generate --pipe
                          line full --gpus 8&gt;
Thu Jan  2 14:51:23 2025: Submitted from host &lt;rmf-login-1&gt;, to Queue &lt;normal&gt;,
                           CWD &lt;$HOME&gt;, Output File &lt;/u/gsamu/job-output/%J.out
                          &gt;, Requested Resources &lt;span[hosts=1]&gt;, Requested GPU
                           &lt;num=8:j_exclusive=yes&gt;;
Thu Jan  2 14:51:24 2025: Dispatched 1 Task(s) on Host(s) &lt;p1-r01-n4&gt;, Allocate
                          d 1 Slot(s) on Host(s) &lt;p1-r01-n4&gt;, Effective RES_REQ
                           &lt;select[((ngpus&gt;0)) &amp;&amp; (type == local)] order[r15s:p
                          g] rusage[ngpus_physical=8.00] span[hosts=1] &gt;;
Thu Jan  2 14:51:25 2025: Starting (Pid 3095851);
Thu Jan  2 14:51:25 2025: External Message "p1-r01-n4:gpus=0,1,2,3,4,5,6,7;EFFE
                          CTIVE GPU REQ: num=8:mode=shared:mps=no:j_exclusive=y
                          es:gvendor=nvidia;" was posted from "gsamu" to messag
                          e box 0;
Thu Jan  2 14:51:26 2025: Running with execution home &lt;/u/gsamu&gt;, Execution CWD
                           &lt;/u/gsamu&gt;, Execution Pid &lt;3095851&gt;;
Thu Jan  2 16:08:05 2025: Done successfully. The CPU time used is 4624.0 second
                          s;
                          HOST: p1-r01-n4; CPU_TIME: 4624 seconds              
                                          GPU ID: 0
                                  Total Execution Time: 4597 seconds
                                  Energy Consumed: 579704 Joules
                                  SM Utilization (%): Avg 9, Max 15, Min 0
                                  Memory Utilization (%): Avg 2, Max 100, Min 0
                                  Max GPU Memory Used: 1956642816 bytes

                              GPU ID: 1
                                  Total Execution Time: 4597 seconds
                                  Energy Consumed: 503956 Joules
                                  SM Utilization (%): Avg 7, Max 11, Min 0
                                  Memory Utilization (%): Avg 2, Max 5, Min 0
                                  Max GPU Memory Used: 1767899136 bytes

                              GPU ID: 2
                                  Total Execution Time: 4597 seconds
                                  Energy Consumed: 501754 Joules
                                  SM Utilization (%): Avg 7, Max 11, Min 0
                                  Memory Utilization (%): Avg 2, Max 5, Min 0
                                  Max GPU Memory Used: 1784676352 bytes

                              GPU ID: 3
                                  Total Execution Time: 4597 seconds
                                  Energy Consumed: 525195 Joules
                                  SM Utilization (%): Avg 7, Max 11, Min 0
                                  Memory Utilization (%): Avg 2, Max 54, Min 0
                                  Max GPU Memory Used: 1767899136 bytes

                              GPU ID: 4
                                  Total Execution Time: 4597 seconds
                                  Energy Consumed: 525331 Joules
                                  SM Utilization (%): Avg 7, Max 12, Min 0
                                  Memory Utilization (%): Avg 2, Max 5, Min 0
                                  Max GPU Memory Used: 1767899136 bytes

                              GPU ID: 5
                                  Total Execution Time: 4597 seconds
                                  Energy Consumed: 502416 Joules
                                  SM Utilization (%): Avg 7, Max 11, Min 0
                                  Memory Utilization (%): Avg 2, Max 5, Min 0
                                  Max GPU Memory Used: 1784676352 bytes

                              GPU ID: 6
                                  Total Execution Time: 4597 seconds
                                  Energy Consumed: 508720 Joules
                                  SM Utilization (%): Avg 7, Max 12, Min 0
                                  Memory Utilization (%): Avg 2, Max 5, Min 0
                                  Max GPU Memory Used: 1784676352 bytes

                              GPU ID: 7
                                  Total Execution Time: 4597 seconds
                                  Energy Consumed: 491041 Joules
                                  SM Utilization (%): Avg 6, Max 12, Min 0
                                  Memory Utilization (%): Avg 2, Max 4, Min 0
                                  Max GPU Memory Used: 1933574144 bytes

GPU Energy Consumed: 4138117.000000 Joules

Thu Jan  2 16:08:05 2025: Post job process done successfully;


GPU_ALLOCATION:
 HOST             TASK GPU_ID  GI_PLACEMENT/SIZE    CI_PLACEMENT/SIZE    MODEL        MTOTAL  FACTOR MRSV    SOCKET NVLINK/XGMI                      
 p1-r01-n4        0    0       -                    -                    NVIDIAH10080 80G     9.0    0G      0      -                               
                  0    1       -                    -                    NVIDIAH10080 80G     9.0    0G      0      -                               
                  0    2       -                    -                    NVIDIAH10080 80G     9.0    0G      0      -                               
                  0    3       -                    -                    NVIDIAH10080 80G     9.0    0G      0      -                               
                  0    4       -                    -                    NVIDIAH10080 80G     9.0    0G      1      -                               
                  0    5       -                    -                    NVIDIAH10080 80G     9.0    0G      1      -                               
                  0    6       -                    -                    NVIDIAH10080 80G     9.0    0G      1      -                               
                  0    7       -                    -                    NVIDIAH10080 80G     9.0    0G      1      -                               

MEMORY USAGE:
MAX MEM: 2 Gbytes;  AVG MEM: 1 Gbytes; MEM Efficiency: 0.00%

CPU USAGE:
CPU PEAK: 1.69 ;  CPU PEAK DURATION: 52 second(s)
CPU AVERAGE EFFICIENCY: 100.69% ;  CPU PEAK EFFICIENCY: 169.23%

Summary of time in seconds spent in various states by  Thu Jan  2 16:08:05 2025
  PEND     PSUSP    RUN      USUSP    SSUSP    UNKWN    TOTAL
  1        0        4601     0        0        0        4602 </code></pre></div>

<ol start="8">
<li>
<p>When the synthetic data generation job completes, it’s output can be viewed
at <em>~/job-output/<!-- raw HTML omitted -->.out</em>. The synthetic data sets will comprise files in
the directory <em>~/.local/share/instructlab/datasets</em>. These files will be named
*skills_train_msgs_*.jsonl* and *knowledge_train_msgs_*.jsonl*.</p>

</li>
<li>
<p>With the synthetic data generation step complete, it’s now time to run the
training. We first set 2 environment variables to point to the following
files:  <em>~/.local/share/instructlab/datasets/knowledge_train_msgs_2025-01-02T09_51_56.jsonl</em>  and <em>~./.local/share/instructlab/datasets/skills_train_msgs_2025-01-02T09_51_56.jsonl</em>.</p>

</li>
</ol>
<p>Afterward, we submit the training job to LSF requesting 8 GPUs and with ilab
options <em>&ndash;pipeline accelerated</em>, <em>&ndash;gpus 8</em>, <em>&ndash;device cuda</em> and
<em>&ndash;data-path</em> pointing to the two above data files that were produced in the
synthetic data generation step.</p>

<div class="highlight"><pre><code class="language-plaintext">$ export SKILLS_PATH=/u/gsamu/.local/share/instructlab/datasets/skills_train_msgs_2025-01-02T09_51_56.jsonl
$ export KNOWLEDGE_PATH=/u/gsamu/.local/share/instructlab/datasets/knowledge_train_msgs_2025-01-02T09_51_56.jsonl
$ bsub -o $HOME/job-output/%J.out -R "span[hosts=1]" -gpu "num=8:j_exclusive=yes" ilab model train --pipeline accelerated --data-path $SKILLS_PATH --data-path $KNOWLEDGE_PATH --device cuda --gpus 8
Job &lt;1135&gt; is submitted to default queue &lt;normal&gt;.</code></pre></div>

<ol start="10">
<li>During job execution, the LSF <em>bpeek</em> command can be used to monitor the
job standard output.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ bpeek -f 1135
&lt;&lt; output from stdout &gt;&gt;
LoRA is disabled (rank=0), ignoring all additional LoRA args
[2025-01-02 12:52:04,359] [INFO] [real_accelerator.py:222:get_accelerator] Setting ds_accelerator to cuda (auto detect)
INFO 2025-01-02 12:52:09,061 numexpr.utils:146: Note: detected 96 virtual cores but NumExpr set to maximum of 64, check "NUMEXPR_MAX_THREADS" environment variable.
INFO 2025-01-02 12:52:09,061 numexpr.utils:149: Note: NumExpr detected 96 cores but "NUMEXPR_MAX_THREADS" not set, so enforcing safe limit of 16.
INFO 2025-01-02 12:52:09,061 numexpr.utils:162: NumExpr defaulting to 16 threads.
INFO 2025-01-02 12:52:09,304 datasets:59: PyTorch version 2.3.1 available.
You are using the default legacy behaviour of the &lt;class 'transformers.models.llama.tokenization_llama_fast.LlamaTokenizerFast'&gt;. This is expected, and simply means that the `legacy` (previous) behavior will be used so nothing changes for you. If you want to use the new behaviour, set `legacy=False`. This should only be set if you understand what it means, and thoroughly read the reason why this was added as explained in https://github.com/huggingface/transformers/pull/24565 - if you loaded a llama tokenizer from a GGUF file you can ignore this message.
INFO 2025-01-02 12:52:09,653 root:617: Special tokens: eos: [32000], pad: [32001], bos: [32005], system: [32004], user: [32002], assistant: [32003]
INFO 2025-01-02 12:52:09,923 root:617: number of dropped samples: 0 -- out of 641
 data arguments are:
{"data_path":"/u/gsamu/.local/share/instructlab/datasets/knowledge_train_msgs_2025-01-02T09_51_56.jsonl","data_output_path":"/u/gsamu/.local/share/instructlab/internal","max_seq_len":4096,"model_path":"/u/gsamu/.cache/instructlab/models/instructlab/granite-7b-lab","chat_tmpl_path":"/u/gsamu/miniforge3/envs/my_env/lib/python3.11/site-packages/instructlab/training/chat_templates/ibm_generic_tmpl.py","num_cpu_procs":16}
tokenizing the dataset with /u/gsamu/.cache/instructlab/models/instructlab/granite-7b-lab tokenizer...
ten largest length percentiles:
quantile 90th: 1459.0
quantile 91th: 1466.0
quantile 92th: 1469.6000000000001
quantile 93th: 1478.2
quantile 94th: 1483.0
quantile 95th: 1488.0
quantile 96th: 1497.1999999999998
quantile 97th: 1516.5999999999997
quantile 98th: 1540.6000000000001
quantile 99th: 1656.0000000000016
quantile 100th: 2578.0

at 4096 max sequence length, the number of samples to be dropped is 0
(0.00% of total)
quantile 0th: 368.0
quantile 1th: 393.0
quantile 2th: 411.2
quantile 3th: 421.2
quantile 4th: 427.2
quantile 5th: 442.0
quantile 6th: 604.4
quantile 7th: 631.8
quantile 8th: 653.8000000000001
quantile 9th: 679.8
quantile 10th: 742.0
at 20 min sequence length, the number of samples to be dropped is 0
checking the validity of the samples...
Categorizing training data type...
unmasking the appropriate message content...
 Samples Previews...
…
…</code></pre></div>

<ol start="11">
<li>During the runtime of the training job, we can observe some GPU utilization
information using the LSF lsload and bhosts commands.  First we need to identify
the server on which the training job is running. This is done using the bjobs
command and checking for the execution host of the job.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ bjobs -w
JOBID   USER    STAT  QUEUE      FROM_HOST   EXEC_HOST   JOB_NAME   SUBMIT_TIME
1135    gsamu   RUN   normal     rmf-login-1 p1-r01-n1   ilab model train --pipeline accelerated --data-path /u/gsamu/.local/share/instructlab/datasets/skills_train_msgs_2025-01-02T09_51_56.jsonl --data-path /u/gsamu/.local/share/instructlab/datasets/knowledge_train_msgs_2025-01-02T09_51_56.jsonl --device cuda --gpus 8 Jan  2 17:51
$ lsload -w -gpu p1-r01-n1
HOST_NAME                 status ngpus gpu_shared_avg_mut gpu_shared_avg_ut ngpus_physical
p1-r01-n1                     ok     8                 0%               22%              8
$ bhosts -w -gpu p1-r01-n1
HOST_NAME            GPU_ID                MODEL     MUSED      MRSV  NJOBS    RUN   SUSP    RSV 
p1-r01-n1                 0   NVIDIAH10080GBHBM3       10G        0G      1      1      0      0
                          1   NVIDIAH10080GBHBM3       10G        0G      1      1      0      0
                          2   NVIDIAH10080GBHBM3       10G        0G      1      1      0      0
                          3   NVIDIAH10080GBHBM3       10G        0G      1      1      0      0
                          4   NVIDIAH10080GBHBM3       10G        0G      1      1      0      0
                          5   NVIDIAH10080GBHBM3       10G        0G      1      1      0      0
                          6   NVIDIAH10080GBHBM3       10G        0G      1      1      0      0
                          7   NVIDIAH10080GBHBM3       10G        0G      1      1      0      0</code></pre></div>

<ol start="12">
<li>Once the job is complete, detailed GPU accounting can again be viewed using
the LSF <em>bhist</em> command as follows below.</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ bhist -l -gpu 1135

Job &lt;1135&gt;, User &lt;gsamu&gt;, Project &lt;default&gt;, Command &lt;ilab model train --pipeli
                          ne accelerated --data-path /u/gsamu/.local/share/inst
                          ructlab/datasets/skills_train_msgs_2025-01-02T09_51_5
                          6.jsonl --data-path /u/gsamu/.local/share/instructlab
                          /datasets/knowledge_train_msgs_2025-01-02T09_51_56.js
                          onl --device cuda --gpus 8&gt;
Thu Jan  2 17:51:48 2025: Submitted from host &lt;rmf-login-1&gt;, to Queue &lt;normal&gt;,
                           CWD &lt;$HOME/.local/share/instructlab/checkpoints&gt;, Ou
                          tput File &lt;/u/gsamu/job-output/%J.out&gt;, Requested Res
                          ources &lt;span[hosts=1]&gt;, Requested GPU &lt;num=8:j_exclus
                          ive=yes&gt;;
Thu Jan  2 17:51:48 2025: Dispatched 1 Task(s) on Host(s) &lt;p1-r01-n1&gt;, Allocate
                          d 1 Slot(s) on Host(s) &lt;p1-r01-n1&gt;, Effective RES_REQ
                           &lt;select[((ngpus&gt;0)) &amp;&amp; (type == local)] order[r15s:p
                          g] rusage[ngpus_physical=8.00] span[hosts=1] &gt;;
Thu Jan  2 17:51:49 2025: Starting (Pid 3462241);
Thu Jan  2 17:51:49 2025: Running with execution home &lt;/u/gsamu&gt;, Execution CWD
                           &lt;/u/gsamu/.local/share/instructlab/checkpoints&gt;, Exe
                          cution Pid &lt;3462241&gt;;
Thu Jan  2 17:51:49 2025: External Message "p1-r01-n1:gpus=0,1,2,3,4,5,6,7;EFFE
                          CTIVE GPU REQ: num=8:mode=shared:mps=no:j_exclusive=y
                          es:gvendor=nvidia;" was posted from "gsamu" to messag
                          e box 0;
Thu Jan  2 17:57:56 2025: Done successfully. The CPU time used is 3024.0 second
                          s;
                          HOST: p1-r01-n1; CPU_TIME: 3024 seconds              
                                          GPU ID: 0
                                  Total Execution Time: 365 seconds
                                  Energy Consumed: 98890 Joules
                                  SM Utilization (%): Avg 20, Max 100, Min 0
                                  Memory Utilization (%): Avg 9, Max 62, Min 0
                                  Max GPU Memory Used: 53022294016 bytes

                              GPU ID: 1
                                  Total Execution Time: 365 seconds
                                  Energy Consumed: 97697 Joules
                                  SM Utilization (%): Avg 53, Max 100, Min 0
                                  Memory Utilization (%): Avg 9, Max 58, Min 0
                                  Max GPU Memory Used: 53087305728 bytes

                              GPU ID: 2
                                  Total Execution Time: 365 seconds
                                  Energy Consumed: 94820 Joules
                                  SM Utilization (%): Avg 53, Max 100, Min 0
                                  Memory Utilization (%): Avg 9, Max 62, Min 0
                                  Max GPU Memory Used: 53221523456 bytes

                              GPU ID: 3
                                  Total Execution Time: 365 seconds
                                  Energy Consumed: 98014 Joules
                                  SM Utilization (%): Avg 53, Max 100, Min 0
                                  Memory Utilization (%): Avg 9, Max 59, Min 0
                                  Max GPU Memory Used: 53041168384 bytes

                              GPU ID: 4
                                  Total Execution Time: 365 seconds
                                  Energy Consumed: 99246 Joules
                                  SM Utilization (%): Avg 53, Max 100, Min 0
                                  Memory Utilization (%): Avg 9, Max 60, Min 0
                                  Max GPU Memory Used: 53045362688 bytes

                              GPU ID: 5
                                  Total Execution Time: 365 seconds
                                  Energy Consumed: 94952 Joules
                                  SM Utilization (%): Avg 53, Max 100, Min 0
                                  Memory Utilization (%): Avg 9, Max 65, Min 0
                                  Max GPU Memory Used: 53047459840 bytes

                              GPU ID: 6
                                  Total Execution Time: 365 seconds
                                  Energy Consumed: 98227 Joules
                                  SM Utilization (%): Avg 53, Max 100, Min 0
                                  Memory Utilization (%): Avg 9, Max 63, Min 0
                                  Max GPU Memory Used: 53127151616 bytes

                              GPU ID: 7
                                  Total Execution Time: 365 seconds
                                  Energy Consumed: 94582 Joules
                                  SM Utilization (%): Avg 52, Max 100, Min 0
                                  Memory Utilization (%): Avg 9, Max 65, Min 0
                                  Max GPU Memory Used: 53481570304 bytes

GPU Energy Consumed: 776428.000000 Joules

Thu Jan  2 17:57:56 2025: Post job process done successfully;


GPU_ALLOCATION:
 HOST             TASK GPU_ID  GI_PLACEMENT/SIZE    CI_PLACEMENT/SIZE    MODEL        MTOTAL  FACTOR MRSV    SOCKET NVLINK/XGMI                      
 p1-r01-n1        0    0       -                    -                    NVIDIAH10080 80G     9.0    0G      0      -                               
                  0    1       -                    -                    NVIDIAH10080 80G     9.0    0G      0      -                               
                  0    2       -                    -                    NVIDIAH10080 80G     9.0    0G      0      -                               
                  0    3       -                    -                    NVIDIAH10080 80G     9.0    0G      0      -                               
                  0    4       -                    -                    NVIDIAH10080 80G     9.0    0G      1      -                               
                  0    5       -                    -                    NVIDIAH10080 80G     9.0    0G      1      -                               
                  0    6       -                    -                    NVIDIAH10080 80G     9.0    0G      1      -                               
                  0    7       -                    -                    NVIDIAH10080 80G     9.0    0G      1      -                               

MEMORY USAGE:
MAX MEM: 104 Gbytes;  AVG MEM: 16 Gbytes; MEM Efficiency: 0.00%

CPU USAGE:
CPU PEAK: 17.86 ;  CPU PEAK DURATION: 49 second(s)
CPU AVERAGE EFFICIENCY: 856.60% ;  CPU PEAK EFFICIENCY: 1785.71%

Summary of time in seconds spent in various states by  Thu Jan  2 17:57:56 2025
  PEND     PSUSP    RUN      USUSP    SSUSP    UNKWN    TOTAL
  0        0        368      0        0        0        368         </code></pre></div>

<ol start="13">
<li>Finally, with the model successfully trained, let’s chat with the new model
to check the result. Here’s we’ll pose it Swiftie specific questions. Note that
the output from the training is written to <em>~/.local/share/instructlab/checkpoints/hf_format</em>. We’ll take the model from the latest checkpoint directory that was
created. Here again, we launch the model chat job via LSF as an interactive
batch job (i.e. <em>bsub -Is</em>).</li>
</ol>
<div class="highlight"><pre><code class="language-plaintext">$ grep hf_format 1135.out
Model saved in /u/gsamu/.local/share/instructlab/checkpoints/hf_format/samples_886
Model saved in /u/gsamu/.local/share/instructlab/checkpoints/hf_format/samples_1776
Model saved in /u/gsamu/.local/share/instructlab/checkpoints/hf_format/samples_2658
Model saved in /u/gsamu/.local/share/instructlab/checkpoints/hf_format/samples_3546
Model saved in /u/gsamu/.local/share/instructlab/checkpoints/hf_format/samples_4435</code></pre></div>

<div class="highlight"><pre><code class="language-plaintext">$ bsub -Is -R "span[hosts=1]" -gpu "num=8:j_exclusive=yes" ilab model chat --model /u/gsamu/.local/share/instructlab/checkpoints/hf_format/samples_4435
Job &lt;1146&gt; is submitted to default queue &lt;interactive&gt;.
&lt;&lt;Waiting for dispatch ...&gt;&gt;
&lt;&lt;Starting on p1-r01-n2&gt;&gt;
INFO 2025-01-02 15:06:07,600 instructlab.model.backends.vllm:105: Trying to connect to model server at http://127.0.0.1:8000/v1
INFO 2025-01-02 15:06:08,876 instructlab.model.backends.vllm:308: vLLM starting up on pid 3744375 at http://127.0.0.1:41531/v1
INFO 2025-01-02 15:06:08,876 instructlab.model.backends.vllm:114: Starting a temporary vLLM server at http://127.0.0.1:41531/v1
INFO 2025-01-02 15:06:08,876 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 1/120
INFO 2025-01-02 15:06:12,244 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 2/120
INFO 2025-01-02 15:06:15,614 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 3/120
INFO 2025-01-02 15:06:18,801 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 4/120
INFO 2025-01-02 15:06:21,952 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 5/120
INFO 2025-01-02 15:06:25,391 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 6/120
INFO 2025-01-02 15:06:28,638 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 7/120
INFO 2025-01-02 15:06:32,103 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 8/120
INFO 2025-01-02 15:06:35,296 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 9/120
INFO 2025-01-02 15:06:38,616 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 10/120
INFO 2025-01-02 15:06:42,015 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 11/120
INFO 2025-01-02 15:06:45,435 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 12/120
INFO 2025-01-02 15:06:48,679 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 13/120
INFO 2025-01-02 15:06:52,025 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 14/120
INFO 2025-01-02 15:06:55,317 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 15/120
INFO 2025-01-02 15:06:58,604 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 16/120
INFO 2025-01-02 15:07:01,927 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 17/120
INFO 2025-01-02 15:07:05,287 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 18/120
INFO 2025-01-02 15:07:08,763 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 19/120
INFO 2025-01-02 15:07:12,131 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 20/120
INFO 2025-01-02 15:07:15,476 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 21/120
INFO 2025-01-02 15:07:18,881 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 22/120
INFO 2025-01-02 15:07:22,203 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 23/120
INFO 2025-01-02 15:07:25,599 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 24/120
INFO 2025-01-02 15:07:28,991 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 25/120
INFO 2025-01-02 15:07:32,234 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 26/120
INFO 2025-01-02 15:07:35,714 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 27/120
INFO 2025-01-02 15:07:38,974 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 28/120
INFO 2025-01-02 15:07:42,265 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 29/120
INFO 2025-01-02 15:07:45,582 instructlab.model.backends.vllm:129: Waiting for the vLLM server to start at http://127.0.0.1:41531/v1, this might take a moment... Attempt: 30/120
INFO 2025-01-02 15:07:45,586 instructlab.model.backends.vllm:136: vLLM engine successfully started at http://127.0.0.1:41531/v1
╭────────────────────────────────────────────────────────────── system ──────────────────────────────────────────────────────────────╮
│ Welcome to InstructLab Chat w/ SAMPLES_4435 (type /h for help)                                                                     │
╰────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
&gt;&gt;&gt; Tell me everything you know about Swifties.                                                                           [S][default]
╭─────────────────────────────────────────────────────────── samples_4435 ───────────────────────────────────────────────────────────╮
│ Swifties are the fandom of the American singer-songwriter Taylor Swift.                                                            │
│ Regarded by journalists as one of the largest, most devoted, and influential fan bases, Swifties are known for their high levels   │
│ of participation, creativity, community, fanaticism, and cultural impact on the music industry and popular culture. They are a     │
│ subject of widespread coverage in the mainstream media.                                                                            │
│                                                                                                                                    │
│ Critics have opined that Swift has redefined artist-fan relationships by establishing an intimate connection with Swifties. She    │
│ has frequently engaged with, helped, credited, and prioritized her fans, who have offered unprecedented support and interest in    │
│ her works irrespective of her wavering reception in the media. They continued to support Swift through her genre transitions,      │
│ unexpected artistic pivots, and her highly publicized controversies such as the 2019 masters dispute, while instigating the        │
│ political scrutiny of Ticketmaster that led to implementation of various laws and stimulated economic growth with the Eras Tour.   │
│ Swift's releases, promotional efforts, and fashion have garnered attention for incorporating Easter eggs and clues that are        │
...
...</code></pre></div>

<p><strong>Conclusions</strong></p>

<p>We’ve demonstrated a simple InstructLab workflow that is scheduled by IBM LSF
in a compute cluster equipped with GPUs.  As part of this example, LSF GPU
scheduling and accounting for GPU workloads was highlighted. For organizations
looking to productionize InstructLab and where there is a pool of GPU equipped
compute resources, LSF provides an ideal way to manage demand from a user
community looking to run these intensive workloads.</p>

<p>At the recent SC24 event, the demonstration went beyond what is shown in this
blog. It incorporated single click job submission via LSF Application Center
using a custom template that was created for InstructLab to submits both the
synthetic data generation job, as well the training job with a single click.
The demo environment was on IBM Cloud using instances equipped with Nvidia GPUs.
The compute instances were automatically scaled up and down by the LSF resource
connector. This will be the topic for a future blog.</p>