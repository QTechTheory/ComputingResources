SLURM guide
===========

Both [ARCUS-B](arcusB.md) and [ARCUS-HTC](arcusHTC.md) use [SLURM](https://slurm.schedmd.com/documentation.html) to submit jobs to the cluster. Here are some useful and advanced SLURM commands and example submission scripts.

  * [Submitting a serial job](#submitting-a-serial-job)
  * [Submitting a multithreaded job](#submitting-a-multithreaded-job)
  * [Submitting a distributed job](#submitting-a-distributed-job)
  * [Submitting multiple serial jobs (one node)](#submitting-multiple-serial-jobs-one-node)
  * [Submitting *many* serial jobs (many nodes)](#submitting-many-serial-jobs-many-nodes)
  * [Submitting multiple multithreaded jobs](#submitting-multiple-multithreaded-jobs)
  * [Submitting multiple distributed & multithreaded jobs](#submitting-multiple-distributed---multithreaded-jobs)
  * [Sweeping parameters between multiple jobs](#sweeping-parameters-between-multiple-jobs)
  * [Checking the status of jobs](#checking-the-status-of-jobs)
  * [Cancelling a job](##cancelling-a-job)
  * [Delaying enqueued jobs](#delaying-enqueued-jobs)
  * [Requeuing and delaying jobs](#requeuing-and-delaying-jobs)
  

Jobs are submitted via a submission script. These consist of a bunch of `#SBATCH` commands interpreted by SLURM which specify *where* and *how* to run the proceeding bash script.  

## Submitting a serial job

Assuming `my_executable` is a precompiled serial program, here's a template of a SLURM submit script to run for 1 hour on a single node in the `nqit` partition (accessible as a member of Simon Benjamin's `oums-quantopo` group), and email `myemail@dom.com` when the job starts and finishes.
```
#!/bin/bash

#SBATCH --job-name=myjob
#SBATCH --mail-user=myemail@dom.com
#SBATCH --mail-type=ALL

#SBATCH --time=1:00:00
#SBATCH --nodes=1

./my_executable
```
Save this as `submit.sh`, and submit the job to the queue via `sbatch submit.sh`.

> **NOTE**:
> - **ARCUS-B** will dedicate an entire node to the job, even though the number of cores `ntasks-per-node` wasn't specified.
> - **ARCUS-HTC** and **ARCUS-C** will dedicate only a single core (meaning multiple jobs may run on a single node), unless one specifies `ntasks-per-node` or reserves the entire node with:
>   ```
>   #SBATCH --exclusive
>   ```


## Submitting a multithreaded job

To use **multiple cores**  to run a **single** instance of a **multithreaded** application, you must additionally specify `--ntasks-per-node` and ensure the environment variable `OMP_NUM_THREADS` is set (for good measure). 
For example, here using the maximum 16 cores for [ARCUS-B](arcusB.md):

```
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16

export OMP_NUM_THREADS=16
./my_executable
```


## Submitting a distributed job

To use **mutliple nodes** to run a **single** instance of a **distributed** (MPI) application, use the `openmpi` module. In this example, we'll use `4` nodes.

```
#SBATCH --nodes=4

## only needed on ARCUS-B:
. enable_arcus-b_mpi.sh

module load openmpi
mpirun -np 4 my_executable
```

If our application is MPI-OpenMP hybridised (like [QuEST](https://github.com/QuEST-Kit/QuEST)), we may wish to use multiple cores on each node to perform additional multithreading. For example, using 16 cores per node:

```
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=16

module load openmpi

## only needed on ARCUS-B:
. enable_arcus-b_mpi.sh

export OMP_NUM_THREADS=16
mpirun -np 4 my_executable
```

> As of 2020 (persisting since 2017), the ARCUS-B network fabric suffers mysterious and unknown latency between some (uncontrollable) nodes. Avoid `ARCUS-B` for crucially fast distributed jobs (especially benchmarking).


## Submitting multiple serial jobs (one node)

To use a **single node** to run **multiple** independent **serial** jobs in parallel, use [GNU Parallel](https://www.gnu.org/software/parallel/). Adding `&` after a command will run it in parallel and asynchronously. Be sure to `wait` for all asynch commands to finish before exciting the job script. For example, here we run `my_executable` with **16** different command-line arguments (numbers `1` to `16`).

```
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16

export OMP_NUM_THREADS=1

for id in `seq 1 16`; do
    ./my_executable $id &
done

# don't forget to wait!!
wait
```

One can "queue" as many serial jobs as desired, and GNU Parallel will manage running them performantly.

```
## queue 100 jobs between the 16 cores

for id in `seq 1 100`; do
    ./my_executable $id &
done

wait
```

## Submitting *many* serial jobs (many nodes)

To use **multiple nodes** to run **many** independent **serial** jobs, combine SLURM `array` with [GNU Parallel](https://www.gnu.org/software/parallel/). For example, here we use **4** nodes, each with **16** cores, to run a total of `4 * 16 = 64` independent instances of `my_executable`:

```
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16
#SBATCH --array=1-4

export OMP_NUM_THREADS=1

NUM_LOCAL=16
RUN_ID_FIRST=$[ (SLURM_ARRAY_TASK_ID-1)*NUM_LOCAL + 1 ]
RUN_ID_LAST=$[ RUN_ID_FIRST + NUM_LOCAL - 1 ]

for id in `seq $RUN_ID_FIRST $RUN_ID_LAST`; do
    ./my_executable $id &
done

# don't forget to wait!
wait
```

Each instance of `my_executable` will receive a unique command-line argument (a number `1` to `64`).

> To use a different number of cores per-node, you must change both `ntasks-per-node` and `NUM_LOCAL`.



## Submitting multiple multithreaded jobs

To use **multiple nodes** (e.g. `4`) to each run independent **multithreaded** jobs (e.g. `16` threads), use a SLURM `array`:

```
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=16
#SBATCH --array=1-4

export OMP_NUM_THREADS=16
./my_executable $SLURM_ARRAY_TASK_ID
```
Each execution of `my_executable` will run on a different node, and receive a unique command-line argument (a number between `1` and `4`).



## Submitting multiple distributed & multithreaded jobs

> **THIS IS UNTESTED**. Who the hell wants to do this anyway?

To launch **multiple** (e.g. `5`) **distributed** jobs, each using e.g. `4` nodes, where a single node in a job uses (e.g.) `16` local cores, add a SLURM `array` to the distributed template:


```
#SBATCH --array=1-5
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=16

## only needed on ARCUS-B:
. enable_arcus-b_mpi.sh

module load openmpi
export OMP_NUM_THREADS=16
mpirun -np 4 my_executable $SLURM_ARRAY_TASK_ID
```

Each of the 5 distributed instances of `my_executable` (each one employing 4 nodes) receives a unique command-line argument (a number between `1` and `5`).


## Sweeping parameters between multiple jobs

It's often tricky to map SLURM jobs to unique assignments of command-line arguments, so that each job performs an independent computation. [This tool](https://github.com/TysonRayJones/PythonTools/tree/master/slurm_param_sweeper) will generate submit scripts which sweep for you!


## Checking the status of jobs

- To see the status of your running or in-queue jobs, use
  ```
  squeue -u [username]
  ```
  where `[username]` is e.g. `wolf5549`. 

- You can see your running and recently finished jobs with (no arguments needed):
  ```
  sacct
  ```

- To see jobs running on the `nqit` reservation, use
  ```
  squeue --reservation nqit
  ```

- To see only jobs that are in queue for the `nqit` reservation, use
  ```
  squeue --reservation nqit --start
  ```

## Cancelling a job

- To cancel a job, use
  ```
  scancel [JOB ID]
  ```

- Cancel all your jobs with 
  ```
  scancel -u [NAME]
  ```

## Delaying enqueued jobs

> Because the `suspend` and `hold` commands don't seem to do anything...

These commands are useful for managing the load on the cluster, by delaying your enqueued jobs to let other jobs run first.

- To delay an enqueued job with id `[JOB ID]` (learn via `squeue`) for (e.g.) `7` days:
  ```bash
  scontrol update JobID=[JOB ID] StartTime=now+7days
  ```
  The `NODELIST(REASON)` field reported by `squeue` will change to `(BeginTime)`.
  > This will have no effect on a job that's already running.

- To delay **multiple** enqueued jobs by (e.g.) `7` days, 
  ```bash
  for jobid in [SPACE SEPARATED LIST OF JOB IDS]; scontrol update JobID=$jobid StartTime=now+7days; done
  ```
  where `[SPACE SEPARATED LIST OF JOB IDS]` is what it says on the tin.
  > For example, if my enqueued jobs have IDs:
  > ```
  > 1234567
  > 1234568
  > 1234569
  > ```
  > then my command to delay them a day is
  > ```
  > for jobid in 1234567 1234568 1234569; scontrol update JobID=$jobid StartTime=now+1days; done
  > ```
  See the proceeding section for an example of a command to delay multiple jobs which share a common prefix.

## Requeuing and delaying jobs

These commands are useful for managing the load on the cluster, by stopping your currently running jobs and requeueing them later, to let other jobs run first.

- To kill a running job and requeue it with a delay (e.g. for `1` day):
  ```bash
  (export jobid=[JOB ID]; scontrol requeue $jobid; scontrol update JobID=$jobid StartTime=now+1day)
  ```
  Sometimes these requeued jobs do *not* appear in `squeue`, but rest assured that they're still enqueud!
  > It's important these `requeue` and `update` commands are done in this single cmd command, since otherwise the job may rerun immediately before the delay is applied

- To kill and requeue (with delay e.g. `1` day) **multiple** running jobs:
  ```bash
  for jobid in [SPACE SEPARATED LIST OF JOB IDS]; do scontrol requeue $jobid; scontrol update JobID=$jobid StartTime=now+1day; done
  ```

- Often many jobs share a common prefix, like those submitted using SLURM `array`. To avoid retyping this prefix, we can save it with `export`.
  Here's how to kill and requeue (with delay e.g. `1` day) multiple running jobs which share a common prefix:
  ```bash
  (export prefix=[COMMON JOB ID PREFIX]; for suffix in [SPACE SEPARATED LIST OF JOB ID SUFFIXES]; do scontrol requeue ${prefix}${suffix}; scontrol update JobID=${prefix}${suffix} StartTime=now+1day; done)
  ```
  > For example, if the current running jobs have IDs
  > ```bash
  > 1234567_10
  > 1234567_11
  > 1234567_12
  > 1234567_13
  > 1234567_14
  > ```
  > then they can be killed and requeued (with a `1` day delay) via:
  > ```bash
  > (export prefix=1234567_1; for suffix in 0 1 2 3 4; do scontrol requeue ${prefix}${suffix}; scontrol update JobID=${prefix}${suffix} StartTime=now+1day; done)
  > ```
