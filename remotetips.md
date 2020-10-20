Tips for Remote Sessions
========================

* [SSH to a specific directory](#ssh-to-a-specific-directory)
* [SSH without passphrase](#ssh-without-passphrase)
* [Mount remote drive](#mount-remote-drive)

## SSH to a specific directory
> Linux and MacOS

To SSH directly into a target remote directory `[DIR]`, run:
```bash
ssh [USER]@[ADDRESS] "cd [DIR]; bash"
```
It is convenient to save this as a bash script to run each time when connecting. For example, I access [ARCUS-B](arcusB.md) via a script `~/arcusb_connect.sh` with contents:
```bash
ssh corp2627@arcus-b.arc.ox.ac.uk "cd /data/oums-quantopo/corp2627 ; bash"
```
which is previously given permission to run via `chmod +x arcusb_connect.sh`

## SSH without passphrase
> Linux and MacOS

Punching in your password every time you SSH to the same machine can be a chore. We *have* RSA keys, let's use them! Here we'll setup passphrase-free SSH. This is sometimes necessary for running Mathematica kernels on remote machines, like [Victor](victor.md) and [Fritz](fritz).

> All the below commands are to be entered in your **local** machine's terminal.

1. Ensure you have an RSA key:
    ```bash
    ssh-keygen -t rsa
    ```
    Do not overwrite your existing key if prompted (hit **n**).
 
2. Add your key to the remote machine (`USER@ADDRESS`):
    ```bash
    cat ~/.ssh/id_rsa.pub | ssh USER@ADDRESS 'cat >> .ssh/authorized_keys'
    ```

3. Add the key to the SSH agent:
    ```bash
    ssh-add -K ~/.ssh/id_rsa
    ```

4. If you're on **MacOS**, you may additionally need to permit the SSH agent to use the key chain. Do this by editing `~/.ssh/config` to feature:
    ```bash
    Host *
        UseKeychain yes
    ```

5. You should now be able to SSH straight into the remote machine without a password prompt!
    ```bash
    ssh USER@ADDRESS
    ```


## Mount remote drive
> MacOS

Copying files back and forth with a remote machine can be a pain. *Mounting* the remote drive will make it appear as a regular directory on your client machine, so you can edit remote files in software as if they were local.

> All the below commands are to be entered in your **local** machine's terminal.

1. Ensure you have [Homebrew](https://brew.sh/) installed:
    ```bash
    ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"
    ```

2. Install osxfuse from Homebrew:
    ```bash
    brew cask install osxfuse
    ```

    3. Restart your computer
    > I know I know, grr.

4. Install sshfs from Homebrew:
    ```bash
    brew install sshfs
    ```

At this point, you can now use `sshfs` to create remote mounts. However, it's best to create a script to activate the mount, which will be deactivated each time you lose internet or shutdown your machine. Here's how:

5. Create a new `MountDir` directory on the desktop (or elsewhere) which will become the remote filesystem:
    ```bash
    mkdir ~/Desktop/MountDir
    ```
    This folder must exist when you attempt to activate the remote mount, so don't delete it!

6. Create a `mounter.sh` script (via `sudo nano ~/mounter.sh`) with contents:
    ```bash
    sudo sshfs -o allow_other,defer_permissions USER@ADDRESS:REMOTEDIR ~/Desktop/MountDir
    ```
    where `USER@ADDRESS` is your usual SSH target info, and `REMOTEDIR` is the directory on the remote machine to be mounted locally; this should be the root of your personal directory on the remote machine. You will only be able to access files/folders within this directory through the remote mount.
    > For example, *my* command to mount my drive on [Victor](victor.md) is
    > ```bash
    > sudo sshfs -o allow_other,defer_permissions tyson@victorslab.materials.ox.ac.uk:/home/tyson ~/Desktop/MountDir
    > ```
    To exit nano, hit keys **control x** then **y** then **Enter**.

7. Give the `mounter.sh` script permissions to run:
    ```bash
    chmod +x ~/mounter.sh
    ```

The setup is complete. To **activate** the mount, run:
```bash
~/mounter.sh
```
This will require you enter both your MacOS root password, and your password for the remote machine.
Afterward `MountDir` on your `Desktop` will have changed to `OXFUSE` and contain your remote directory. 

To unmount (not often necessary), run: 
```bash
sudo umount ~/Desktop/MountDir
```

Spontaneous disconnection will sometimes require you to restart your machine in order to run `~/mounter.sh` again.
