
Fritz
======

Fritz is a desktop machine (Dell Precision 5820) in the Materials office 40.04.

## Hardware
- 18 core (36 threads) CPU (Intel i9-10980XE, 3GHz, 25MB cache)
- 64GB RAM
- ***two*** NVIDIA Quadro [P5000](https://images.nvidia.com/content/pdf/quadro/data-sheets/192195-DS-NV-Quadro-P5000-US-12Sept-NV-FNL-WEB.pdf) GPUs (each 16GB, 2560 cores)
 - The GPUs are on separate sockets.
- 1TB NVMe drive
- 2TB SATA drive (7200rpm)

## Software
- Ubuntu 20
- Windows 10 (via swappable boot drive)
- Mathematica via network license server `ouit-mathlic.it.ox.ac.uk`

## Access
- SSH to `fritzlab.materials.ox.ac.uk` (IP `129.67.85.63`). This may require being in the materials network, either on-site ethernet, or via VPN server `vpn.ox.ac.uk` (using `oxfordusername@ox.ac.uk`)
- Physically in office 40.04

## Job Submission 

Fritz does *not* use a job scheduler. Instead, users should manually check the current load on the system before launching their jobs. 

Check the CPU load with 
```
htop
```
and the GPU load with
```
nvidia-smi 
```