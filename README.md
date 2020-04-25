# unlock-cryptroot

Script to unlock an Ubuntu or Debian encrypted root filesystem via
ssh.

To unlock the target system (after initial setup; see below):
  1. Run the following from your workstation:
     ```sh
     unlock-cryptroot target.example.com
     ```
  2. Enter the password to your ssh key (if there is one).
  3. Enter the drive encryption password(s).

For additional options and default file locations, run:
```sh
unlock-cryptroot --help
```

Tested on:
  * Ubuntu 16.04 (Xenial)
  * Ubuntu 15.10 (Wily)
  * Ubuntu 14.04 (Trusty)

Relevant bug reports:
  * [Ubuntu bug #595648](https://bugs.launchpad.net/bugs/595648)
  * [Debian bug
    #782024](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=782024)

## Setup Instructions

Initial setup on the target machine (the machine you want to unlock
from remote):

  1. Boot the target machine
  2. Install dropbear into the initramfs:
       * For 16.04 (Xenial): `sudo apt-get install dropbear-initramfs`
       * For 15.10 (Wily) and older: `sudo apt-get install dropbear`
  3. If you are running 15.10 (Wily) or older, or if you are running
     16.04 (Xenial) and you wish to use a non-default IP address or
     device, set the [`ip=` kernel boot
     parameter](https://www.kernel.org/doc/Documentation/filesystems/nfs/nfsroot.txt):
       1. Edit `/etc/default/grub`
       2. Add your `ip=` parameter to the `GRUB_CMDLINE_LINUX` variable
       3. Save your changes
       4. Run `sudo update-grub` to install the changes
  4. For 16.04 (Xenial) only: Prepare keys for public key authentication
     (this is already done for you on 15.10 (Wily) and older):
       1. Generate an ssh key pair for logging in to the initramfs:
          ```sh
          sudo sh -c '(umask 0077 && mkdir -p /etc/initramfs-tools/root/.ssh)'
          sudo ssh-keygen -t rsa -b 4096 -o -a 100 \
              -f /etc/initramfs-tools/root/.ssh/id_rsa
          ```
       2. Add the public key to the initramfs's `authorized_keys`:
          ```sh
          sudo cp /etc/initramfs-tools/root/.ssh/id_rsa.pub \
              /etc/initramfs-tools/root/.ssh/authorized_keys
          ```
       3. Update the initramfs:
          ```sh
          sudo update-initramfs -u
          ```
  5. For 15.10 (Wily) and older (fixed in 16.04 (Xenial)):  The kernel
     `ip=` parameter conflicts with the system's normal networking
     configuration, so you must set up a script to deconfigure the
     interface after the drive is unlocked but before the normal
     networking configuration is applied.  See the example `etc_*`
     files.

Initial setup on your workstation (the machine you will be sitting at
when you unlock the target machine):

  6. Copy the ssh private key for the target machine's initramfs to
     the machine that will be doing the remote unlocking:
     ```sh
     T=target.example.com # change as necessary
     scp root@"$T":/etc/initramfs-tools/root/.ssh/id_rsa \
         ~/.ssh/id_rsa.initramfs_"$T"
     ```
