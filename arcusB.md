ARCUS-B
=======

The `oums-quantopo` group (lead by Simon Benjamin) has privileged access to the `nqit` reservation in ARCUS-B.

## Hardware

The `nqit` reservation includes `112` of the nodes listed [here](https://www.arc.ox.ac.uk/arc-systems). These include:

- 246 **CPU nodes** with:
 * 16 core CPU (Intel Haswell, 1.2GHz, 20MB cache)
 * 64GB RAM
 
- 92 **CPU nodes** with:
 * 16 core CPU (Intel Haswell, 1.2GHz, 20MB cache)
 * 128GB RAM

- 9 **CPU nodes** with:
 * 16 core CPU (Intel Haswell, 1.2GHz, 20MB cache)
 * 256GB RAM


> ARCUS-B includes the following nodes *outside* the `nqit` reservation. 
> These might no longer be exempt from billing.
>
> - 33 **CPU nodes** with:
>  * 20 core (40 threads) CPU (Intel Broadwell, 1.2GHz, 26MB cache)
>  * 128GB RAM
> 
> - 3 **CPU nodes** with:
>  * 20 core  (40 threads) CPU (Intel Broadwell, 1.2GHz, 26MB cache)
>  * 256GB RAM

## Access

These nodes live in `arcus-b.arc.ox.ac.uk`. Access from outside of the Oxford University network may first require VPN to server `vpn.ox.ac.uk` (using `oxfordusername@ox.ac.uk`). 

Access using your regular ARC account `[USER]`:

```
ssh [USER]@@arcus-b.arc.ox.ac.uk
```

## Job Submission

Jobs are submitted using [SLURM](https://slurm.schedmd.com/documentation.html).
See [here](slurmguide.md) for a general guide on submitting and parallelising jobs.
Below are details specific to ARCUS-B.

- All submission scripts to ARCUS-B should include:
 ```
 #SBATCH --account=oums-quantopo
 #SBATCH --reservation=nqit
 ```
 Here's an example of a `1` hour job running on a single `64GB` node (reserving all `16` cores):
 ```
 #!/bin/bash
 #SBATCH --account=oums-quantopo
 #SBATCH --reservation=nqit

 #SBATCH --job-name=myjob
 #SBATCH --time=1:00:00
 #SBATCH --nodes=1

 ./my_application
 ```

- The larger RAM nodes can be requested by specifying `mem`, though one must be careful to specify an amount *less* than that of the node. To request `128GB`, use:
 ```
 #SBATCH --mem=100GB
 ```
 To request `256GB`, use
 ```
 #SBATCH --mem=200GB
 ```
 
- The higher core-count (20 core Broadwell) nodes can be requested using `ntasks-per-node`, and removing the `reservation` flag.
 ```
 ## SBATCH --reservation=nqit
 #SBATCH --ntasks-per-node=20
 ```

## Job Restrictions

As of 19th Oct 2020:
- The maximum allowed `time` is `120` hours (`120:00:00`).
- The maximum allowed `nodes` is `25`.

Jobs requesting longer times or more nodes will be enqueued, but will not run (`squeue` will report `PartitionTimeLimit` or `AssocGrpNodeLimit`). An admin can later permit these to begin running.

## Notes
- MPI jobs on ARCUS-B require a `. enable_arcus-b_mpi.sh` command to be called in the submission script (see [example](slurmguide.md#submitting-a-distributed-job)).
- Distributed jobs on ARCUS-B may suffer from anomalously high latency due to an unresolved fabric problem (since 2017).

## Guides
- See [here](https://gist.github.com/TysonRayJones/5d31bfd5dbf6d8417e345f32225db712) to install [ProjectQ](https://projectq.ch/).
- See [here](https://gist.github.com/TysonRayJones/af7bedcdb8dc59868c7966232b4da903) to install [GSL](https://www.gnu.org/software/gsl/).