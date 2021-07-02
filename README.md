
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

These workstations should use your [Single Sign On](https://help.it.ox.ac.uk/oxford-username-and-sso) credentials (the same as ARC), setup by an admin via [this guide](workstationsetup.md). The admin will privately communicate you a default password, which you should immediately change the first time you connect, via `passwd`.

While these desktops can be accessed directly, they are particularly convenient as workstations for remote Mathematica kernels, accessed through notebooks on your own machine. We detail how to set this up [here](remotemmakernel.md).

## Links
In total, this repo contains:
- [[link](arc.md)] Info about ARC, and the group's privileged access to:
  - [ARCUS-B](arcusB.md)
  - [ARCUS-C](arcusC.md)
  - [Big Chungus](bigchungus.md)
- [[link](slurmguide.md)] A guide on using ARC's Job Scheduler, SLURM.
- [[link](remotetips.md)] Tips for accessing these facilities remotely
- Info about the office workstations:
  - [Victor](victor.md)
  - [Fritz](fritz.md)
  - [Igor](igor.md)
- [[link](remotemmakernel.md)] A guide for using these workstations as remote Mathematica kernels.
- [[link](workstationsetup.md)] A guide for Simon or another admin to create new user accounts on the office workstations, and setup the compiler and questlink environments.

We link also to external resources:
- [[link](https://github.com/TysonRayJones/PythonTools/tree/master/slurm_param_sweeper)] A tool for generating SLURM scripts which sweep parameters
- [[link](https://github.com/TysonRayJones/PythonTools/tree/master/mathematica)] A tool for converting Python data structures to Mathematica
- [[link](https://github.com/TysonRayJones/CTools/tree/master/mathematica)] A tool for converting C data structures to Mathematica
- [[link](https://gist.github.com/TysonRayJones/5d31bfd5dbf6d8417e345f32225db712)] A guide for installing ProjectQ on [ARCUS-B](arcusB.md) (and [ARCHER](https://www.archer.ac.uk/))
- [[link](https://gist.github.com/TysonRayJones/af7bedcdb8dc59868c7966232b4da903)] A guide for installing GSL on MacOS, Linux and [ARCUS-B](arcusB.md)
- [[link](https://gist.github.com/TysonRayJones/29a1f3a4f773dbe3aaae36a65343830b)] A guide for using external NVIDIA GPUs with MacOS
- [[link](https://github.com/QTechTheory/QuESTlink/blob/master/Doc/README.md)] A guide for running QuESTlink
- [[link](https://gist.github.com/TysonRayJones/42d4fb572e97e53309ca9d8b04240622)] A guide for rendering LaTeX-style plots in Mathematica