
QTechTheory Computing Resources
===============================

This repo documents the computing facilities available to members of Simon Benjamin's Quantum Technology Theory Group, and how to use them.

## Facilities

The QTechTheory group has privileged access to the [Advanced Research Computing](https://www.arc.ox.ac.uk/) (ARC) supercomputing facilities, as detailed [here](arc.md). These are accessed with ARC credentials, and employed using the SLURM job scheduler, as detailed [here](slurmguide.md).

The group also has access to several desktops located in the Materials offices, with group-managed accounts, and which allow direct (even physical) access and job launching.
These include 
- [Victor](victor.md) (Linux, 12C CPU, 24GB GPU, 32GB RAM)
- [Fritz](fritz.md) (Linux, 36C CPU, *two* 16GB GPUs, 64GB RAM)
- [Igor](igor.md) (MacOS, 8C CPU, 24GB eGPU, 16GB RAM)

While these desktops can be accessed directly, they are particularly convenient as workstations for remote Mathematica kernels, accessed through notebooks on your own machine. We detail how to set this up [here](remotemmakernel.md).

## Links
In total, this repo contains:
- [[link](arc.md)] Info about ARC, and the group's privileged access to:
 - [ARCUS-B](arcusB.md)
 - [ARCUS-HTC Dev](arcusHTCdev.md)
 - [Big Chungus](bigchungus.md)
- [[link](slurmguide.md)] A guide on using ARC's Job Scheduler, SLURM.
- [[link](remotetips.md)] Tips for accessing these facilities remotely
- Info about the office workstations:
 - [Victor](victor.md)
 - [Fritz](fritz.md)
 - [Igor](igor.md)
- [[link](remotemmakernel.md)] A guide for using these workstations as remote Mathematica kernels.