Goal: Have a *virtualbox instance* with a running *lxd* service which running a *nix os container* which is running a *nginx* server.

### Creating a NixOS base image
```
lxc image import $(nixos-generate -f lxc-metadata) $(nixos-generate -f lxc) --alias nixos
```

### Issues with a running NixOS container
#### nscd.service fails to start
```
root@nixos:~]# systemctl
...
nscd.service                           loaded failed     failed    Name Service Cache Daemon
...
```

#### systemd-udev-trigger.service fails to start
```
root@nixos:~]# systemctl
...
systemd-udev-trigger.service           loaded failed     failed    udev Coldplug all Devices 
...
```

#### systemd-journald-audit.socket fails to start
```
root@nixos:~]# systemctl
...
systemd-journald-audit.socket          loaded failed     failed    Journal Audit Socket
...
```

#### getty@tty1.service keps restarting and spams journalctl
```
root@nixos:~]# journalctl
...
Oct 18 01:46:35 nixos systemd[1]: Started Getty on tty1.
Oct 18 01:46:35 nixos agetty[592]: /dev/tty1: cannot open as standard input: No such file or directory
Oct 18 01:46:45 nixos systemd[1]: getty@tty1.service: Main process exited, code=exited, status=1/FAILURE
Oct 18 01:46:45 nixos systemd[1]: getty@tty1.service: Failed with result 'exit-code'.
Oct 18 01:46:45 nixos systemd[1]: getty@tty1.service: Service has no hold-off time (RestartSec=0), scheduling restart.
Oct 18 01:46:45 nixos systemd[1]: getty@tty1.service: Scheduled restart job, restart counter is at 97.
Oct 18 01:46:45 nixos systemd[1]: Stopped Getty on tty1
```
