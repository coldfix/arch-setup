### Overview

Installation guide for an archlinux setup with the following aspects:

* GPT-BIOS boot partition
* LUKS on LVM
* encrypted root `/` and `/home` partition
* separate non-encrypted `/usr` partition (for performance)
* a key for `/home` is stored in `/etc/key` (for automatic unlocking)

A GTP-BIOS boot partition is usually not needed for new installations on UEFI
capable devices. However, if you have another OS that requires a legacy BIOS
boot, it might be necessary. For example, if you had already installed another
OS on your hard drive without paying attention to the boot mode and now you
bought a new HDD where you want to install linux (good for you).

LUKS on LVM with an unencrypted `/usr` partition is not very common approach,
but it may be an option if you worry about performance on an old-ish machine.
This may also be good if you plan to span encrypted partitions over multiple
disks.

* [Cipher benchmark for dm-crypt / LUKS](http://blog.wpkg.org/2009/04/23/cipher-benchmark-for-dm-crypt-luks/)
* [LUKS on LVM](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LUKS_on_LVM)
* [Expanding LVM on multiple disks](https://wiki.archlinux.org/index.php/Dm-crypt/Specialties#Expanding_LVM_on_multiple_disks)

In other cases, LVM on LUKS is the more straightforward approach with your
whole system encrypted!

* [LVM on LUKS](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS)

In addition to following the steps described here, you would be well advised
to read the following resources before and during the installation process:

* [Installation Guide](https://wiki.archlinux.org/index.php/Installation_guide)
* [dm-crypt](https://wiki.archlinux.org/index.php/Dm-crypt)
* [LVM](https://wiki.archlinux.org/index.php/LVM)


### Host OS (live USB)

Boot the installation medium.

##### EFI

Once the root console is up, check whether you have booted in UEFI mode:

    ls /sys/firmware/efi/efivars

If there is nothing here, you are booted in legacy mode. If you do not have
another OS that needs to be booted in BIOS mode, you should reboot, set
"UEFI/Legacy Boot" to "UEFI Only" and disable CSM support in your BIOS menu!


##### Network

If you were plugged in via ethernet during boot, your network should be up
already. Verify:

    ping google.com

Otherwise, list your network interfaces to find the name of your ethernet
interface:

    ip link

And bring up the correct one, e.g.:

    ip link set enp0s25 up
    dhcpcd enp0s25

Check your IP:

    ifconfig

* [Network configuration](https://wiki.archlinux.org/index.php/Network_configuration)


##### SSH (optional)

I find it a lot more convenient to perform the rest of the installation
procedure from an already running machine via SSH (allowing me to copy-paste
commands). This requires a root password and enabling password login:

    passwd
    echo "PasswordAuthentication yes\nPermitRootLogin yes" >> /etc/ssh/sshd_config
    systemctl start sshd

And now, just login into your live system from another PC:

    ssh root@192.168.0.XX

I recommend to use a standard terminal such as `xterm` on the client, otherwise
you may have problems with missing terminfo definitions on the host machine.

* [Install from SSH](https://wiki.archlinux.org/index.php/Install_from_SSH)


##### System clock

Ensure the system clock is accurate:

    timedatectl set-ntp true
    timedatectl set-local-rtc 0

* [Time](https://wiki.archlinux.org/index.php/Time)


##### Partitioning (EFI)

For legacy BIOS boot, skip to the next section.

If you are booted in UEFI mode, create partitions as follows:

    gdisk /dev/sda

      Create new partition table:     o
      Create EFI boot partition:      n . . +512M ef00
      Create LVM partition:           n . . . 8e00
      Write and exit to shell:        w

    mkfs.vfat -F32 -n boot /dev/sda1

The first step erases all existing partitions on the disk. Make absolutely
sure this is what you want or skip it.

* [Partitioning](https://wiki.archlinux.org/index.php/Partitioning)
* [EFI System Partition](https://wiki.archlinux.org/index.php/EFI_System_Partition)


##### Partitioning (BIOS-GPT)

Setup a BIOS-GPT partition:

    gdisk /dev/sda

      Create new partition table:     o
      Switch to 1-byte alignment:     x L 1 m
      Create BIOS boot partition:     n 1 34 2047 ef02
      Revert partition alignment:     x L 2048 m
      Create LVM partition:           n . . . 8e00
      Write and exit to shell:        w

* [Partitioning](https://wiki.archlinux.org/index.php/Partitioning)
* [GRUB/GPT specific instructions](https://wiki.archlinux.org/index.php/GRUB#GUID_Partition_Table_.28GPT.29_specific_instructions) (for an explanation of step 2,3,4)


##### Setup LVM

Setup a physical volume and a volume group named lunix:

    pvcreate /dev/sda2
    vgcreate lunix /dev/sda2

Create logical volumes, the first one is only needed if you follow the BIOS-GPT
branch:

    lvcreate lunix -n boot -L 512M
    lvcreate lunix -n root -L 32G
    lvcreate lunix -n swap -L 8G
    lvcreate lunix -n usr  -L 32G
    lvcreate lunix -n home -l 100%FREE
    vgchange -ay

List your physical volumes, volume groups, and logical volumes:

    pvdisplay
    vgdisplay
    lvdisplay


##### Encrypt LUKS devices

Choose cipher parameters (or leave empty for defaults):

    cipher=(--cipher aes-xts-plain64 --key-size 512 --hash sha512)

Encrypt root and home partitions. We leave /usr unencrypted for performance:

    cryptsetup "${cipher[@]}" -y luksFormat /dev/lunix/root
    cryptsetup "${cipher[@]}" -y luksFormat /dev/lunix/home

About choosing a passphrase:

* [cryptsetup - Security Aspects](https://gitlab.com/cryptsetup/cryptsetup/wikis/FrequentlyAskedQuestions#5-security-aspects)
* [The Diceware Passphrase Home Page](http://world.std.com/~reinhold/diceware.html)

Now unlock the LUKS containers:

    cryptsetup open /dev/lunix/root crypt-root
    cryptsetup open /dev/lunix/home crypt-home

*NOTE:* you should probably encrypt your swap as well…


##### Security (OPTIONAL)

*WARNING:* This may take ages and its usefulness is doubtful.

* [Secure Deletion of Data from Magnetic and Solid-State Memory](https://www.usenix.org/legacy/publications/library/proceedings/sec96/full_papers/gutmann/index.html) if you are really paranoid. On the other hand, in this case this setup is probably nothing for you.
* [Overwriting Hard Drive Data](http://computer-forensics.sans.org/blog/2009/01/15/overwriting-hard-drive-data/) for a more recent analysis on this topic

Make used space on encrypted partitions indistinguishable (if paranoid, do
this with a temporary encrypted partition):

    dd if=/dev/zero of=/dev/mapper/crypt-root bs=4M
    dd if=/dev/zero of=/dev/mapper/crypt-home bs=4M

Delete old data on non-encrypted partitions (need only if drive was used
before):

    dd if=/dev/zero of=/dev/lunix/boot
    dd if=/dev/zero of=/dev/lunix/usr

You may use the following function from another console (Ctrl+Alt+F2) to get
continuous status updates every 10s on the main console:

    dd_info() {
        while pkill -USR1 '^dd$'; do
            sleep 10s
        done
    }


##### Prepare filesystems

Create filesystems:

    mkfs.ext4 /dev/mapper/crypt-root -L root
    mkfs.ext4 /dev/mapper/crypt-home -L home -m0
    mkfs.ext4 /dev/lunix/boot -L boot
    mkfs.ext4 /dev/lunix/usr  -L usr
    mkswap    /dev/lunix/swap

I'm using `-m0` for home, since I'm greedy and there should be no need to
reserve space for the root user here.

Mount filesystems:

    mount  /dev/mapper/crypt-root /mnt
    mkdir -p /mnt/boot
    mkdir -p /mnt/home
    mkdir -p /mnt/usr
    mount  /dev/mapper/crypt-home /mnt/home
    mount  /dev/lunix/boot        /mnt/boot
    mount  /dev/lunix/usr         /mnt/usr
    swapon /dev/lunix/swap


##### Unlock home partition automatically

Add key to home (so it can be automounted):

    mkdir /mnt/etc/key -p
    dd of=/mnt/etc/key/home if=/dev/urandom bs=1024 count=4
    cryptsetup luksAddKey /dev/lunix/home /mnt/etc/key/home
    echo "crypt-home      /dev/lunix/home     /etc/key/home" >>/mnt/etc/crypttab

Make sure the file is accessible with root privileges only!

    chmod 0700 /mnt/etc/key
    chmod 0400 /mnt/etc/key/home

* [crypttab](https://wiki.archlinux.org/index.php/Dm-crypt/System_configuration#crypttab)


##### Install and enter guest OS

Install base system:

    pacstrap /mnt base vim zsh
    genfstab -U /mnt >> /mnt/etc/fstab

Edit `/mnt/etc/fstab` and set `passno` to 0 (the last number in the line) for
the `/usr` partition (recommended if you have `/usr` as separate partition):

    vim /mnt/etc/fstab

You may also want to add a `noatime` option for all filesystems on an SSD, e.g.:

    UUID=...   /usr       ext4        rw,noatime,relatime,data=ordered    0 0

Now you can finally chroot into the new OS:

    arch-chroot /mnt zsh


### Guest OS (newly installed)

Before we forget, set a new root password:

    passwd

Change shell:

    chsh -s /usr/bin/zsh

Enable `[multilib]` repository:

    vim /etc/pacman.conf
    pacman -Syy


##### Locale

Edit `/etc/locale.gen` and uncomment all lines with locales you want to
generate, e.g.:

    vim /etc/locale.gen
    locale-gen

Set the default locale:

    echo "LANG=en_US.utf-8" > /etc/locale.conf

Set the time zone:

    ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime

Set a host name:

    echo lunix > /etc/hostname

Download and use your favorite console keyboard layout:

    base=https://raw.githubusercontent.com/untasty/keyboard/master/
    wget $base/usr/share/kbd/keymaps/c++.map -O /usr/share/kbd/keymaps/c++.map
    echo "KEYMAP=c++" >> /etc/vconsole.conf


##### Create a non-root user

Create a powerful non-root user:

    NOROOT=thomas

    useradd $NOROOT -m -s /usr/bin/zsh \
        -G games,lp,optical,power,storage,video,wheel,network,users

    passwd $NOROOT

If you forgot a group, you can later on add it as follows:

    groupadd kalu
    gpasswd -a thomas kalu


##### Install yaourt

It can be helpful to have an AUR helper such as `yaourt` (but ultimately, you
can of course do everything manually).

Install dependencies:

    pacman -S base-devel wget yajl git sudo

Define a neat `aur_install` helper function to help with the build temporarily:

    aur_install() { (
        cd /tmp
        sudo -u $NOROOT git clone "https://aur.archlinux.org/$1" &&
        cd $1 &&
        sudo -u $NOROOT makepkg &&
        pacman -U $1-*.pkg.tar.xz
    ) }

And use it to install yaourt and package-query (dependency) from the AUR:

    aur_install package-query
    aur_install yaourt


##### Console font

The most reasonable console fonts I have found so far are in an AUR package:

    sudo -u $NOROOT yaourt -S terminus-font-ll2-td1

It has a tilde (~) symbol that is properly vertically centered, and
distinguishable characters `il1I` (eye, el, one, capital eye).

Set it up as the default font used on boot:

    cat <<VCONSOLE >>/etc/vconsole.conf
    FONT=ter-216n
    FONT_MAP=8859-2_to_uni
    VCONSOLE

You may want to generate a preliminary initramfs to verify that everything
worked:

    mkinitcpio -p linux


##### Set hardware clock to UTC

Enable NTP:

    pacman -S ntp
    systemctl enable ntpd

    hwclock --systohc --utc
    ntpd -gq
    hwclock -w


##### Create initial ramdisk

Edit `/etc/mkinitcpio.conf` and:

Add the following `MODULES` (make consolefont working):
- `i915`: [early KMS](https://wiki.archlinux.org/index.php/Kernel_mode_setting#Early_KMS_start) for intel chips
- `dm_mod`: for `/boot` on LVM
- `ext4`: for the ext4 `/boot`

Ensure that the following `HOOKS` are defined in this order (among others):

- `base`, `udev`
- `consolefont`    (display custom terminal font early)
- `keymap`         (change to preferred keymap before password prompt)
- …`block`…        (before lvm2!)
- `lvm2`           (need lvm before encrypt: LUKS on LVM)
- `encrypt`        (encrypt hook)
- `filesystems`    (encrypt should be loaded before this)
- `keyboard`
- `shutdown`       (for usr)
- `fsck`           (for usr)
- `usr`            (needed since /usr is on separate partition)

Example:

    HOOKS="base udev autodetect consolefont keymap modconf block lvm2 encrypt filesystems keyboard shutdown fsck usr"

Create the initramfs:

    mkinitcpio -p linux

* [mkinitcpio](https://wiki.archlinux.org/index.php/Mkinitcpio)
* [/usr as a separate partition](https://wiki.archlinux.org/index.php/Mkinitcpio#.2Fusr_as_a_separate_partition)


##### Install grub

Install boot loader:

    pacman -S grub
    pacman -S os-prober

Installing `os-prober` lets you find other OS (dual boot windows). On the other
hand it may cause ugly error messages during `grub-mkconfig` or `grub-install`,
particularly within the chroot environment. You may want to repeat these steps
after rebooting.

**IMPORTANT:** Add the following parameter in `/etc/default/grub`:

    GRUB_CMDLINE_LINUX="cryptdevice=/dev/lunix/root:crypt-root root=/dev/mapper/crypt-root"

Now generate the boot menu:

    grub-mkconfig -o /boot/grub/grub.cfg

And install grub to the hard drive:

* If booting in EFI mode:

      pacman -S efibootmgr
      grub-install --target=x86_64-efi --efi-directory=/boot

* For GPT-BIOS boot (you may also need `--target=i386-pc`?):

      grub-install /dev/sda

##### 3. Profit

Now install further packages as required and reboot into the new system!

Before you boot into your new system without remote access, I recommend
installing:

    # useful stuf like `ifconfig`:
    pacman -S net-tools

    pacman -S networkmanager
    systemctl enable NetworkManager

    pacman -S openssh
    systemctl enable sshd

And on your client machine:

    ssh-copy-id root@192.168.0.XX
