.. _nextflow_3-page:

*******************
3 Nextflow
*******************

Decoupling resources, parameters and Nextflow script
=======================

When making complex pipelines it is convenient to keep the definition of resources needed, the default parameters, and the main script separately from each other.
This can be achieved using two additional files:

- nextflow.config
- params.config

The **nextflow.config** file allows to indicate resources needed for each class of processes.
This is achieved by labeling processes in the nextflow.config file:

.. literalinclude:: ../nextflow/test2/nextflow.config
   :language: groovy
   :lines: 3-17

The first part defines the "default" resources for a process:

.. literalinclude:: ../nextflow/test2/nextflow.config
   :language: groovy
   :lines: 3-17
   :emphasize-lines: 1-4


Then are specified the resources needed for a class of processes labeled **bigmem**. In brief, the default options will be overridden for the processes labeled **bigmem** and **onecpu**:

.. literalinclude:: ../nextflow/test2/nextflow.config
   :language: groovy
   :lines: 3-17
   :emphasize-lines: 6-14

.. tip::
	You can add the default configuration for shell executions within to the nextflow.config file:
.. code-block:: groovy

	process {
  		shell = ['/bin/bash', '-euo', 'pipefail']
		...

In the script **/test2/test2.nf file**, there are two processes to run two programs:

- `fastQC <https://www.bioinformatics.babraham.ac.uk/projects/fastqc/>`__ - a tool that calculates a number of quality control metrics on single fastq files;
- `multiQC <https://multiqc.info/>`__ - an aggregator of results from bioinformatics tools and samples for generating a single html report.


.. literalinclude:: ../nextflow/test2/test2.nf
   :language: groovy
   :emphasize-lines: 66


You can see that the process **fastQC** is labeled 'bigmem'.


The last two rows of the config file indicate which containers to use.
In this example, -- and by default, if the repository is not specified, -- a container is pulled from the DockerHub.
In the case of using a singularity container, you can indicate where to store the local image using the **singularity.cacheDir** option:

.. code-block:: groovy

	process.container = 'biocorecrg/c4lwg-2018:latest'
	singularity.cacheDir = "$baseDir/singularity"


Let's now launch the script **test2.nf**.

.. code-block:: console
   :emphasize-lines: 42,43

	cd test2;
	nextflow run test2.nf

	N E X T F L O W  ~  version 20.07.1
	Launching `test2.nf` [distracted_edison] - revision: e3a80b15a2
	BIOCORE@CRG - N F TESTPIPE  ~  version 1.0
	=============================================
	reads                           : /home/ec2-user/git/CRG_Nextflow_Jun_2022/nextflow/nextflow/test2/../testdata/*.fastq.gz
	executor >  local (2)
	[df/2c45f2] process > fastQC (B7_input_s_chr19.fastq.gz) [  0%] 0 of 2
	[-        ] process > multiQC                            -
	Error executing process > 'fastQC (B7_H3K4me1_s_chr19.fastq.gz)'

	Caused by:
	  Process `fastQC (B7_H3K4me1_s_chr19.fastq.gz)` terminated with an error exit status (127)

	Command executed:

	  fastqc B7_H3K4me1_s_chr19.fastq.gz

	Command exit status:
	  127

	executor >  local (2)
	[df/2c45f2] process > fastQC (B7_input_s_chr19.fastq.gz) [100%] 2 of 2, failed: 2 ✘
	[-        ] process > multiQC                            -
	Error executing process > 'fastQC (B7_H3K4me1_s_chr19.fastq.gz)'

	Caused by:
	  Process `fastQC (B7_H3K4me1_s_chr19.fastq.gz)` terminated with an error exit status (127)

	Command executed:

	  fastqc B7_H3K4me1_s_chr19.fastq.gz

	Command exit status:
	  127

	Command output:
	  (empty)

	Command error:
	  .command.sh: line 2: fastqc: command not found

	Work dir:
	  /home/ec2-user/git/CRG_Nextflow_Jun_2022/nextflow/nextflow/test2/work/c5/18e76b2e6ffd64aac2b52e69bedef3

	Tip: when you have fixed the problem you can continue the execution adding the option `-resume` to the run command line


We will get a number of errors since no executable is found in our environment/path. This is because the executables are stored in our docker image and we have to tell Nextflow to use the docker image, using the `-with-docker` parameter.


.. code-block:: console
   :emphasize-lines: 1

	nextflow run test2.nf -with-docker

	N E X T F L O W  ~  version 20.07.1
	Launching `test2.nf` [boring_hamilton] - revision: e3a80b15a2
	BIOCORE@CRG - N F TESTPIPE  ~  version 1.0
	=============================================
	reads                           : /home/ec2-user/git/CRG_Nextflow_Jun_2022/nextflow/nextflow/test2/../testdata/*.fastq.gz
	executor >  local (3)
	[22/b437be] process > fastQC (B7_H3K4me1_s_chr19.fastq.gz) [100%] 2 of 2 ✔
	[1a/cfe63b] process > multiQC                              [  0%] 0 of 1
	executor >  local (3)
	[22/b437be] process > fastQC (B7_H3K4me1_s_chr19.fastq.gz) [100%] 2 of 2 ✔
	[1a/cfe63b] process > multiQC                              [100%] 1 of 1 ✔


This time it worked because Nextflow used the image specified in the **nextflow.config** file and containing the executables.

Now let's take a look at the **params.config** file:

.. literalinclude:: ../nextflow/test2/params.config
   :language: groovy

As you can see, we indicated two pipeline parameters, `reads` and `email`; when running the pipeline, they can be overridden using `\-\-reads` and `\-\-email`.

This file is included thanks again to the **nextflow.config** file, here shown entirely

.. literalinclude:: ../nextflow/test2/nextflow.config
   :language: groovy
   :emphasize-lines: 1




Now, let's examine the folders generated by the pipeline.

.. code-block:: console

	ls  work/2a/22e3df887b1b5ac8af4f9cd0d88ac5/

	total 0
	drwxrwxr-x 3 ec2-user ec2-user  26 Apr 23 13:52 .
	drwxr-xr-x 2 root     root     136 Apr 23 13:51 multiqc_data
	drwxrwxr-x 3 ec2-user ec2-user  44 Apr 23 13:51 ..


We observe that Docker runs as "root". This can be problematic and generates security issues. To avoid this we can add this line of code within the process section of the config file:

.. code-block:: groovy

	containerOptions = { workflow.containerEngine == "docker" ? '-u $(id -u):$(id -g)': null}


This will tell Nextflow that if it is run with Docker, it has to produce files that belong to a user rather than the root.

Publishing final results
-------------------------

The script **test2.nf** generates two new folders, **output_fastqc** and **output_multiQC**, that contain the result of the pipeline output.
We can indicate which process and output can be considered the final output of the pipeline using the **publishDir** directive that has to be specified at the beginning of a process.

In our pipeline, we define these folders here:

.. literalinclude:: ../nextflow/test2/test2.nf
   :language: groovy
   :emphasize-lines: 42-46,64,84



You can see that the default mode to publish the results in Nextflow is `soft linking`. You can change this behavior by specifying the mode as indicated in the **multiQC** process.

.. note::
	IMPORTANT: You can also "move" the results but this is not suggested for files that will be needed for other processes. This will likely disrupt your pipeline


Adding help section to a pipeline
=============================================

Here we describe another good practice: the use of the `\-\-help` parameter. At the beginning of the pipeline, we can write:


.. literalinclude:: ../nextflow/test2/test2.nf
   :language: groovy
   :emphasize-lines: 25-40


so that launching the pipeline with `\-\-help` will show you just the parameters and the help.

.. code-block:: console

	nextflow run test2.nf --help

	N E X T F L O W  ~  version 20.07.1
	Launching `test2.nf` [mad_elion] - revision: e3a80b15a2
	BIOCORE@CRG - N F TESTPIPE  ~  version 1.0
	=============================================
	reads                           : /home/ec2-user/git/CRG_Nextflow_Jun_2022/nextflow/nextflow/test2/../testdata/*.fastq.gz
	This is the Biocore's NF test pipeline
	Enjoy!

EXERCISE
===============

- Look at the very last EXERCISE of the day before. Change the script and the config file using the label for handling failing processes.

.. raw:: html

   <details>
   <summary><a>Solution</a></summary>

The process should become:

.. literalinclude:: ../nextflow/test1/sol/sol_lab.nf
   :language: groovy
   :emphasize-lines: 38



and the nextflow.config file would become:

.. literalinclude:: ../nextflow/test1/sol/nextflow.config
   :language: groovy


.. raw:: html

   </details>
|
|

- Now look at **test2.nf**.
Change this script and the config file using the label for handling failing processes by retrying 3 times and incrementing time.

You can specify a very low time (5, 10 or 15 seconds) for the fastqc process so it would fail at the beginning.

.. raw:: html

   <details>
   <summary><a>Solution</a></summary>


The code should become:

.. literalinclude:: ../nextflow/test2/retry/retry.nf
   :language: groovy
   :emphasize-lines: 66



while the nextflow.config file would be:

.. literalinclude:: ../nextflow/test2/retry/nextflow.config
   :language: groovy
   
   
.. raw:: html

   </details>
|
|
