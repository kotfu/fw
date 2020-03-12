# fw
Redundant OpenBSD Firewall Configuration


## Administration Commands

### Switching Primary Firewall

We rely on `carp` to determine which firewall is currently the master. There
may be occasion when you wish to manually switch which firewall is the
master. You can't force anything to be the master, but you can signal that
you don't want to be the master so another member of the carp group has a
higher chance of being promoted.

`/etc/rc` uses the carp demotion technique to try and prevent becoming the
master while booting. To demote yourself (i.e. to make yourself not be the
master):

```
# ifconfig -g carp carpdemote 128
```

When you want to undemote yourself:

```
# ifconfig -g carp -carpdemote 128
```

See ifconfig(8) for more info.

This method is nice, but it doesn't survive a reboot: carpdemote is always back
to normal when the system boots. If you are going to reboot the primary firewall
and don't want it to be the master when it comes back up, use the following on
the secondary:

```
# ifconfig carp0 advskew 32
# ifconfig carp1 advskew 32
```

These commands allow the secondary to advertise faster, and therefore more
likely to be the master. When you want the secondary to advertise more slowly
you can put it back by:

```
# ifconfig carp0 advskew 128
# ifconfig carp1 advskew 128
```

For reference: here's how I typically configure baseline advskew values for each
firewall:

- fw1 64 (typically master)
- fw2 128 (typically backup)


## What's included

### fw/patches

To get the desired behavior, I had to patch some of the scripts that are
part of the OpenBSD distribution. This directory contains several diffs
which modify stock files.


### fw/bin

There are some handy scripts here which add additional functionality

- duckdns - updates duckdns, a dynamic dns service
- rc.earlyboot - patch /etc/rc to run this before the network is started


### fw/etc

Example configuration files. These need to be copied to /etc and modified
appropriately.

- duckdns.conf - configuration file for fw/bin/duckdns


## Prerequisites and Requirements

- OpenBSD
- curl (get via packages "# pkg_add curl")


## By Function

### Dynamic DNS via duckdns.org

1. Create a file /etc/duckdns.conf
2. log in at www.duckdns.org
3. get the token from there
4. set up a domain (i.e. kotfu.duckdns.org)
5. copy fw/etc/duckdns.conf to /etc
6. edit DOMAIN, TOKEN, and INTERFACE variables in /etc/duckdns.conf
7. # chown root /etc/duckdns.conf
8. # chmod 600 /etc/duckdns.conf
9. run /etc/fw/bin/duckdns.sh from /etc/daily.local
