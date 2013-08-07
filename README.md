# Arch-Setup

## Overview

### Features

* GPT-BIOS boot partition
* LUKS on LVM
* encrypted root `/` and `/home` partition
* separate non-encrypted `/usr` partition

### Flaws

* use of `/dev/urandom` as RNG
* `/usr` is not encrypted
* a key for `/home` is stored in `/etc/key` (for automatic unlocking)
* the encryption is only as safe as the chosen password
* a cryptheader backup is proposed

## Usage

**NOTE: This is NOT meant for unattended execution!**

This is meant as an inspiration/guidance how to perform an crypt-setup. It is
by no means complete and will not work without user intervention.

After booting into the live archlinux installation system, you can start to
execute the commands from `host`. In the chroot environment, execute the
commands from `client`.

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

Additionally, I recommend to have a look at

* [cryptsetup - Security Aspects](http://code.google.com/p/cryptsetup/wiki/FrequentlyAskedQuestions#5._Security_Aspects)
* [The Diceware Passphrase Home Page](http://world.std.com/~reinhold/diceware.html)
* [Cipher benchmark for dm-crypt / LUKS](http://blog.wpkg.org/2009/04/23/cipher-benchmark-for-dm-crypt-luks/)

Overwriting old data:

* [Secure Deletion of Data from Magnetic and Solid-State Memory](https://www.usenix.org/legacy/publications/library/proceedings/sec96/full_papers/gutmann/index.html) if you are really paranoid. On the other hand, in this case this setup is probably nothing for you.
* [Overwriting Hard Drive Data](http://computer-forensics.sans.org/blog/2009/01/15/overwriting-hard-drive-data/) for a more recent analysis on this topic

