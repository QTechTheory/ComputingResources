ARCUS-HTC Dev
=============

The `oums-quantopo` group (lead by Simon Benjamin) has privileged access to a `qh-devel` partition in ARCUS-HTC-Dev.

## Hardware

The hardware in the `qh-devel` partition includes:

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

These nodes live in `arcus-htc-dev`, accessible from an internal ARC network `oscgate.arc.ox.ac.uk`. Access from outside of the Oxford University network may first require VPN to server `vpn.ox.ac.uk` (using `oxfordusername@ox.ac.uk`). 

Access using your regular ARC account `[USER]` by two tunnels:

```
ssh [USER]@oscgate.arc.ox.ac.uk
```
then
```
ssh [USER]@arcus-htc-dev
```

This is the level from which jobs should be submitted. 

Though it is permitted on the development node, one should **not** SSH directly to a compute node, unless they have secured the node via SLURM. The compute nodes are accessible from within `arcus-htc-dev` via
```
ssh [USER]@arcus-htc-node[201-245]
ssh [USER]@arcus-htc-gpu[201-208]
```
One can check the CPU load of a node using `top`, and the GPU load via `nvidia-smi`.

## Job Submission

From `arcus-htc-dev`, one can submit [SLURM](https://slurm.schedmd.com/documentation.html) jobs to the CPU or GPU nodes. See [here](slurmguide.md) for a general guide on submitting and parallelising jobs. Below describes only the specifics to ARCUS-HTC-Dev.

#### CPU

- To request a single CPU core for an hour, create submission script `submit.sh` with contents:
 ```
 #!/bin/bash
 
 #SBATCH --account=oums-quantopo
 #SBATCH --partition=qh-devel 
 
 #SBATCH --nodes=1
 #SBATCH --time=1:00:00
 
 ./my_executable
 ```
 and submit it with `sbatch submit.sh`.
 > Note that (unlike ARCUS-B), this job runs on a single CPU core despite `--nodes=1`; the remaining cores (and possibly GPU resources) remain available for other jobs.

- To request e.g. 10 cores, add
 ```
 #SBATCH --ntasks-per-node=10
 ``` 
See [here](slurmguide.md) for how to use these cores to parallelise tasks.
- To reserve the entire node and use all **96** hardware threads, use 
 ```
 #SBATCH --exclusive
 ```
 
#### GPU

- To request a GPU, merely add `--gres=gpu:1` to the CPU script:
 ```
 #!/bin/bash
 
 #SBATCH --account=oums-quantopo
 #SBATCH --partition=qh-devel 
 
 #SBATCH --nodes=1
 #SBATCH --gres=gpu:1
 #SBATCH --time=1:00:00
 
 ./my_executable
 ```
 This will reserve a single CPU core and a single GPU.

- To reserve both GPUs on a node, use 
 ```
 #SBATCH --gres=gpu:2
 ```
- To reserve an entire GPU node (all cores, both GPUs), use:
 ```
 #SBATCH --gres=gpu:2
 #SBATCH --exclusive
 ```
 
 


#### Specific node

One can submit to a specific node, so that for example they could directly SSH into the node during the job, via:
 ```
 #SBATCH --nodelist=arcus-htc-node201
 ```
The CPU nodes are available at `arcus-htc-node[201-245]`, and the GPU nodes at `arcus-htc-gpu[201-208]`. 


## Job Restrictions

As of 19th Oct 2020:
- There is  **no** `time` restriction.
- All `45 + 8` nodes can be requested by a single job.