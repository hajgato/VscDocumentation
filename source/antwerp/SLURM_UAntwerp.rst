.. _Antwerp Slurm:

Slurm @ UAntwerp
================

**This is preliminary documentation for the pilot users. It will be further refined as the pilot progresses.**

What and why?
-------------

Since the start of the VSC, Torque and Moab were used as the resource manager and scheduler
respectively. The resource manager is responsible for keeping track of resources and making
sure jobs use the resources allocated to them. The scheduler is the piece of software that
prioritises jobs that are waiting in the queue and decides which job can start with which
resources. It is clear that both have to work together very closely. Torque and Moab were
developed and supported by Adaptive Computing. This company was acquired by ALA Services 
Technology Companies. Since then the software isn't well supported anymore, resulting in
problems to keep it running on our systems. 
Therefore the decision was taken to transfer to a different resource manager and scheduler
software. Slurm Workload Manager was chosen due to its wide use in academic supercomputer
centres. We've been preparing for this switch for over two years now by stressing in the
introductory courses those features of Torque and Moab that resemble Slurm features
the most.

Slurm Workload Manager is also used on the clusters at UGent (but with a wrapper that still
accepts Torque job scripts with some limitations) and will also be the scheduler on the
successor of the BrENIAC Tier-1 system.

Historically, Slurm was an acronym of **S**\imple **L**\inux **U**\tility for 
**R**\esource **M**\anagement. The development started around 2002 at Lawrence Livermore
National Lab as a resource manager for Linux cluster. Slurm has always had a very modular
architecture. From 2008 on increasingly sophisticated scheduling plugins were added
to Slurm. Nowadays it is used on some of the largest systems in the world. Slurm is
completely Open Source though commercial support can be obtained from SchedMD, a
spin-off company of the Slurm development.


Slurm concepts
--------------

* **Nodes**: The actual compute resources in Slurm.
* **Partition**: A job queue with limits and access controls, basically the equivalent
  of a queue in Torque.
* **Job**: A resource allocation request.
* **Job step**: A set of (possibly parallel) tasks within a job. A job can consist of
  just a single job step, or can contain multiple job steps which may use all or just
  a part of the resource allocation of a job and can run sequentially or in parallel
  (or a mix of that). The job script itself is a special job step, called the batch
  job step, but additional job
  steps can be created (e.g., for running a parallel MPI application).
* **Task**: A task is executed within a job step and essentially corresponds to a 
  Linux process: a single- or multithreaded process, or a single rank within a MPI
  process. Specifying the number of tasks one wants to run simultaneously and the 
  number of cores per task is a very convenient way to request resources to Slurm
  as afterwards starting a MPI or hybrid MPI/OpenMP program using the ``srun``
  command is very easy.
* **Core**: A core is a physical core in a system.
* **CPU**: A CPU is a virtual core in a system, in other words, a hardware thread
  in a system with hyperthreading/SMT enabled. On a system with hyperthreading/SMT
  disabled, virtual cores are just physical cores.


Slurm commands
--------------


Submitting a job script: sbatch
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`Slurm sbatch manual page on the web <https://slurm.schedmd.com/sbatch.html>`_ 

The ``sbatch`` is the command in Slurm to submit a job script.
A job script first contains a list of resources and other instructions to
Slurm, and this is followed by a set of commands that will be executed on the
first node of the job. At successful completion, ``sbatch`` will print a message
containing the unique jobid for the job.

Resource specifications and other instructions can be specified in three 
different ways: command line options, environment variables, and ``#SBATCH``
lines in the job script.

* Slurm ``sbatch`` has a lot of command line options. We will only list the
  most important command line options below. Command line options of Slurm
  take precedence over environment variables of ``#SBATCH`` lines in the
  job script.
* Some command line options can also be passed to ``sbatch`` through environment
  variables instead. A list of those can be found in the 
  `sbatch manual page <https://slurm.schedmd.com/sbatch.html>`_. The name of those
  variables starts with ``SBATCH_`` and the remaining part is derived from the
  matching command line option.
