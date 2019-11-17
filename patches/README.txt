# System File Patches

## rc.earlyboot

Patches `/etc/rc` so that it runs `/etc/fw/bin/rc.earlyboot` before starting
the network. `ntpd` now requires an https constraint in order to set the
time when booting. `ntpd` gets started long before `ifstated`, which in our
setup is responsible for configuring interfaces, routing, and dns resolution.
`rc.earlyboot` configures these items before `/etc/rc` starts the network.

```
# cd /etc
# patch --backup < /etc/fw/patches/rc.earlyboot.diff
```
