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
