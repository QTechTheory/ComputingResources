ARCUS-C
=============

The `oums-quantopo` group (lead by Simon Benjamin) has privileged access to the `short` partition in ARCUS-C.

## Hardware

The hardware in the `short` partition includes:

- 45 **CPU nodes** ([SR630](https://lenovopress.com/lp0643.pdf)), each with:
  * 48 core (96 threads) CPU (two [Xeon 8268](https://ark.intel.com/content/www/us/en/ark/products/192481/intel-xeon-platinum-8268-processor-35-75m-cache-2-90-ghz.html), 2.9GHz, 72MB cache)
  * 384GB RAM
 
- 8 **GPU nodes** ([SR670](https://lenovopress.com/lp0923.pdf)) with:
  * 48 core (96 threads) CPU (two [Xeon 8268](https://ark.intel.com/content/www/us/en/ark/products/192481/intel-xeon-platinum-8268-processor-35-75m-cache-2-90-ghz.html), 2.9GHz, 72MB cache)
  * 384GB RAM
  * ***two*** NVIDIA [V100](https://images.nvidia.com/content/technologies/volta/pdf/tesla-volta-v100-datasheet-letter-fnl-web.pdf) GPUs (each 32GB, 5120 cores + 640 tensor cores)
    - the two GPUs are on separate sockets, like the Xeon CPUs.
   
- Network fabric ([QM8700](https://www.mellanox.com/products/infiniband-switches/QM8700)):
  * 16 Tb/s bandwidth
  * <130 ns latency
 
 
## Access

These nodes live in `arc-c`, accessible from gateway ARC network `oscgate.arc.ox.ac.uk`. Access from outside of the Oxford University network may first require VPN to server `vpn.ox.ac.uk` (using `oxfordusername@ox.ac.uk`). 

Access using your regular ARC account `[USER]` by two tunnels:

```bash
ssh [USER]@oscgate.arc.ox.ac.uk
```
then
```bash
ssh [USER]@arc-login
```

This is the level from which jobs should be submitted. 

## Job Submission

From `arc-login`, one can submit [SLURM](https://slurm.schedmd.com/documentation.html) jobs to the CPU or GPU nodes. See [here](slurmguide.md) for a general guide on submitting and parallelising jobs. Below describes only the specifics to ARCUS-C.

`arc-login` can be used to submit jobs both to `arcus-htc` and to `arc-c`, so it is important to specify from within the submit script that you want the job to run on the new nodes, otherwise the scheduler may change the location depending on availability. The flags below should always be present to ensure that the job is run on the new nodes, with no time restrictions:

  ```bash
  #!/bin/bash

  #SBATCH --account=oums-quantopo
  #SBATCH --partition=short
  #SBATCH --constraint=cpu_gen:Cascade_Lake
  #SBATCH --constraint=cpu_sku:Platinum_8628
  #SBATCH --clusters=arc 
  ```



#### CPU

- To request a single CPU core for an hour, create submission script `submit.sh` with contents:
  ```bash
  #!/bin/bash
   
  #SBATCH --account=oums-quantopo
  #SBATCH --partition=short
  #SBATCH --constraint=cpu_gen:Cascade_Lake
  #SBATCH --constraint=cpu_sku:Platinum_8628
  #SBATCH --clusters=arc 
   
  #SBATCH --nodes=1
  #SBATCH --time=1:00:00
   
  ./my_executable
  ```
  and submit it with `sbatch submit.sh`.
  > Note that (unlike ARCUS-B), this job runs on a single CPU core despite `--nodes=1`; the remaining cores (and possibly GPU resources) remain available for other jobs.

- To reserve the entire node and use all **96** hardware threads, use 
  ```bash
  #SBATCH --exclusive
  ```

- In the submission script, it is important to remember to add
  ```bash
  export OMP_NUM_THREADS=10
  ``` 
  to use the required number of threads on a given job

#### GPU

- To request a GPU, merely add `--gres=gpu:1` to the CPU script:
  ```bash
  #!/bin/bash
   
  #SBATCH --account=oums-quantopo
  #SBATCH --partition=short
  #SBATCH --constraint=cpu_gen:Cascade_Lake
  #SBATCH --constraint=cpu_sku:Platinum_8628
  #SBATCH --clusters=arc 
   
  #SBATCH --nodes=1
  #SBATCH --gres=gpu:1
  #SBATCH --time=1:00:00
   
  ./my_executable
  ```
  This will reserve a single CPU core and a single GPU.

- To reserve both GPUs on a node, use 
  ```bash
  #SBATCH --gres=gpu:2
  ```

- To reserve an entire GPU node (all cores, both GPUs), use:
  ```bash
  #SBATCH --gres=gpu:2
  #SBATCH --exclusive
  ```
 

#### Specific Node

One can submit to a specific node, so that for example they could directly SSH into the node during the job, via:
 ```bash
 #SBATCH --nodelist=arc-c[001]
 ```
The CPU nodes are available at `arc-c[001-045]`, and the GPU nodes at `arc-c[001-008]`. 


#### Memory

Though it is not currently the case, the memory allocation per job may be hard-restricted. For this reason, it is advisable to specify the required memory per node in the submit script, using:

 ```bash
 #SBATCH --mem=<memory_in_MB>
 ```

#### Distributed Simulations

Load the required compiler for compilation. These are listed using `module avail mpi`, and the specific module should be loaded using `module load <moduleName>`

The number of MPI ranks per node (which should usually be 1) can be set using:
 ```bash
 #SBATCH --ntasks-per-node=1
 ```

And the number of nodes using:
 ```bash
 #SBATCH --nodes=<numNodes>
 ```

There are also a number of flags that must always be added to a submission script to enable a distributed run. These are:

 ```bash
  export UCX_IB_MLX5_DEVX=n
  export I_MPI_DEBUG=9
  export FI_PROVIDER=mlx
  export OMP_NUM_THREADS=<numThreads>
 ```

In the submission script, the appropriate MPI module should also be loaded using 
 ```bash
 module load moduleName
 ```
 And the executable should be run using
 ```bash
 mpirun -np <numNodes> my_executable
 ```

## Example Scripts
 
For a job that is to run for 1 hour on a single core of a single node:

 ```bash
  #!/bin/bash
   
  #SBATCH --account=oums-quantopo
  #SBATCH --partition=short
  #SBATCH --constraint=cpu_gen:Cascade_Lake
  #SBATCH --constraint=cpu_sku:Platinum_8628
  #SBATCH --clusters=arc 
   
  #SBATCH --nodes=1
  #SBATCH --time=1:00:00

  ./my_executable
  ```

For a job that is to run with a limit of 24 hours, using 48 threads on each of 8 nodes, requiring 384GB per node:

 ```bash
  #!/bin/bash
   
  #SBATCH --account=oums-quantopo
  #SBATCH --partition=short
  #SBATCH --constraint=cpu_gen:Cascade_Lake
  #SBATCH --constraint=cpu_sku:Platinum_8628
  #SBATCH --clusters=arc 
   
  #SBATCH --nodes=8
  #SBATCH --time=24:00:00
  #SBATCH --ntasks-per-node=1
  #SBATCH --exclusive
  #SBATCH --mem=384000

  export UCX_IB_MLX5_DEVX=n
  export I_MPI_DEBUG=9
  export FI_PROVIDER=mlx
  export OMP_NUM_THREADS=48

  module load OpenMPI/4.0.3-GCC-9.3.0
  mpirun -np 8 my_executable
  ```
