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
    
8. Restart Igor via: 
    ```bash
    sudo shutdown -r now
    ```

9. Inform the new users of their default password, and instruct them to immediately SSH to their new accounts, and change their passwords via `passwd`.


## Installing Mathematica

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

## Setting up QuESTlink

It is helpful to pre-prepare multithreaded and GPU-accelerated QuESTlink executable on the workstations, for users to use from their [remote kernels](remotemmakernel.md).

TODO:


must compile to here:

```Mathematica
Import["https://quest.qtechtheory.org/QuEST.m"]
CreateLocalQuESTEnv["/questlink/serial"];
```
replacing `serial` with `multi` or `gpu` for multithreaded and GPU-accelerated modes respectively.

> rationale is top of filesystem makes it platform agnostic