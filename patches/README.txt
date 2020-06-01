# System File Patches

diffs all generated with `diff -cu f1 f2`

## rc.earlyboot.diff

Patches `/etc/rc` so that it runs `/etc/fw/bin/rc.earlyboot` before starting
the network. `ntpd` now requires an https constraint in order to set the
time when booting. `ntpd` gets started long before `ifstated`, which in our
setup is responsible for configuring interfaces, routing, and dns resolution.
`rc.earlyboot` configures these items before `/etc/rc` starts the network.

```
# cd /etc
# patch --forward --backup < /etc/fw/patches/rc.earlyboot.diff
```


## daily.diff

Patch `/etc/daily` so it uses https://github.com/bcicen/slackcat instead of mail
to deliver output. This patch requires a `~/.slackcat` file with a proper token
in root's home directory.

```
# cd /etc
# patch --forward --backup < /etc/fw/patches/daily.slackcat.diff
```


## weekly.diff

Patch `/etc/weekly` so it uses https://github.com/bcicen/slackcat instead of
mail to deliver output. This patch requires a `~/.slackcat` file with a proper
token in root's home directory.

```
# cd /etc
# patch --forward --backup < /etc/fw/patches/weekly.slackcat.diff
```
