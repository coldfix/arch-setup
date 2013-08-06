# Arch-Setup

## Overview

### Features

* GPT-BIOS boot partition
* LVM on LUKS
* encrypted root `/` and `/home` partition
* separate non-encrypted `/usr` partition

### Flaws

* A key for `/home` is stored on `/` in order
* `/usr` is not encrypted
* use of `/dev/urandom`
* the encryption is only as safe as the chosen password

## Usage

**NOTE: This is NOT meant for unattended execution!**

This is meant as an inspiration/guidance how to perform an crypt-setup. It is
by no means complete and will not work without user intervention.

File overview:

* `host`: basic setup from the installation medium
* `client`: basic setup from within the chroot environment of the new system
* `config`: contains common and target system configuration
* `etc`: the actual configuration files on the destination system
* `map`: terminal keyboard layouts used during installation

You should NOT blindly copy the `/etc` configuration files but instead
carefully compare the differences and adapt them to your needs.

## References

Make sure you read the following archlinux resources before and/or during
the installation process:

* the official [Installation Guide](https://wiki.archlinux.org/index.php/Official_Arch_Linux_Install_Guide) and the [Beginner's Guide](https://wiki.archlinux.org/index.php/Beginners'_Guide)
* [dm-crypt with LUKS](https://wiki.archlinux.org/index.php/Dm-crypt_with_LUKS)
* [LVM](https://wiki.archlinux.org/index.php/LVM)
* [Partitioning](https://wiki.archlinux.org/index.php/Partitioning)
* [mkinitcpio](https://wiki.archlinux.org/index.php/Mkinitcpio) and especially [/usr as a separate partition](https://wiki.archlinux.org/index.php/Mkinitcpio#.2Fusr_as_a_separate_partition)

