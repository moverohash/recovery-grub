

**To add to the answer provided by user @kirill-a and flesh it out a bit more:**

Here is what I did recently to restore the GRUB boot loader on a Windows 8 and Debian 8 dual-boot machine, after a Windows 8 reinstallation cleared the previous GRUB boot loader entry from the beginning of the disk.

**REPAIR GRUB2: Live USB/CD 'chroot' method on Linux**

These instructions apply generally to an unencrypted, non-LVM disk on Debian-based distros. Minor changes are needed in directory names and utilities used under RHEL/SUSE-based and possibly Arch-based distros.

1. **Start with a bootable Live USB or CD of the distro of your choice.**

   - Use `lsblk` to determine the kernel name descriptor (i.e., `/dev/xxyN`) of the block device with a missing or damaged GRUB boot loader.

2. **All the following actions are to be done as root (use `su` or `sudo`).**

   - Create a temporary mount point for the installed Linux:

     ```bash
     mkdir -p /mnt/linux
     ```

     (The `-p` option creates the parent directory `/mnt` if it doesn't already exist.)

   - Using `/dev/xxyN` from the previous `lsblk` command:

     ```bash
     mount /dev/xxyN /mnt/linux
     ```

   - The following command is only necessary if you have a separate `/boot` partition; `/dev/xxyN` here is representing the kernel name descriptor of your `/boot` partition.

     ```bash
     mount /dev/xxyN /mnt/linux/boot
     ```

   - Then:

     ```bash
     mount -t proc none /mnt/linux/proc
     mount -t sysfs sys /mnt/linux/sys
     mount -o bind /dev /mnt/linux/dev
     mount -t devpts pts /mnt/linux/dev/pts
     chroot /mnt/linux /bin/bash
     grep -v rootfs /proc/mounts > /etc/mtab
     grub-install /dev/xxy
     ```

     (Here, `/dev/xxy` = the device name and number on which to install the GRUB boot loader, e.g., `/dev/sda`, not including the root partition number as in `/dev/sda1`.)

   - If you want to make any other changes/customizations to GRUB, now is the time to edit the `/etc/default/grub` file and save.

     ```bash
     grub-mkconfig -o /boot/grub/grub.cfg
     ```

3. **Reboot and verify.**

Note: There are several additional steps to this procedure if your GRUB2 boot loader resides on a Linux system with an LVM LV root and/or an encrypted root volume. Feel free to message me here; I have these additional instructions written down and have applied them successfully several times to an LVM LV on an SSD which contains a root volume encrypted with the kernel dm-crypt module.
