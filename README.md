# fw
Redundant OpenBSD Firewall Configuration


## What's included

### fw/patches

To get the desired behavior, I had to patch some of the scripts that are part of
the OpenBSD distribution.


### fw/bin

There are some handy scripts here which add additional functionality

- duckdns - updates duckdns, a dynamic dns service


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
1. log in at www.duckdns.org
2. get the token from there
3. set up a domain (i.e. kotfu.duckdns.org)
4. copy fw/etc/duckdns.conf to /etc
4. set DOMAIN, TOKEN, and INTERFACE variables in /etc/duckdns.conf
5. run /etc/fw/bin/duckdns.sh from /etc/daily.local
