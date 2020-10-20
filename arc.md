ARC Supercomputers
==================

[Advanced Research Computing](https://www.arc.ox.ac.uk/) (ARC) is an Oxford-wide HPC facility.
It is divided into `ARCUS-B` and `ARCUS-HTC` clusters, with additional facilities in a developmental `ARCUS-HTC Dev` cluster. All systems run Linux.
All researchers of Oxford University have access to the hardware listed [here](https://www.arc.ox.ac.uk/arc-systems), via a credit system.

Members of Simon Benjamin's QTechTheory can make use of privileged access (credit-free) of the following hardware:

- [[info](arcusB.md)] CPU nodes on `ARCUS-B`
- [[info](arcusHTCdev)] CPU and GPU nodes on `ARCUS-HTC-Dev`
- [[info](bigchungus.md)] "Big Chungus" - a 6TB CPU node on `ARCUS-HTC`.

## Access

All clusters are accessed with the same ARC credentials, the username for which will match your Oxford [Single Sign-On](https://help.it.ox.ac.uk/oxford-username-and-sso) (e.g. `corp1234`). Your ARC password can be managed on the `myaccount` arc server:
```
ssh [NAME]@myaccount.arc.ox.ac.uk
``` 
and running `passwd`.

We recommend setting up passphrase-free access for these clusters, as detailed [here](remotetips.md#ssh-without-passphrase).

## Storage

All clusters conveniently share a network file system directly accessible by both submit nodes (from which you submit jobs) and compute nodes (which ultimately run your jobs). A user `[NAME]` has private directories:
- 15GB of storage at:
 ```
 /home/[NAME]/
 ```
 Though this is the default directory (where you'll SSH to), we recommend *not* using it, since jobs will unexpectedly fail if writing to this directory near the storage limit.
 
- 2TB of storage at:
 ```
 /data/oums-quantopo/[NAME]/
 ```
 where `oums-quantopo` is the reservation managed by Simon. We recommend SSH'ing directly into this directory, as detailed [here](remotetips.md#ssh-to-a-specific-directory). 
 
To copy files back and forth between your machine and ARC, we recommend setting up a virtual mount as detailed [here](remotetips.md##mount-remote-drive)

## Submitting jobs

Jobs on all clusters are submitted and scheduled using [SLURM](https://slurm.schedmd.com/documentation.html), which we include an introductory and advanced guide for [here](slurmguide.md).

Environments and utilites, like compilers, can be loaded with the [`module`](https://curc.readthedocs.io/en/latest/compute/modules.html) system.




