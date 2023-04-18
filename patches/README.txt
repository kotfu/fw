# System File Patches

diffs all generated with `diff -cu f1 f2`

## daily.slackcat.diff

Patch `/etc/daily` so it uses https://github.com/bcicen/slackcat instead of mail
to deliver output. This patch requires a `~/.slackcat` file with a proper token
in root's home directory.

```
# cd /etc
# patch --forward --backup < /etc/fw/patches/daily.slackcat.diff
```


## weekly.slackcat.diff

Patch `/etc/weekly` so it uses https://github.com/bcicen/slackcat instead of
mail to deliver output. This patch requires a `~/.slackcat` file with a proper
token in root's home directory.

```
# cd /etc
# patch --forward --backup < /etc/fw/patches/weekly.slackcat.diff
```

## monthly.slackcat.diff

Patch `/etc/monthly` so it uses https://github.com/bcicen/slackcat instead of
mail to deliver output. This patch requires a `~/.slackcat` file with a proper
token in root's home directory.

```
# cd /etc
# patch --forward --backup < /etc/fw/patches/monthly.slackcat.diff
```
