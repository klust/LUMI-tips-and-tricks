# Misc remarks for the login nodes

!!! Note "Message Ville on May 15, 2024:"
    We are also experimenting with a cgroup based CPU usage limit which is now in place on uan01. 
    It will limit the number of cores a user can consume on a single ssh session to 16 cores. 
    If this works well, we will put this into use on the other uan's as well. 
    We started also looking into limiting the memory usage and enabling the use of NVME as 
    /tmp and /var/tmp backing storage. We may have a path forward with that using 
    private namespaced /tmp mounts for each user. We could enable this later 
    on the LUST uan node if you'd like to test and see it action.

    We can also have a separate directory on the nvme for that too. We would just mount the NVME 
    under /local and create /local/tmp with mode 777 and then have the namespaced tmp mounts 
    backed by directory like /local/ns/tmp and /local/ns/var-tmp. This way we would not take away 
    any use case (although the path would change) and still get the benefits. I have this exact 
    setup now on one of the uan's (which is not user or lust accessible) made by hand and I think 
    it looks pretty nice. If you like we can change uan06 to have the same setup tomorrow for 
    you to check out. This is not yet in the configuration management, so the changes will 
    be lost at boot, but we can enable it on the fly for uan06 and new sessions will get 
    the private tmp. If it is good we could start the process to get that change done for uan[01-04] too.

To see the current limits:

```bash
$ cat /etc/systemd/system/user-.slice.d/50-userlimits.conf
[Slice]
CPUQuota=1600%
MemoryMax=96G
```

We have a private tmp:

```bash
$ cat /etc/security/namespace.conf
/tmp     /local/ns/tmp/         level      root
/var/tmp /local/ns/var-tmp/     level      root
```

