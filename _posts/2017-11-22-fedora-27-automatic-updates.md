---
layout: post
title: Fedora 27 Automatic Updates
categories:
---

# {{ page.title }}

I thought I'd write a quick short port on setting up automatic updates on Fedora 27.

```
$ dnf install dnf-automatic
```

I am only going to do security updates and have them applied. What you do is up to you. :)

```
# diff /etc/dnf/automatic.conf /etc/dnf/automatic.conf.orig
5c5
< upgrade_type = security
---
> upgrade_type = default
18c18
< apply_updates = yes
---
> apply_updates = no
```

What timers do we have right now? (Ah, systemd, let's replace cron. Sigh.)

```
$ systemctl list-timers *dnf-*
NEXT                         LEFT       LAST                         PASSED       UNIT                ACTIVATES
Wed 2017-11-22 08:36:26 EST  58min left Wed 2017-11-22 07:36:24 EST  1min 47s ago dnf-makecache.timer dnf-makecache.service

1 timers listed.
Pass --all to see loaded but inactive timers, too.
```

Hmm there are several dnf timers.

```
root# ls -1 /usr/lib/systemd/system/dnf*.timer
/usr/lib/systemd/system/dnf-automatic-download.timer
/usr/lib/systemd/system/dnf-automatic-install.timer
/usr/lib/systemd/system/dnf-automatic-notifyonly.timer
/usr/lib/systemd/system/dnf-automatic.timer
/usr/lib/systemd/system/dnf-makecache.timer
```

What does automatic.timer do? Runs the dnf-automatic service.

```
root# cat /usr/lib/systemd/system/dnf-automatic.service
[Unit]
Description=dnf automatic
# See comment in dnf-makecache.service
ConditionPathExists=!/run/ostree-booted

[Service]
Type=oneshot
Nice=19
IOSchedulingClass=2
IOSchedulingPriority=7
Environment="ABRT_IGNORE_PYTHON=1"
ExecStart=/usr/bin/dnf-automatic /etc/dnf/automatic.conf --timer
```

Ok, let's start the timer.

```
root# systemctl enable dnf-automatic.timer && systemctl start dnf-automatic.timer
Created symlink /etc/systemd/system/basic.target.wants/dnf-automatic.timer → /usr/lib/systemd/system/dnf-automatic.timer.
```

Status:

```
root# systemctl status dnf-automatic.timer
● dnf-automatic.timer - dnf-automatic timer
   Loaded: loaded (/usr/lib/systemd/system/dnf-automatic.timer; enabled; vendor preset: disabled)
   Active: active (waiting) since Wed 2017-11-22 07:41:30 EST; 6min ago
  Trigger: Thu 2017-11-23 07:43:53 EST; 23h left

Nov 22 07:41:30 comput0r systemd[1]: Started dnf-automatic timer.

```

Let's try running.

```
root# dnf-automatic
Last metadata expiration check: 0:13:23 ago on Wed 22 Nov 2017 07:36:25 AM EST.
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
The following updates have been applied on 'comput0r':
============================================================================================================================================================================
 Package                                    Arch                                 Version                                        Repository                             Size
============================================================================================================================================================================
Upgrading:
 git                                        x86_64                               2.14.3-2.fc27                                  updates                               1.1 M
 git-core                                   x86_64                               2.14.3-2.fc27                                  updates                               4.1 M
 git-core-doc                               x86_64                               2.14.3-2.fc27                                  updates                               2.3 M
 openssl                                    x86_64                               1:1.1.0g-1.fc27                                updates                               564 k
 openssl-libs                               x86_64                               1:1.1.0g-1.fc27                                updates                               1.3 M
 perl-Git                                   noarch                               2.14.3-2.fc27                                  updates                                68 k
 xen-libs                                   x86_64                               4.9.0-14.fc27                                  updates                               674 k
 xen-licenses                               x86_64                               4.9.0-14.fc27                                  updates                               117 k

Transaction Summary
============================================================================================================================================================================
Upgrade  8 Packages

```

Logs of installed packages are kept.

```
root# tail /var/log/dnf.rpm.log
2017-11-22T12:49:56Z INFO Upgraded: xen-libs-4.9.0-14.fc27.x86_64
2017-11-22T12:49:56Z INFO Upgraded: openssl-1:1.1.0g-1.fc27.x86_64
2017-11-22T12:49:56Z INFO Cleanup: openssl-1:1.1.0f-9.fc27.x86_64
2017-11-22T12:49:56Z INFO Cleanup: git-2.14.3-1.fc27.x86_64
2017-11-22T12:49:56Z INFO Cleanup: git-core-doc-2.14.3-1.fc27.x86_64
2017-11-22T12:49:56Z INFO Cleanup: git-core-2.14.3-1.fc27.x86_64
2017-11-22T12:49:56Z INFO Cleanup: xen-libs-4.9.0-13.fc27.x86_64
2017-11-22T12:49:56Z INFO Cleanup: xen-licenses-4.9.0-13.fc27.x86_64
2017-11-22T12:49:57Z INFO Cleanup: perl-Git-2.14.3-1.fc27.noarch
2017-11-22T12:49:57Z INFO Cleanup: openssl-libs-1:1.1.0f-9.fc27.x86_64
```

So, there you go. Will this computer be more secure now? I hope so. :)
