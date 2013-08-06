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

*NOTE: Under no circumstances should you execute the files!*

This is meant as an inspiration/guide how to perform an crypt-setup. It is by
no means complete and will not work without user intervention.

File overview

* `host`: basic setup from the installation medium
* `client`: basic setup from within the chroot environment of the new system
* `config`: contains common and target system configuration
* `etc`: the actual configuration files on the destination system
* `map`: terminal keyboard layouts used during installation

You should *not* blindly copy the `/etc` configuration files but instead
carefully compare the differences and adapt them to your needs.

