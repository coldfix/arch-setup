### Overview

This guide describes additional or alternative steps that have to be performed
to install archlinux on a GPT disk when booting in BIOS mode. It assumes that
in all other regards [the main guide](https://github.com/coldfix/arch-setup/blob/master/README.md) is followed.

A GPT-BIOS boot partition is usually not needed for new installations on UEFI
capable devices. However, if you have another OS that requires a legacy BIOS
boot, it might be necessary. For example, if you had already installed another
OS on your hard drive without paying attention to the boot mode and now you
bought a new HDD where you want to install linux (good for you).


### Host OS

##### EFI

Once the root console is up, check whether you have booted in UEFI mode:

    ls /sys/firmware/efi/efivars

If there is something here, you are booted in EFI mode. You probably don't
need the steps described here. Head back to [the main guide](https://github.com/coldfix/arch-setup/blob/master/README.md).


##### Partitioning

Instead of creating an ESP as in the main guide, you have to setup a BIOS-GPT
partition like this:

    gdisk /dev/sda

      Create new partition table:     o
      Switch to 1-byte alignment:     x L 1 m
      Create BIOS boot partition:     n 1 34 2047 ef02
      Revert partition alignment:     x L 2048 m
      Create LVM partition:           n . . . 8e00
      Write and exit to shell:

* [Partitioning](https://wiki.archlinux.org/index.php/Partitioning)
* [GRUB/GPT specific instructions](https://wiki.archlinux.org/index.php/GRUB#GUID_Partition_Table_.28GPT.29_specific_instructions) (for an explanation of step 2,3,4)


##### Setup LVM

Additional volumes:

    lvcreate lunix -n boot -L 512M


#### Prepare filesystems

Additional filesystems:

    mkfs.ext4 /dev/lunix/boot -L boot
    mount /dev/lunix/boot /mnt/boot


### Guest OS

##### Create initial ramdisk

Additional `MODULES`:
- `dm_mod`: for `/boot` on LVM
- `ext4`: for the ext4 `/boot`


##### Install grub

(extra kernel parameter `dolvm`?)

Use this command to install grub instead:

    grub-install /dev/sda

(you may also need `--target=i386-pc`?)

Note that you don't need `efibootmgr` in this case.
