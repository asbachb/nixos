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

Change keyboard if applicable:
```
sudo loadkeys de
```

Create one partition of you assign enough ram to you vm. Create two partitions if you need a swap partition.
```
sudo su

parted /dev/sda mktable gpt
parted /dev/sda mkpart primary linux-swap 1MiB 1GiB
parted /dev/sda mkpart primary ext4 1GiB 100%
parted /dev/sda set 2 boot on
```
