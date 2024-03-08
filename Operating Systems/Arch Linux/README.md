# Installing Arch Linux on the JTS Securebook 5

The Justice Tech Solutions Securebook 5 (known colloquially as the
"prisonbook") is a laptop designed for use in correctional facilities
and educational institutes.  They have a Celeron processor, 4GB of
memory, 54WH batteries, and some of them even have wifi.  And the case
is made of clear plastic.

They're great.. except they're locked by default with a bios password
and have an allow-list preventing the use of arbitrary hard drives.
This makes running anything on them hard since the recycling company
that's been selling them has shredded the hard drive before selling it.

```
Hard disk have been changed!
System will not boot continue.
Please press ESC for Power OFF
```

Not to worry!  It's possible to work around all that with a laptop SATA
hard drive, a separate linux computer that can write to the hard drive, and
a victim Prisonbook.

The magic to all of this is the fact that the Securebook 5 WILL boot from
a hard drive if you've written an ISO to a partition.  The EFI seems to
think that the partition is a bootable install utiltity and bypasses the
restrictions, booting using that partition.  We can use this to boot an
arbitrary OS.

Let's get Arch Linux up and running.

## Creating the Prisonbook bootloader ISO

While a standard ISO is sufficient for booting, it'd be great to have
an ISO that automatically boots over to grub on the disk.  This simulates
a standard MBR to boot up!

First, we need to prepare the folders & grub configuration.  Create the
grub folder with `mkdir -p iso/boot/grub`.

Include a grub configuration at `iso/boot/grub/grub.cfg`:

```
set timeout_style=hidden
set timeout=0

insmod part_msdos
insmod chain

menuentry "Boot GRUB from Disk" {
  set root=(hd0,2)
  chainloader /EFI/GRUB/GRUBX64.EFI
}
```

Then create the bootloader iso with `grub-mkrescue -o bootloader.iso ./iso/`
and save it for later.

## Partition the disks

The TLDR is that you need primary partition 1 to be for the ISO bootloader
and primary partition 2 to be the EFI boot partition.

The disk should be set up with at least three partitions.

* The first partition is the ISO bootloader, with about 100MB of space.
* The second partition is the EFI boot partition, with about 800M of space.
* At least one partition for the linux installation.

If you want other partitions, y'know, be your best self.

> [!WARNING]
> Do not mark any of the partitions as bootable!
> 
> As soon as it's marked bootable the securebook 5 bios will deny access to the drive.

To help, here's how I set up my partitions:
```
Disk /dev/sda: 465.76 GiB, 500107862016 bytes, 976773168 sectors
Disk model: 0ASG            
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 33553920 bytes
Disklabel type: dos
Disk identifier: 0xbc27c835

Device     Boot    Start       End   Sectors   Size Id Type
/dev/sda1           2048    206847    204800   100M ef EFI (FAT-12/16/32)
/dev/sda2         206848   1845247   1638400   800M ef EFI (FAT-12/16/32)
/dev/sda3        1845248  17469439  15624192   7.5G 82 Linux swap / Solaris
/dev/sda4       17469440 976773167 959303728 457.4G 83 Linux
```

## Install Arch Linux from another Computer

### Set up the Bootstrap Environment

Basically: [Follow the guide on the Arch wiki.](https://wiki.archlinux.org/title/Install_Arch_Linux_from_existing_Linux)

Go into the `tmp` directory and download the Arch Linux
bootstrap `tar.gz`, extracting it to the directory with
the `--numeric-owner` option to preserve the user / group IDs.

```bash
cd /tmp
wget https://mirrors.edge.kernel.org/archlinux/iso/2024.03.01/archlinux-bootstrap-x86_64.tar.gz
tar xzf archlinux-bootstrap-x86_64.tar.gz --numeric-owner
```

Then you'll need to set up the bootstrap root & chroot into it.

```bash
mount --bind /tmp/root.x86_64 /tmp/root.x86_64
cd /tmp/root.x86_64
cp /etc/resolv.conf etc
mount -t proc /proc proc
mount --make-rslave --rbind /sys sys
mount --make-rslave --rbind /dev dev
mount --make-rslave --rbind /run run    # (assuming /run exists on the system)
chroot /tmp/root.x86_64 /bin/bash
```

Don't forget to initialize and populate pacman's keys via
`pacman-key --init` and `pacman-key --populate`.

### Install Arch Linux

Basically: [Follow standard install guide,](https://wiki.archlinux.org/title/installation_guide) until you need a bootloader.

There's a few slightly different steps, detailed below.

#### Formatting 

Initialize the EFI boot partition via `mkfs.vfat /dev/sdX2`.

Initialize the swap partition via `mkswap /dev/sdX3`.

Initialize the root partition via `mkfs.ext4 /dev/sdX4`.

Then you'll need to mount the root and boot partitions.

```
mount /dev/sda4 /mnt
mount --mkdir /dev/sda2 /mnt/boot
```

#### Installing GRUB and the bootloader ISO

This approach to a bootloader for the Prisonbook consists of two stages. 
The first stage is the ISO which can be caught by EFI, the second is a 
standard bootloader.

The first stage of the bootloader is installed by copying the bootloader
ISO into the first partition we've set up via the command
`dd if=prisonbook-boot.iso of=/dev/sdX1` where `X` is the expected drive.

Once that's been copied over, install `grub` and `efibootmgr` with
`pacman -S grub efibootmgr`

When installing grub we don't want to update the bootsector, nor do we
want to modify EFI vars.  If any of these mark the drive as bootable, it
means we'll fail with the HDD changes error message.

```
grub-install --target=x86_64-efi \
    --efi-directory=/boot \
    --bootloader-id=GRUB \
    --no-bootsector \
    --no-nvram
```
