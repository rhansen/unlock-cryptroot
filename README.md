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
  * Ubuntu 14.04 (Trusty)

Relevant bug reports:
  * [Ubuntu bug #595648](https://bugs.launchpad.net/bugs/595648)
  * [Debian bug
    #782024](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=782024)

## Target Setup Instructions

### Ubuntu 16.04 (Xenial Xerus) or newer

  1. Install the dropbear ssh server into the initramfs:
     ```sh
     sudo apt-get install dropbear-initramfs
     ```
  2. If you wish to use a non-default IP address or network device,
     set the [`ip=` kernel boot
     parameter](https://www.kernel.org/doc/Documentation/filesystems/nfs/nfsroot.txt):
       1. Edit `/etc/default/grub`
       2. Add your `ip=` parameter to the `GRUB_CMDLINE_LINUX` variable
       3. Save your changes
       4. Run `sudo update-grub` to install the changes
  3. Prepare keys for public key authentication:
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

### Ubuntu 14.04 (Trusty Tahr) or older

  1. Install dropbear into the initramfs:
     ```sh
     sudo apt-get install dropbear
     ```
  2. Set the [`ip=` kernel boot
     parameter](https://www.kernel.org/doc/Documentation/filesystems/nfs/nfsroot.txt):
       1. Edit `/etc/default/grub`
       2. Add your `ip=` parameter to the `GRUB_CMDLINE_LINUX` variable
       3. Save your changes
       4. Run `sudo update-grub` to install the changes
  3. The kernel `ip=` parameter conflicts with the system's normal
     networking configuration, so you must set up a script to
     deconfigure the interface after the drive is unlocked but before
     the normal networking configuration is applied. See the example
     `etc_*` files.

## Remote Workstation Setup Instructions

  1. Copy the ssh private key for the target machine's initramfs to
     the machine that will be doing the remote unlocking:
     ```sh
     T=target.example.com # change as necessary
     scp root@"$T":/etc/initramfs-tools/root/.ssh/id_rsa \
         ~/.ssh/id_rsa.initramfs_"$T"
     ```
