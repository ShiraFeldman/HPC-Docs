# Portable Batch System (PBS) Job Scripts

**Job scripts** form the basis of *batch jobs*.
A job script is simply a text file with instructions of the work to
execute.  Job scripts are *usually* written in `bash` or `tcsh` and
thus mimic commands a user would execute interactively through a
shell; but instead are executed on specific resources allocated by the
scheduler when available.  Scripts can also be written in other
languages - commonly Python.


## Anatomy of a Job Script
Sample basic PBS scripts are listed below:
!!! example "PBS job scripts"
    === "bash"
        ```bash
        #!/bin/bash
        #PBS -N hello_pbs
        #PBS -A <project_code>
        #PBS -j oe
        #PBS -k eod
        #PBS -q main
        #PBS -l walltime=00:05:00
        #PBS -l select=2:ncpus=128:mpiprocs=128

        ### Set temp to scratch
        export TMPDIR=${SCRATCH}/${USER}/temp && mkdir -p $TMPDIR

        ### specify desired module environment
        module purge
        module load ncarenv/23.09 gcc/12.2.0 cray-mpich/8.1.25
        module list

        ### Compile & Run MPI Program
        mpicxx -o hello_world_derecho /glade/u/home/benkirk/hello_world_mpi.C -fopenmp
        mpiexec -n 256 -ppn 128 ./hello_world_derecho
        ```

        ---

        The first line denotes the interpreter to be used for the script.
        ```bash
        #!/bin/bash
        ```
        indicates this is a `bash` script, and that the `bash` shell will be used to parse its contents.


    === "tcsh"
        ```tcsh
        #!/bin/tcsh
        #PBS -N hello_pbs
        #PBS -A <project_code>
        #PBS -j oe
        #PBS -k eod
        #PBS -q main
        #PBS -l walltime=00:05:00
        #PBS -l select=2:ncpus=128:mpiprocs=128

        source /etc/csh.cshrc

        ### Set temp to scratch
        setenv TMPDIR ${SCRATCH}/${USER}/temp && mkdir -p ${TMPDIR}

        ### specify desired module environment
        module purge
        module load ncarenv/23.09 gcc/12.2.0 cray-mpich/8.1.25
        module list

        ### Compile & Run MPI Program
        mpicxx -o hello_world_derecho /glade/u/home/benkirk/hello_world_mpi.C -fopenmp
        mpiexec -n 256 -ppn 128 ./hello_world_derecho
        ```

        ---

        The first line denotes the interpreter to be used for the script.
        ```tcsh
        #!/bin/tcsh
        ```
        indicates this is a `tcsh` script, and that the `tcsh` shell will be used to parse its contents.

    === "Python"
        ```python
        #!/glade/u/apps/opt/conda/envs/npl/bin/python
        #PBS -N hello_pbs
        #PBS -A <project_code>
        #PBS -j oe
        #PBS -k eod
        #PBS -q main
        #PBS -l walltime=00:05:00
        #PBS -l select=1:ncpus=128:mpiprocs=128

        import sys
        print("Hello, world!!\n\n")

        print("Python version:")
        print(sys.version)
        print("Version info:")
        print(sys.version_info)
        ```

        ---

        The  first line denotes the interpreter to be used for the script.
        ```python
        #!/glade/u/apps/opt/conda/envs/npl/bin/python
        ```
        indicates this is a `python` script (and, specifically, the NCAR NPL instance), and that the `python` interpreter will be used to parse its contents.

**Focusing on the `bash` example for discussion**, the remainder of the script contains two main sections:

1.  The lines beginning with `#PBS` are **directives** that will be interpreted by PBS when this script is submitted with `qsub`.
    Each of these lines contains an instruction that will be used by `qsub` to control job resources, execution, etc...

2.  The remaining **script contents** are simply `bash` commands that will be run inside the batch environment on the selected resources and define the work to be done in this job.


### `#PBS` Directives

The example above contains several **directives#* which are interpreted by the `qsub` submission program:

