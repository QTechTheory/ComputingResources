
Victor
======

Victor is a desktop machine (Dell Precision 5820) in the Materials office 40.10.

## Hardware 
- 6 core (12 threads) CPU (Intel Xeon W-2133, 3.6GHz, 8MB Cache)
- 32GB RAM
- NVIDIA Quadro [P6000](https://www.nvidia.com/content/dam/en-zz/Solutions/design-visualization/documents/Quadro-P6000-US-12Sept.pdf) GPU (24GB, 3840 cores)
- 1TB NVMe drive
- 256GB NVMe drive (Dell ultra-speed)

## Software
- Ubuntu 18
- Windows 10 (alternate boot)
- Mathematica via network license server `ouit-mathlic.it.ox.ac.uk`

## Access
- SSH to `victorslab.materials.ox.ac.uk` (IP `129.67.87.96`). This may require being in the materials network, either on-site ethernet, or via VPN server `vpn.ox.ac.uk` (using `oxfordusername@ox.ac.uk`)
- Physically in office 40.10

## Job Submission 

Victor does *not* use a job scheduler. Instead, users should manually check the current load on the system before launching their jobs. 

Check the CPU load with 
```
htop
```
and the GPU load with
```
nvidia-smi 
```