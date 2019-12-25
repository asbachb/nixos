# How to install nixos on VirtualBox

Used software:
* NixOS 19.09
* VirtualBox 6.0

## Download minimal iso
see https://nixos.org/nixos/download.html for latest iso

## Create a virtual machine 
Create a new virtual machine in VirtualBox with
* Type: Linux
* Version: Other Linux (64-bit)

In order to use automatic screen resizing you need to adjust these settings:
* Display / Screen / Graphics Controller: VBoxVGA

Override video drivers in `/etc/nixos/configuration.nix` (xserver won't start with VBoxVGA and without this option)

```
services.xserver.videoDrivers = mkOverride 40 [ "virtualbox" "vmware" "cirrus" "vesa" "modesetting"];
```

## Installation
Mount latest iso for minimal installation and boot virtual machine.

Change keyboard if applicable.
```
sudo loadkeys de
```

Partioning depends 1) if you want to use mbr or gpt is partition table and b) if you need a swap partition. Most simple would be MBR + No Swap Partition.
In this example a 2GB swap parition will be configured.
```
sudo su

# partition table
parted /dev/sda mktable msdos

# swap partition
parted /dev/sda -- mkpart primary linux-swap 0% 2GiB
mkswap -L swap /dev/sda1
swapon /dev/sda1

# root partition
parted /dev/sda -- mkpart primary ext4 2GiB 100%
parted /dev/sda set 2 boot on
mkfx.ext4 -L nixos /dev/sda2
mount /dev/sda2 /mnt
```

Generate initial nixos configuration.
```
nixos-generate-configuration --root /mnt
```

Adjust configuration to your needs.
```
nano /mnt/etc/nixos/configuration.nix
```

Install nixos.
```
nixos-install
```
