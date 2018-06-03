---
layout: post
title: OpenSuse Leap 42.3 won't boot after latest udev update
category: OpenSuse
tags: [OpenSuse, Linux, LUKS, NVMe]
---

## Prolog 

On 28.05.2018 I was happily informed that new updates are available for my favorite
OpenSuse distribution. I was not thinking too much and just executed:
```shell sudo zypper up``` as usual. Unfortunately this does not end up well.
I've noticed that there was a new kernel released, I've decided to reboot my workstation
to start using latest released kernel.
I've restarted my machine, it won't boot anymore. Probably you wont 
have this problem if you are a regular OpenSuse user. My configuration is very specific: because it's 
my work laptop its required to have encrypted disks, which in this case happens to be NVMe SSD.


## Symptoms
Normally during the boot process user is asked about password and if it's correct boot continues as
usual. This time password dialog was not started by systemd.


## The cause

The reason for this is that when udev is updated sometimes disk-id (```/dev/disk/by-id/*```) generation scheme
changes and if you have disk encryption enabled your encrypted disk-id is stored in /etc/cryptab and entry
in this file is not updated. See screenshot taken from emergency shell:
![Emergency shell]({{ "/images/emergency_shell.png" | absolute_url }}) 


## Fix

Here is a short instruction step by step howto fix unbootable system:

1. Wait for emergency shell(~2-3 minutes in my case).

2. Decrypt *all* encrypted drives/partitions (you quite possibly only have one) that are part of your system. The lsblk command may be helpful, or you may find these under paths like /dev/nvme*, or it may be easier to look at the symlinks under /dev/disk/by-id/ (be aware there are likely lots of duplicates, pointing to the same drive/partition, in this location). Run cryptsetup for each one:

   cryptsetup open <device> <some-unique-name>

   For example:

   ```cryptsetup open /dev/nvme0n1p2 name1```

    <some-unique-name> needs to be different for each device, obviously. 

    Enter the normal passphrase for each device when prompted.

3. Mount decrypted filesystems
   * If you are using LVM, run:
     ```
     lvm_scan
     lvm lvscan
     lvm vgchange -ay
     lvm lvdisplay
     ```
     
     Verify you can see all your logical volumes.

     Note the "LV Path" of each logical volume.

     Mount the logical volume that corresponds to the / filesystem for your system on /mnt. It may have an "LV Name" of root, or it may not. Use the "LV Path" to mount it:

     mount <lv-path> /mnt

     For example:

     ```
      mount /dev/system/root /mnt
     ```

   * If you ARE NOT using LVM, mount the drive/partition that corresponds to the / filesystem for your system on /mnt:

     mount \<device\> /mnt

      For example:
     ```
      mount /dev/nvme0n1p2 /mnt
     ```
4. If you have a very unusual setup where /etc is on a separate filesystem from /, mount it:

   mount <device/lv-path> /mnt/etc

   For example:
   ```
   mount /dev/system/etc /mnt/etc
   ```
5. Prepare and enter a chroot environment for your system:
   ```
   mount -t proc none /mnt/proc
   mount --rbind /dev /mnt/dev
   mount --rbind /sys /mnt/sys
   chroot /mnt /bin/bash
   ```
6. Mount the remaining filesystems (if any) using information from your system's /etc/fstab file:

   ```
   mount -a
   ```
   It will mount your boot partition which is required for this fix to work.

7. Edit `/etc/crypttab` to fix the entry that is no longer created by udev. In this case NVMe disk id naming convention changed from nvme-20 to nvme-eui.0.
  So I had to replace:
  `/dev/disk/by-id/nvme-20xxxxxxx`

   With: `/dev/disk/by-id/nvme-eui.0xxxxx`

8. (May be optional) I'm not sure if this is required but I've executed /usr/lib/systemd/system-generators/systemd-cryptsetup-generator

9. Regenerate the initramfs boot images with dracut.

   ```
   dracut -f
   ```
10. Exit from the chroot into your system, and reboot:
   ```
   umount -a
   exit
   reboot
   ```
   

## Resources:
* [https://bugzilla.opensuse.org/show_bug.cgi?id=1063249](https://bugzilla.opensuse.org/show_bug.cgi?id=1063249)
* [https://bugzilla.opensuse.org/show_bug.cgi?id=904987](https://bugzilla.opensuse.org/show_bug.cgi?id=904987)
* [https://bugzilla.suse.com/show_bug.cgi?id=1095096](https://bugzilla.suse.com/show_bug.cgi?id=1095096)
* [https://doc.opensuse.org/documentation/leap/startup/html/book.opensuse.startup/cha.trouble.html#sec.trouble.data.recover.rescue.access](https://doc.opensuse.org/documentation/leap/startup/html/book.opensuse.startup/cha.trouble.html#sec.trouble.data.recover.rescue.access)