* All command line options can also be passed in ``#SBATCH`` lines in the job script.
  These lines should follow immediately below the shebang in the first block of 
  comment lines (lines that start with ``#``) as otherwise they will
  be ignored by Slurm.

Requesting compute resources
""""""""""""""""""""""""""""

Slurm supports several ways to request CPU cores and/or GPUs for a job. 

The easiest way to request CPU cores is by following the "task"-idea
of Slurm and specifying the number of parallel tasks and cores per task
that you need. By specifying resources this way, it is very easy afterwards
to start OpenMP, MPI and hybrid MPI/OpenMP programs in the right configuration.

* The number of tasks is specified by ``--ntasks=<number of tasks>`` or 
  ``-n <number of tasks>``. The ``=``-sign in the long option format can
  be omitted, and the space between the flag and the value in the short form
  (``-n``) can also be omitted.
* The number of CPUs (hardware threads) per task is then specified by
  ``--cpus-per-task=<value>`` or ``-c <value>``. The default size of a task
  is one node which if often not what is desired, so this flag is almost always
  needed.

Requesting memory
"""""""""""""""""

Slurm jobs can also request an amount of RAM space (resident memory). Swap space 
cannot be requested; the system uses a default which is function of the 
requested RAM space. In case of the UAntwerp clusters, swapping for jobs is
disabled since the nodes don't have drives suitable for the load caused by
swapping and since swapping is extremely detrimental to the performance of
the cluster.

Slurm has various ways to request memory. Unfortunately there is currently no
way to request memory per task. The preferred method for requesting memory in
Slurm on the UAntwerp clusters is to specify the amount of memory per CPU (per
hardware thread, which in the case of the UAntwerp clusters is per core):
``--mem-per-cpu=<amount><unit>``(e.g., ``--mem-per-cpu=1g``). The amount is an
integer, ``<unit>`` can be either ``k`` for kilobytes, ``m`` for megabyte or
``g`` for gigabyte.

Requesting wall time
""""""""""""""""""""

The requested compute time is specified using ``--time=<time>`` or ``-t <time>``.
``<time>`` is specified in mm (minutes), mm\:ss (minutes and seconds), hh\:mm\:ss
(hours, minutes and seconds), d-hh (days and hours), d-h\:mm (days, hours and minutes)
or d-h\:mm\:ss (days, hours, minutes and seconds) format. The ``-`` is not a typo!

Specifying a job name
"""""""""""""""""""""

The default name of a job is the name of the job script. The name can however be changed 
using ``--job-name=<name>`` or ``-J <name>``. 

Redirecting stdout and stderr
"""""""""""""""""""""""""""""

By default, Slurm redirects both stdout and stderr to the same file, named ``slurm-<jobid>.out``. 
There are two flags to ``sbatch`` to change this behaviour:

* ``--output=<output file>`` or ``-o <output file>`` will redirect all output to the file 
  specified by ``<output file>`` rather than the default.
* ``--error=<error file>`` or ``-e <error file>`` will redirect output sent to stderr to 
  the file specified by ``<error file>``. Output sent to stdout is still sent to the default
  file, unless ``--output`` is also used.

Hence:

* No ``--output`` and no ``--error``: stdout and stderr are both sent to the default output
  file ``slurm-<jobid>.out``.
* ``--output`` specified but no ``--error``: stdout and stderr are both sent to the file
  pointed to by ``--output``.
* No ``--output``, but ``--error`` specified: stdout is redirected to the default output file
  ``slurm-<jobid>.out`` while stderr is redirected to the file pointed to by ``--error``.
* Both ``--output`` and ``--error`` are specified: stdout is redirected to the file pointed to
  by ``--output`` and stderr is redirected to the file pointed to by ``--error``.
  
The file name can (and usually will) be a template. It can contain replacement symbols preceded 
by a % that allow to use the jobid etc. in the name of the file to ensure unique file names. 
The most useful of such symbols is ``%j`` which will be replaced by the unque jobid.
A full list of replacement symbols can be found in 
`the sbatch manual page <https://slurm.schedmd.com/sbatch.html>`_.

Sending mail at specific events
"""""""""""""""""""""""""""""""

Slurm can send mail when a job starts, fails or ends normally, and on a number of other occasions.
Two flags influence this behaviour:

* ``--mailtype=<type>`` specifies when mail should be sent. ``<type>`` is a comma-separated list
  of type values. Type values include START, END and FAIL to denote respectively the start of a 
  job, end of a job and failure of a job, but there are many other options that can be found in
  `the sbatch manual page <https://slurm.schedmd.com/sbatch.html>`_.
* ``--mail-user=<mail address>`` specifies to which mail address the mails should be sent. The
  default value is the mail address associated with the VSC-account of the submitting user.

The job environment
"""""""""""""""""""

The Slurm ``sbatch`` command by default copies the environment in which the job script was submitted
(at least, the environment seen by the ``sbatch`` command, so all exported variables and functions).
This implies that, e.g., all modules that were loaded when you submitted the job script, will
be loaded in your job environment. This poses a number of risks:

* Some modules adapt their behaviour to the environment in which they were loaded. 
  One important example are the modules that provide MPI on the cluster. When 
  launched in a Slurm job environment, some environment variables are set to
  ensure maximal integration with Slurm. However, when loaded on the login nodes
  these variables are not set as otherwise running a MPI program as a regular 
  program without ``mpirun`` (and launching just a single process) would fail.
  The latter is a problem for, e.g., Python when some module loads the Python 
  MPI package.
* You may be working in a different environment than the one you used the previous
  time you ran the job script, and as a consequence of this your job script that 
  previously functioned well may now function differently.
* Paths may be different on the login nodes and compute nodes. This can happen during
  OS upgrades of the cluster. These can often be done without downtime or interrupting
  work on the cluster, but that implies that some nodes will be running one version while
  other nodes will be running another version of the OS setup.

Therefore, to avoid accidental mistakes, we advise you to apply one of the following solutions:

* Clear your module environment using ``module purge`` and reconstruct your environment by first
  loading the appropriate calcua module (``module load calcua/supported`` will do for most users)
  and then loading the appropriate application modules.
* Use the option ``--export=NONE`` (either with the ``sbatch`` command or as a ``#SBATCH`` line 
  in the job script. This automatically implies the option ``--get-user-env``, and the effect of
  the combination of both options is that the environment in which ``sbatch`` executes is not
  propagated (the ``--export=NONE``) and an environment that resembles the environment that you 
  get when you would log on to the cluster is constructed (the ``--get-user-env`` which is 
  implied). There is a difference though with what you get when executing your
  ``.bash_profile`` script: The environment only contains exported variables and functions and
  no aliases or variables or functions that are not exported by ``.bash_profile``.  


Starting multiple copies of a process in a job script: srun
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`Slurm srun manual page on the web <https://slurm.schedmd.com/srun.html>`_ 

The ``srun`` command is used to start a new job step in a job script. The most common case is
to start a parallel application. ``srun`` integrates well with major MPI implementations and 
can be used instead of ``mpirun`` or ``mpiexec`` to start a parallel MPI application. It then
takes your resource requests and allocated resources into account and does a very good job
of starting each MPI rank on the right set of cores even without having to use additional
command line options. Further down this section/page there are a couple of examples that
demonstrate the power of the ``srun`` command. The advantage of this way of working is that
all processes run under the strict control of Slurm, ensuring that if something goes wrong,
they are also cleaned up properly.

The ``srun`` command can also be used outside of a resource allocation, i.e., at the command
line of the login nodes, outside a job script or an allocation obtained with ``salloc`` (see 
further in the text). It will then first create the resource allocation before executing the
command given as an argument to ``srun``. One useful case which we discuss further down in this
text is to start an interactive session on a single node. Most of the command line options of 
``sbatch`` to specify the properties of the allocation can also be used with ``srun``. 

Just as ``sbatch``, ``srun`` will propagate the environment. When ``srun`` is used in
a job script to start a parallel application, this is also very sensible and desired
behaviour as it ensures the processes started with Slurm run in the right environment
created by the job script.


Managing jobs
~~~~~~~~~~~~~

Checking the queue
""""""""""""""""""

`Slurm squeue manual page on the web <https://slurm.schedmd.com/squeue.html>`_ 

The Slurm comand to list jobs in the queue is ``squeue``. 

The basic command without options will show basic information about all your jobs in the queue.
There are a number of useful command line options though:
* The ``--log`` or ``-l`` flag adds some additional information. 
* ``-o <output format>`` or ``--format=<output format>`` allows you to specify 
  your custom output format that can show a lot more information. Likewise,
  ``-O <output format>`` or ``--Format=<output format>`` can show even more
  information but with a longer syntax for the output format. See the 
  `squeue manual page <https://slurm.schedmd.com/squeue.html>`_ for information
  on all format options.
* It is possible to show that information for only one or a selection of your
  jobs by using ``-j <job_id_list>``or ``--jobs=<job_id_list>``. 

The column "REASON" lists why a job is waiting for execution. It distinguishes between
30+ different reasons, way to much to discuss here, but some of the codes speak for
themselves.


Kill/delete a job
"""""""""""""""""

`Slurm scancel manual page on the web <https://slurm.schedmd.com/scancel.html>`_ 

The Slurm command to cancel a job is ``scancel``. In most cases, it takes only a 
single argument, the unique identifier of the job to cancel. 

For a job array (see below) it is also possible to cancel only some of the jobs in
the array by specifying the array elements as follows:

.. code:: bash
   
   scancel 20_[1-3]
   scancel 20_4 20_6

The first command would kill jobs 1, 2 and 3 in the job array with job id 20,
the second command would kill jobs 4 and 6 of that job array.


Getting more information on a running job
"""""""""""""""""""""""""""""""""""""""""

`Slurm sstat manual page on the web <https://slurm.schedmd.com/sstat.html>`_ 

The ``sstat`` command displays information on running jobs. By default, without
any arguments, ``sstat`` will show you information on pertaining to CPU, Task, 
Node, Resident Set Size (RSS) and Virtual Memory (VM)
for all your running jobs. However, it is possible to specify a particular job
you want information about by specifying ``-j <jobid>`` or ``--jobs=<jobid>``.
It is possible to specify multiple jobs as a comma-separated list of job IDs.
By default it will only show information about the lowest job step running in 
a particular job unless ``--allsteps`` or ``-a`` is also specified.
It is also possible to request information on a specific job step of a job
by using ``<jobID.JobStep>``, i.e., add the number of the job step to the
job ID, separated by a dot.

To show additional information not shown by the default format, one can
specify a specific format using the ``-o``,  ``--format`` and ``--fields``
flags. We refer to the `manual page <https://slurm.schedmd.com/sstat.html>`_
for further information.


Getting information about a job after it finishes
"""""""""""""""""""""""""""""""""""""""""""""""""

`Slurm sacct manual page on the web <https://slurm.schedmd.com/sacct.html>`_ 

Whereas ``sstat`` is used to show near real-time information for running jobs,
``sacct`` shows the information as it is kept by Slurm in the job accounting
log/database. Hence it is particularly useful to show information about jobs 
that have finished already. It allows you to see how much CPU time, walltime, 
memory, etc. were used by the application. 

By default, ``sacct`` shows the job ID, job name, partition name, account name,
number of CPUs allocated to the job, the state of the job and the exit code
of completed jobs. For now there is no reason to be concerned about the
account name as we do not use accounting to control the amount of compute time
a user can use. Several options modify the format:

* ``--brief`` or ``-b`` shows only the job id, state and exit ode.
* ``--long`` or ``-l`` shows an overwhelming amount of information, probably more than you
  want to know as a regular user.
* ``--format`` or ``-o`` can be used to specify your own output format. We refer 
  the the `manual page <https://slurm.schedmd.com/sacct.html>`_ for an overview of 
  possible fields and how to construct the format string.
  
By default, ``sacct`` will show information about jobs that have been running
since midnight. There are however a number of options to specify for which jobs 
you want to see information:

* ``--jobs=<jobIDs>`` or ``-j <jobIDs>`` with a comma-separated list of jobIDs (in
  the same format as for ``sstat``) will only show information on those jobs 
  (or job steps).
* ``--startime=<time>`` or ``-S <time>``: Show information about all jobs that 
  have been running since the indicated start time. There are four possible 
  formats for ``<time>``: HH:MM[:SS] [AM|PM], MMDD[YY] or MM/DD[/YY] or MM.DD[.YY],
  MM/DD[/YY]-HH:MM[:SS] and YYYY-MM-DD[THH:MM[:SS]] (where [] denotes an optional
  part).
* ``--endtime=<time>`` or ``-E <time>``: Show information about all jobs that 
  have been running before the indicated end time. By combining a start time and 
  end time it is possible to specify a window for the jobs.

 
Job types: Examples
-------------------

Shared memory job
~~~~~~~~~~~~~~~~~

Shared memory programs often need to be told how many threads they should use. 
Unfortunately, there is no standard way that works for all programs. Some programs
require an environment variable to be set, others have a parameter in the input file
and some interpreters (e.g., Matlab) require it to be set in the code being interpreted.

OpenMP is a popular technology for creating shared memory programs. Some OpenMP programs
will read the number of threads from the input file and then set it using OpenMP library functions.
But most OpenMP programs simply use the environment variable ``OMP_NUM_THREADS`` to
determine the number of threads that should be used. The following job script is an 
example of this. It assumes there is a program ``omp`` in the current directory 
compiled with the intel/2019b toolchain. It will be run on 10 cores.

.. code:: bash
   
   #!/bin/bash
   #
   #SBATCH --job-name=OpenMP-demo
   #SBATCH --ntasks=1 --cpus-per-task=10 --mem=2g
   #SBATCH --time=05:00
   
   # Build the environment
   module purge
   module load calcua/2020a
   module load intel/2020a
   
   # Set the number of threads
   export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
   
   # Run the program
   ./omp
   
In fact, when using Intel OpenMP, not setting the variable ``OMP_NUM_THREADS``
seems to work fine also as the runtime seems to recognize Slurm and pick up
the right number of threads.
 
 
MPI program
~~~~~~~~~~~
 
Running distributed memory programs usually requires a program starter.
In the case of MPI programs, the usual way to start a program is through
the ``mpirun`` or ``mpiexec`` command. The major MPI implementations will
recognize Slurm (sometimes with the help of some environment variables)
and work with Slurm to start the MPI processes on the correct cores
and under the control of the resource manager (so that they are cleaned
up properly if things go wrong). 
However, with several implementations, it is also possible to use the 
Slurm ``srun`` command to start the MPI processes. This is the case
for programs compiled with Intel MPI as the example below shows. The
example assumes there is a MPI program called ``mpi_hello`` in the current
directory compiled with Intel MPI.

.. code:: bash
   
   #!/bin/bash
   #
   #SBATCH --job-name=mpihello
   #SBATCH --ntasks=56 --cpus-per-task=1 --mem-per-cpu=512m
   #SBATCH --time=5:00
   
   # Build the environment
   module purge
   ml calcua/2020a
   ml intel/2020a
   
   # Run the MPI program
   srun ./mpi_hello

In the above case, 56 MPI ranks will be spawned, corresponding to two
nodes on a cluster with 28 cores per node.
 
 
Hybrid MPI/OpenMP program
~~~~~~~~~~~~~~~~~~~~~~~~~

Some programs are hybrids combining a distributed memory technology with a shared
memory technology. The idea is that shared memory doesn't scale beyond a single
node (and often not even to the level of a single node), while distributed 
memory programs that spawn a process per core may also suffer from too much memory
and communication overhead. Combining both can sometimes give better performance
for a given number of cores. Especially the combination of MPI and OpenMP is
popular. Such programs require a program starter and need the number of threads
to be set in one way or another. With many MPI implementations (and the ones
we use at the VSC), ``srun`` is an ideal program starter and will start the
hybrid MPI/OpenMP processes on the right sets of cores.
The example below assumes ``mpi_omp_hello`` is a program compiled with
the Intel toolchain that uses both MPI and OpenMP. It starts 8 processes
with 7 threads each, so it would occupy two nodes on a cluster with 28 cores
per node.

.. code:: bash
   
   #! /bin/bash
   #SBATCH --ntasks=8 --cpus-per-task=7 --mem-per-cpu=512m
   #SBATCH --time=5:00
   #SBATCH --job-name=hybrid-hello-test
   
   module purge
   module load calcua/supported
   module load intel/2020a

   # Set the number of threads per MPI rank
   export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

   # Start the application
   srun ./mpi_omp_hello
 
As with shared memory programs, it turns out that setting OMP_NUM_THREADS is 
not needed most of the time when the Intel compilers were used for the application
as they pick up the correct number of threads from Slurm.

 
Job array
~~~~~~~~~

`Slurm manual page on job array <https://slurm.schedmd.com/job_array.html>`_

Slurm also supports job arrays. This is a mechanism to submit and manage a collection of
similar jobs simultaneously much more efficiently then when they are submitted as
many regular jobs. When submitting a job array, a range of index values is given.
The job script is then started for each of the index values and that value is
passed to the job through the ``SLURM_ARRAY_TASK_ID`` variable.

E.g., assume that there is a program called ``test_set`` in the current directory
that reads from an input file and writes to an output file, and assume that we want
run this for a set of input files named ``input_1.dat`` to ``input_100.dat``, writing
the output to ``output_1.dat`` till ``output_100.dat``. The job script would look like:

.. code:: bash
   
   #!/bin/bash -l
   
   #SBATCH --ntasks=1 --cpus-per-task=1
   #SBATCH --mem-per-cpu=512M
   #SBATCH --time 15:00
   
   INPUT_FILE="input_${SLURM_ARRAY_TASK_ID}.dat"
   OUTPUT_FILE="output_${SLURM_ARRAY_TASK_ID}.dat"
   
   ./test_set ${SLURM_ARRAY_TASK_ID} -input ${INPUT_FILE}  -output ${OUTPUT_FILE}
   
Assume the filename of the script is ``job_array.slurm``, then it would be
submitted using

.. code:: bash

   sbatch --array=1-100 job_array.slurm

Within the VSC, the package ``atools`` was developed to ease management of job arrays
and to start programs using parameter values stored in a CSV file that can be generated
easily using a spreadsheet program. On the UAntwerp clusters, the most recent version of
the package is available as the module ``atools/slurm``. 
For information on how to use atools, we refer to the 
`atools documentation on ReadTheDocs <https://atools.readthedocs.io/en/latest/>`_.

Interactive job
~~~~~~~~~~~~~~~

Starting an interactive job in Slurm is a bit more cumbersome then it was with 
Torque. We do need to distinguish between two scenarios:
1. A request for a number of cores on a single node
2. Requests that involve multiple nodes, e.g., to test an MPI program.

Simple request, cores on a single node
""""""""""""""""""""""""""""""""""""""

This kind of requests can be done easily by using ``srun`` on the command line of
one of the login nodes. E.g.,

.. code:: bash

   srun --nodes=1 --cpus-per-task=10 --time=10:00 --mem=4G --pty bash 
  
or briefly

.. code:: bash

   srun -N1 -c10 -t10 --mem=4G --pty bash 

will start a bash shell on a compute node and allocate 10 cores and 4 GB of memory
to that session. The maximum wall time of the job is set to 10 minutes.

Requesting cores on multiple nodes
""""""""""""""""""""""""""""""""""

Using multiple nodes in an interactive job is a two-step process. 
First a Slurm *job* is created using ``salloc``. This command takes most of the
same paramters as ``sbatch``. 

.. code:: bash

   salloc --nodes=2 --cpus-per-task=20 --time=10:00 --mem=4G bash
   
or briefly

.. code:: bash

   salloc -N2 -c20 -t10 --mem=4G bash

will make an allocation for 2 nodes with 20 cores each. It will then start
``bash``. However, ``bash`` will not run on one of the nodes allocated to the
job, but on the node where you executed the ``salloc`` command (which would
typically be a login node). 

In that shell you can then create *job steps* using ``srun`` in the same way
as you would do in a batch script using ``srun``. E.g.,

.. code:: bash

   srun -l hostname
   
will execute the ``hostname`` command on both nodes of the allocation.

  








 