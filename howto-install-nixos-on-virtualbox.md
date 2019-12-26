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
Note: In order to use `mkOverride` ensure your `configuration.nix`

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

Example configuration
```
# Edit this configuration file to define what should be installed on
# your system.  Help is available in the configuration.nix(5) man page
# and in the NixOS manual (accessible by running ‘nixos-help’).

{ config, pkgs, lib, ... }:

with lib;

{
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
    ];

  # Use the GRUB 2 boot loader.
  boot.loader.grub.enable = true;
  boot.loader.grub.version = 2;
  # boot.loader.grub.efiSupport = true;
  # boot.loader.grub.efiInstallAsRemovable = true;
  # boot.loader.efi.efiSysMountPoint = "/boot/efi";
  # Define on which hard drive you want to install Grub.
  boot.loader.grub.device = "/dev/sda"; # or "nodev" for efi only

  # networking.hostName = "nixos"; # Define your hostname.
  # networking.wireless.enable = true;  # Enables wireless support via wpa_supplicant.

  # The global useDHCP flag is deprecated, therefore explicitly set to false here.
  # Per-interface useDHCP will be mandatory in the future, so this generated config
  # replicates the default behaviour.
  networking.useDHCP = false;
  networking.interfaces.enp0s3.useDHCP = true;

  # Configure network proxy if necessary
  # networking.proxy.default = "http://user:password@proxy:port/";
  # networking.proxy.noProxy = "127.0.0.1,localhost,internal.domain";

  # Select internationalisation properties.
  i18n = {
    consoleFont = "Lat2-Terminus16";
    consoleKeyMap = "de";
    defaultLocale = "en_GB.UTF-8";
  };

  # Set your time zone.
  time.timeZone = "Europe/Berlin";

  # List packages installed in system profile. To search, run:
  # $ nix search wget
  environment.systemPackages = with pkgs; [
    kate firefox git
  ];
  
  environment.etc."gitconfig".text = ''
    [user]
      name = Benjamin Asbach
      email = asbachb@users.noreply.github.com
  '';

  # Some programs need SUID wrappers, can be configured further or are
  # started in user sessions.
  # programs.mtr.enable = true;
  # programs.gnupg.agent = { enable = true; enableSSHSupport = true; };

  # List services that you want to enable:

  # Enable the OpenSSH daemon.
  # services.openssh.enable = true;

  # Open ports in the firewall.
  # networking.firewall.allowedTCPPorts = [ ... ];
  # networking.firewall.allowedUDPPorts = [ ... ];
  # Or disable the firewall altogether.
  # networking.firewall.enable = false;

  # Enable CUPS to print documents.
  # services.printing.enable = true;

  # Enable sound.
  # sound.enable = true;
  # hardware.pulseaudio.enable = true;

  # Enable the X11 windowing system.
  services.xserver.enable = true;
  services.xserver.layout = "de";
  # services.xserver.xkbOptions = "eurosign:e";

  # Enable touchpad support.
  # services.xserver.libinput.enable = true;

  # Enable the KDE Desktop Environment.
  services.xserver.displayManager.sddm.enable = true;
  services.xserver.desktopManager.plasma5.enable = true;

  services.xserver.videoDrivers = mkOverride 40 ["virtualbox" "vmware" "cirrus" "vesa" "modesetting"];

  # Define a user account. Don't forget to set a password with ‘passwd’.
  users.users.vm = {
    isNormalUser = true;
    extraGroups = [ "wheel" "vboxsf" ]; # Enable ‘sudo’ for the user.
  };

  # This value determines the NixOS release with which your system is to be
  # compatible, in order to avoid breaking some software such as database
  # servers. You should change this only after NixOS release notes say you
  # should.
  system.stateVersion = "19.09"; # Did you read the comment?

}


```

Install nixos.
```
nixos-install
```
