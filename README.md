# unlock-cryptroot

Script to unlock an Ubuntu or Debian encrypted root filesystem via
ssh.

Tested on:
* Ubuntu 14.04 (trusty)
* Ubuntu 15.10 (wily)

Relevant bug reports:

* [Ubuntu bug #595648](https://bugs.launchpad.net/bugs/595648)
* [Debian bug #782024](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=782024)

## Instructions

Initial setup on the target machine (the machine you want to unlock
from remote):

1. Boot the target machine
2. Install the `dropbear` package:  `sudo apt-get install dropbear`
3. Edit `/etc/default/grub` and add `ip=dhcp` (or whatever is
   appropriate for your network) to the `GRUB_CMDLINE_LINUX` variable
4. Run `sudo update-grub` to install the changes you made to
   `/etc/default/grub`
5. Because the kernel IP configuration conflicts with your system's
   normal networking configuration, you'll need to set up a script to
   deconfigure the interface after unlocking but before the normal
   networking configuration is applied.  See the example `etc_*`
   files.

Initial setup on your workstation (the machine you will be sitting at
when you unlock the target machine):

6. copy the ssh private key from target machine's initramfs:

   ```sh
   T=target.example.com # change as necessary
   cd ~/.ssh
   scp root@"$T":/etc/initramfs-tools/root/.ssh/id_rsa ./id_rsa.initramfs_"$T"
   ```

To unlock the target system, run the following from your workstation:

```sh
unlock-cryptroot target.example.com
```

and enter your password.
