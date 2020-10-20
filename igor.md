Igor
====

Igor is a Macbook Pro (15 inch, 2016) with a connected external GPU in the Materials office 40.10.

## Hardware
- 4 core (8 threads) CPU (Intel i7-6700HQ, 2.6GHz, 256KB Cache)
- 16GB RAM
- eGPU (AKiTiO node via thunderbolt) NVIDIA Quadro P6000 GPU (24GB)
- 1TB SSD drive

## Software
- macOS High Sierra (v10.13.6)
- Mathematica via network license server `ouit-mathlic.it.ox.ac.uk`

## Access
- SSH to `igorslab.local` from the materials ethernet network.
- SSH to `igor-gpu.materials.ox.ac.uk`. This may require being in the materials network, either on-site ethernet, or via VPN server `vpn.ox.ac.uk` (using `oxfordusername@ox.ac.uk`)
- Physically in office 40.10

## Job Submission 

Igor does *not* use a job scheduler. Instead, users should manually check the current load on the system before launching their jobs. 

Check the CPU load with 
```
htop
```
and the GPU load with
```
nvidia-smi 
```

## Notes
- Igor's MacOS should never be upgraded from High Sierra since later OS are incompatible with NVIDIA GPU drivers.
- Igor's eGPU is occasionally borrowed for connection to other machines. Check whether it's currently connected with `nvidia-smi`.