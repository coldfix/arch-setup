# Arch-Install

## Overview

### Features

* GPT-BIOS boot partition
* LVM on LUKS
* encrypted root `/` and `/home` partition
* separate non-encrypted `/usr` partition

### Flaws

* A key for `/home` is stored on `/` in order
* `/usr` is not encrypted

## Usage

*NOTE: Under no circumstances should you execute the files!*

This is meant as an inspiration/guide how to perform an crypt-setup. It is by
no means complete and will probably not work out of the box.

* `host`: basic setup from the installation medium
* `client`: basic setup from within the chroot environment of the new system

