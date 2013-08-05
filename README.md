# Arch-Install

## Features

* GPT-BIOS boot partition
* LVM on LUKS
* encrypted root `/` and `/home` partition
* separate non-encrypted `/usr` partition

## Flaws

* A key for `/home` is stored on `/` in order
* `/usr` is not encrypted

