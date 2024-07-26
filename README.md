# Redundant OpenBSD Firewall Configuration

## Syncing

We assume that any firewall configuration changes will be made in the files
on fw1 and then replicated to fw2. The `bin/sync-fw-config` script copies
all config files from fw1 to fw2 and restarts all services to ensure the
new files are being used. Some services require changes to the config
files to account for the fact that fw1 and fw2 have different hostnames
and IP addresses.

It also syncs /etc/fw, where the git repo containing all these files
is cloned. This presents a chicken and egg problem because if you don't
have the repo you can't sync the files. Initially you will have to clone
this repo to both firewalls so that you can run the sync on fw2.

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
likely to be the master. This can take a few seconds to take effect, be patient.
When you want the secondary to advertise more slowly you can put it back by:

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
- gwif-master - run by ifstated to configure the internet gateway interface when we are master
- gwif-backup - run by ifstated to configure the internet gateway interface when we are backup
- ns-master - run by ifstated to configure /etc/resolv.conf when we are master
- ns-backup - run by ifstated to configure /etc/resolv.conf when we are master

### fw/etc

Example or actual configuration files. These need to be copied to /etc and modified
appropriately.

- etc/fw.conf - sample fw.conf used by many of these scripts
- daily.local - daily file which updates duckdns


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
7. chown root /etc/duckdns.conf
8. chmod 600 /etc/duckdns.conf
9. run /etc/fw/bin/duckdns.sh from /etc/daily.local with something like:
    ```
    next_part "Updating Duck DNS:"
    /etc/fw/bin/duckdns
    ```

### Redundant time servers

We use `ifstated` to bring up and down redundant network interfaces. `ifstated` starts
after `ntpd`. If your CMOS battery is not working, then `ntpd` can't set the clock
because there isn't a working network configuration, like there would be if you were
not using `ifstated`. The culprit turns out to be the default gateway. When we boot,
we need to set the default gateway to the other firewall. After `ifstated` is running
it will change it to be wherever it should be.

To solve this add the following to the bottom of the `hostname.igc1` file (or whatever
the file is for the internal network interface):
```
!(. /etc/fw.conf && route add default $GWIP)
```

If the other firewall is up, then we'll have internet connectivity during the boot
process.

I have USB gps devices connected to both firewalls. When connected it registers as
`/dev/cuaU0`. You can view the raw data with `cu -l /dev/cuaU0` at 9600 baud.
To make it work as a time and location sensor, you have to do `ldattach nmea /dev/cuaU0`.
To have this always happen when the machine boots, add this line to `/etc/ttys`:
```
cuaU0   "/sbin/ldattach nmea"           unknown on  softcar
```

You can see if the GPS sensor is working with `sysctl hw.sensors`.

The ublox GPS dongle that I have has a tiny green LED on it. That LED blinks green when
it's synced to satellites. It will blink once when you first plug it in, or when the
machine boots. If it hasn't been connected to satellites for a while, it may have to
redownload a bunch of data from the satellites, which can take up to 30 minutes with
no LED status blinking before it starts to sync.

The orientation of the USB dongle is critical. The side with the blinking light needs
to face up towards the sky, because that's where the antenna is. If it's upside down
it probably won't get a satellite lock.