* `-N hello_pbs` provides a *job name*.  This name will be displayed by the scheduler for diagnostic and file output.  If omitted, and a script is used to submit the job, the job's name is the  name  of  the  script.
* `-A <project_code>` indicates which *NCAR Project Accounting code* resource allocation will be applicable to this job. (You will want to replace `<project_code>` with your project's specific code.)
* `-j oe` requests we *combine any standard text output (`o`) and error (`e`) into one output file*.
  (By default, PBS will write program output and error to different log files.  This behavior is contrary to what many users expect from terminal interaction, where output and error are generally interspersed. This optional flag changes that behavior.)
* `-q main` specifies the desired PBS *queue* for this job.
* `-l walltime=00:05:00` requests 5 minutes as the maximum job execution (*walltime*) time.  Specified in `HH:MM:SS` format.
* `-l select=2:ncpus=128:mpiprocs=128` is a computational *resource chunk* request, detailing the quantity and configuration of *compute nodes* required for this job. This example requests a *selection* of 2 nodes, where each node must have 128 CPU cores, each of which we will use as an MPI rank in our application.


### Script Contents

The remaining script contains shell commands that define the job execution workflow.  The commands here are arbitrary, however we strongly recommend the general structure presented above.  This includes:

1.  **Explicitly setting the `TMPDIR` variable**.

    As described [here](../storing-temporary-files.md), many programs
    write temporary data to `TMPDIR`, which is usually small and
    shared among st users.  Specifying your own directory for temporary
    files can help you avoid the risk of your own programs and other
    users' programs failing when no more space is available.

2.  **Loading and reporting the specific module environment required for this job.**

    While strictly not necessary (in general, the system default modules will be loaded anyway), we recommend this as best practice as it facilitates debugging and reproducing later.  (While the system default modules will change over time, manually specifying module versions allows you to recreate the same execution environment in the future.)

4.  **(*Optional*) Defining any environment variables specific to the chosen module environment.**

    Occasionally users will want to define particular run time environment variables e.g. for a specific MPI or library chosen via the `module load` commands.


5.  **Remaining job-specific steps.**

    In the example above, we first compile and then execute `hello_world_mpi.C`, a simple MPI program.


---


## Common `#PBS` directives
<!-- FIXME -->

### Resource `select` statements
<!-- FIXME -->

### Job Priority
<!-- FIXME -->

### Listing of common `#PBS` directives
<!-- FIXME -->

## Execution environment variables
Within the **script contents** of the job script, it is common for the
specifics of the job to depend slightly on the PBS and specific
`module` execution environment.  Both running under PBS and loading
certain [module files](../../environment-and-software/user-environment/modules.md)
create some environment variables that might be useful when writing
portable scripts; for example scripts that might be shared among users
or executed within several different configurations.

!!! tip "Use common environment variables to write portable PBS batch scripts"
    Avoid hard-coding paths into your shell scripts if instead any of
    the environment variables below might be used.  This will
    facilitate moving scripts between systems, users, and application
    versions with minimal modifications, as output paths can be
    defined generically as opposed to hard-coded for each user.


### PBS execution environment variables
PBS creates a number of environment variables that are accessible within a job's execution environment.
Some of the more useful ones are:


|  <div style="width:110px">Variable</div> | Value |
|---------------|-------|
| `PBS_ACCOUNT`   | The NCAR Project Accounting code used for this job. |
| `PBS_JOBID`     | The PBS Job ID for this job.<br>Example: `1473351.desched1` |
| `PBS_JOBNAME`   | The name of this job. Matches the `-N` specified.<br>Example: `hello_pbs` |
| `PBS_O_WORKDIR` | The working directory from where the job was submitted. |
| `PBS_SELECT`    | The resource specification `-l select=` line for this job.<br>This can be useful for setting runtime-specific configuration options that might depend on resource selection.<br>(e.g. processor layout, CPU binding, etc...)<br>Example: `2:ncpus=128:mpiprocs=2:ompthreads=2:mem=200GB:Qlist=cpu:ngpus=0`  |
| `PBS_NODEFILE`  | A file whose contents lists the nodes assigned to this job.<br> Typically listed as one node name per line, for each MPI rank in the job.<br>Each node will be listed for as many times as it has MPI ranks. <br>Example: `/var/spool/pbs/aux/1473351.desched1` |

<!-- | `PBS_QUEUE`     | The name of the PBS execution queue.<br>In general this will be an *execution* queue, and different than the *routing* queue specified by `-q` in the job script.<br>Example: `cpu` | -->


### NCAR `module`-specific execution environment variables

| <div style="width:200px">Variable</div>                  | Value                                                                                                           |
|----------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| <p style="text-align: center;"><strong>Machine and Software Environment</strong></p> {: colspan=2} |
| `NCAR_HOST`                                              | Specifies the *host class* of machine, e.g. `derecho` or `casper`                                               |
| `NCAR_BUILD_ENV_COMPILER`                                | A unique string identifying the host and compiler+version currently loaded.<br>Example: `casper-oneapi-2023.2.1` |
| `NCAR_BUILD_ENV_MPI`                                     | A unique string identifying the host, compiler+version, and mpi+version currently loaded.<br> Example: `casper-oneapi-2023.2.1-openmpi-4.1.5` |
| `NCAR_BUILD_ENV`                                         | A unique string identifying the current *build environment*, identical to `NCAR_BUILD_ENV_MPI` when an MPI module is loaded, or `NCAR_BUILD_ENV_COMPILER` if only a compiler is loaded. |
| `LMOD_FAMILY_COMPILER`<br>`LMOD_FAMILY_COMPILER_VERSION` | Specifies the type and version of compiler currently loaded, if any.<br>Example:`intel`, `gcc`, `nvhpc`         |
| `LMOD_FAMILY_MPI`<br>`LMOD_FAMILY_MPI_VERSION`           | Specifies the type and version of MPI currently loaded, if any.<br>Example:`openmpi`, `cray-mpich`, `intel-mpi` |
| <p style="text-align: center;"><strong>User and File System Paths</strong></p> {: colspan=2} |
| `${USER}`    | The `username` of user executing the script. |
| `${HOME}`    | The GLADE home file space for the user executing the script.<br>Example: `/glade/u/home/${USER}` |
| `${WORK}`    | The GLADE work file space for the user executing the script.<br>Example: `/glade/work/${USER}` |
| `${SCRATCH}` | The GLADE scratch file space for the user executing the script.<br>Example: `/glade/derecho/scratch/${USER}` |

---

## Sample PBS job scripts

<!-- PBS_SELECT=2:ncpus=128:mpiprocs=2:ompthreads=2:mem=200GB:Qlist=cpu:ngpus=0+1:ncpus=128:mpiprocs=4:ompthreads=4:mem=200GB:Qlist=cpu:ngpus=0 -->