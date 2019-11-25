Goal: Have a *virtualbox instance* with a running *lxd* service which running a *nix os container* which is running a *nginx* server.

### Phase 1: Creating a NixOS base image
```
lxc image import $(nixos-generate -f lxc-metadata) $(nixos-generate -f lxc) --alias nixos
```

### Issues with running NixOS containers
#### nscd.service fails to start
```
root@nixos:~]# systemctl
...
nscd.service                           loaded failed     failed    Name Service Cache Daemon
...
```
```
[root@nixos:~]# journalctl -u nscd.service
...
Oct 18 01:30:29 nixos systemd[1]: Starting Name Service Cache Daemon...
Oct 18 01:30:29 nixos systemd[351]: nscd.service: Failed to set up mount namespacing: /run/systemd/unit-root/run/nscd: Permission denied
Oct 18 01:30:29 nixos systemd[351]: nscd.service: Failed at step NAMESPACE spawning /nix/store/h8jaqdprgf9rv8ldhd2j0nv5i5b32k01-glibc-2.27-bin/sbin/nscd: Permission denied
Oct 18 01:30:29 nixos systemd[1]: nscd.service: Control process exited, code=exited, status=226/NAMESPACE
Oct 18 01:30:29 nixos systemd[1]: nscd.service: Failed with result 'exit-code'.
Oct 18 01:30:29 nixos systemd[1]: Failed to start Name Service Cache Daemon.
Oct 18 01:30:29 nixos systemd[1]: nscd.service: Service RestartSec=100ms expired, scheduling restart.
Oct 18 01:30:29 nixos systemd[1]: nscd.service: Scheduled restart job, restart counter is at 3.
Oct 18 01:30:29 nixos systemd[1]: Stopped Name Service Cache Daemon.
```
##### Solution
It's a little bit unclear how to solve the issue within a unprivileged container. So disable the service for now.
```
services.nscd.enable = false;
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

#### nix-channel could not be updated since proc access error
```
[root@nixos:~]# nix-channel --update 
unpacking channels...
warning: Nix search path entry '/nix/var/nix/profiles/per-user/root/channels' does not exist, ignoring
error: while setting up the build environment: mounting /proc: Operation not permitted
error: program '/nix/store/bnrk3fsrwxfalix265sx4y3r2909dc6m-nix-2.3/bin/nix-env' failed with exit code 1
```
