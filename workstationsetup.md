Setting up the office workstations
==================================



## Adding new users

1. Connect (as root) to [Victor](victor.md) and [Fritz](fritz.md) in turn, via:
    ```bash
    ssh victor@victorslab.materials.ox.ac.uk
    ```
    ```bash
    ssh fritz@fritzlab.materials.ox.ac.uk
    ```
    and follow the instructions 2-5 for each.

2. Choose a new default password for the users, replacing `[newpassword]` below. Absolutely ***do not*** copy and paste the below command verbatim.
    ```bash 
    export DEFAULT=[newpassword]
    ```

3. Enter the [Single Sign On](https://help.it.ox.ac.uk/oxford-username-and-sso)'s of the new users in a space separated list, like so (replacing `[example123]` with e.g. `wolf9999`):
    ```bash
    export NEWUSERS=( [example123] [example456] [example789] )
    ```

4. Run the following command to create each user as **sudo** with the same default password as above:
    ```bash
    for USER in ${NEWUSERS[*]}; do sudo useradd -m $USER; sudo usermod -aG sudo $USER; sudo usermod --password $(openssl passwd -1 $DEFAULT) $USER; done
    ```
    This will prompt you to enter the root password for the `victor` and `fritz` accounts.
    
    For clarity, this command is one-line of the following:
    ```bash 
    for USER in ${NEWUSERS[*]}
    do
        # create the user and a home directory for them (-m)
        sudo useradd -m $USER
        
        # elevate the user to sudo
        sudo usermod -aG sudo $USER
        
        # set the user's password to $DEFAULT
        sudo usermod --password $(openssl passwd -1 $DEFAULT) $USER
    done
    ```

5. Connect (as root) to [Igor](igor.md), via:
    ```bash
    ssh igor@@igor-gpu.materials.ox.ac.uk
    ```
    
6. Set the same environment variables as before:
    ```bash
    export DEFAULT=[newpassword]
    export NEWUSERS=( [example123] [example456] [example789] )
    ```

7. Run the following command to create each user as an **admin** acount with the default password as above:
    ```bash
    for USER in ${NEWUSERS[*]}; do export MAXID=$(dscl . -list /Users UniqueID | awk 'BEGIN { max = 500; } { if ($2 > max) max = $2; } END { print max + 1; }'); export NEWID=$((MAXID+1)); sudo dscl . -create /Users/$USER; sudo dscl . -create /Users/$USER UserShell /bin/bash; sudo dscl . -create /Users/$USER RealName "$USER"; sudo dscl . -create /Users/$USER UniqueID "$NEWID"; sudo dscl . -create /Users/$USER PrimaryGroupID 80; sudo dscl . -create /Users/$USER NFSHomeDirectory /Users/$USER; sudo dscl . -passwd /Users/$USER $DEFAULT; sudo dscl . -append /Groups/admin GroupMembership $USER; sudo mkdir /Users/$USER; done
    ```
    This will prompt you to enter the root password for `igor`.
    
    For clarity, this command is a one-line of the following (in a clearer order):
    ```bash
    for USER in ${NEWUSERS[*]}
    do
        # find a unique id for the new user
        export MAXID=$(dscl . -list /Users UniqueID | awk 'BEGIN { max = 500; } { if ($2 > max) max = $2; } END { print max + 1; }')
        export NEWID=$((MAXID+1))
        
        # create the new user
        sudo dscl . -create /Users/$USER
        sudo dscl . -create /Users/$USER UserShell /bin/bash
        sudo dscl . -create /Users/$USER RealName "$USER"
        sudo dscl . -create /Users/$USER UniqueID "$NEWID"
        
        # set the user's password to $DEFAULT
        sudo dscl . -passwd /Users/$USER $DEFAULT
    
        # give the user admin privileges
        sudo dscl . -create /Users/$USER PrimaryGroupID 80
        sudo dscl . -append /Groups/admin GroupMembership $USER
        
        # create the user's home directory (first cmd fails?)
        sudo dscl . -create /Users/$USER NFSHomeDirectory /Users/$USER 
        sudo mkdir /Users/$USER
    done
    ```
    
8. Restart Igor via: 
    ```bash
    sudo shutdown -r now
    ```

9. Inform the new users of their default password, and instruct them to immediately SSH to their new accounts, and change their passwords via `passwd`.
    > Immediately!


## Installing Mathematica

> As of Oct 2020, this has already been completed for Fritz, Victor and Igor

Since the office workstations are connected to the materials network, they are all licensed to run Mathematica. The easiest method to install Mathematica is by using the machines physically, and downloading Mathematica like a desktop application.

When prompted for an Activation Key, select `Other ways to activate` then `Connect to a network license server`, and use server name:
```
ouit-mathlic.it.ox.ac.uk
```

This will install Mathematica both as a desktop application, and as a command-line utility and kernel accessible by all user accounts.

Executing code asynch can be done via the `wolframscript` alias, which points to:
- **Victor**:
  ```
  /usr/bin/wolframscript
  ```
- **Fritz**:
  ```
  /usr/bin/wolframscript
  ```
- **Igor**:
  ```
  /usr/local/bin/wolframscript
  ```
  
The Mathematica kernel, needed for running remote kernels on the workstations, is located at:
- **Victor**:
  ```
  /home/Wolfram/Mathematica/11.3/SystemFiles/Kernel/Binaries/Linux-x86-64/WolframKernel
  ```
- **Fritz**:
  ```
  /usr/local/Wolfram/Mathematica/12.1/SystemFiles/Kernel/Binaries/Linux-x86-64/WolframKernel
  ```
- **Igor**:
  ```
  /Applications/Mathematica.app/Contents/MacOS/MathKernel
  ```
  
The kernel file location be found by, from within any Mathematica service (e.g. `wolframscript`, or in a notebook), by running:
```Mathematica
First @ $CommandLine
```


## Setting up compilers

It is helpful to prepare a C/C++ and CUDA compiling environment on the work-stations, needed for using QuESTlink as below. Repeat the below steps for Victor and Fritz (Linux) and Igor (MacOS).

1. Ensure [GNUMake](https://www.gnu.org/software/make/) is installed, via `make --version`. If not, run:
    - Linux:
      ```bash 
      sudo apt-get install build-essential
      ```
    - MacOS using [Homebrew](https://formulae.brew.sh/):
      ```bash 
      sudo brew install make
      ```
      > If `brew` fails, obtain it with:
      > ```bash
      > /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" 
      > ```

2. Ensure a C++ compiler is installed via `g++ --version`. If not, run:
    - Linux: 
      ```bash
      sudo apt install g++
      ```
      > If this fails on Ubuntu, try first running
      > ```bash 
      > sudo add-apt-repository ppa:ubuntu-toolchain-r/test
      > sudo apt-get update
      > ```
    - MacOS:
      ```bash
      brew install gcc
      ```

3. Install additional compiler tools:
    - Linux:
      ```bash
      sudo apt-get install uuid-dev
      ```
      > If this fails, also try `uuid`, `uuid-devel` and `libuuid-devel` in place of `uuid-dev`
      
4. Ensure an NVDIA compiler is installed, via `nvcc --version`. If not, run:
    - Linux:
      ```bash 
      sudo apt install nvidia-cuda-toolkit
      ```
    - MacOS: follow the steps [here](https://gist.github.com/TysonRayJones/29a1f3a4f773dbe3aaae36a65343830b). 
    

## Setting up QuESTlink

> As of Oct 2020, this has already been completed for Fritz, Victor and Igor

It is helpful to pre-prepare multithreaded and GPU-accelerated QuESTlink executable on the workstations, for users to use from their [remote kernels](remotemmakernel.md). This requires the compilers are setup as above.

1. Connect (as root) to [Victor](victor.md), [Fritz](fritz.md) and [Igor](igor.md) in turn, via:
    ```bash
    ssh victor@victorslab.materials.ox.ac.uk
    ```
    ```bash
    ssh fritz@fritzlab.materials.ox.ac.uk
    ```
    ```bash
    ssh igor@igor-gpu.materials.ox.ac.uk
    ```
    and follow the proceeding instructions.

2. Clone the QuESTlink repository to `/questlink/`:
    ```bash
    sudo git clone https://github.com/QTechTheory/QuESTlink.git /questlink
    ```
    ```bash 
    cd /questlink
    ```

3. Compile QuESTlink in serial, multithreaded and GPU-accelerated modes (as illustrated [here](https://github.com/QTechTheory/QuESTlink/blob/master/Doc/README.md)):
    - Fritz and Victor:
      ```bash
      sudo make EXE=serial OS=LINUX COMPILER_TYPE=GNU MULTITHREADED=0
      sudo make clean
      ```
      ```bash
      sudo make EXE=multi OS=LINUX COMPILER_TYPE=GNU MULTITHREADED=1
      sudo make clean
      ```
      ```bash
      sudo make EXE=gpu OS=LINUX COMPILER_TYPE=GNU MULTITHREADED=0 GPUACCELERATED=1 GPU_COMPUTE_CAPABILITY=61
      sudo make clean
      ```
      Note this final command assumed the workstation's GPU has a compute-capability of `6.1`. For new workstations, check the compute-capability [here](https://developer.nvidia.com/cuda-gpus) and modify the value above.
      > If these commands fail, try including `COMPILER=g++-8` or another installed compiler
    
    - Igor:
      ```bash
      sudo make EXE=serial OS=MACOS COMPILER_TYPE=CLANG MULTITHREADED=0
      sudo make clean
      ```
      ```bash
      sudo make EXE=multi OS=MACOS COMPILER_TYPE=CLANG MULTITHREADED=1
      sudo make clean
      ```
      ```bash 
      sudo make EXE=gpu COMPILER=clang-3.9 OS=MACOS COMPILER_TYPE=CLANG GPUACCELERATED=1 GPU_COMPUTE_CAPABILITY=61
      sudo make clean
      ```
      Note in this final command, we assumed `clang-3.9` was compatible with the installed `nvcc` version, and that the workstation's GPU has a compute-capability of `6.1`. For new workstations, compiler compatibility [here](https://gist.github.com/ax3l/9489132), and check the compute-capability [here](https://developer.nvidia.com/cuda-gpus) and modify the value above.
      > If compilation fails in GPU mode, try manually removing `-framework Foundation` from `/questlink/makefile`.

4. Test the compilation by running each of the compiled commands, each type using `ctrl C` to escape the (successful) `Create link` prompt:
    ```bash
    ./serial 
    ```
    ```bash
    ./multi 
    ```
    ```bash
    ./gpu 
    ```

5. Copy and rename the module:
    ```bash
    sudo cp Link/QuESTlink.m link.m
    ```
    

Mathematica sessions on the workstation (locally, or via a remote kernel) can use these QuESTlink environments via:

```Mathematica
Import["/questlink/link.m"]
CreateLocalQuESTEnv["/questlink/serial"];
```
replacing `serial` with `multi` or `gpu` for multithreaded and GPU-accelerated modes respectively.

