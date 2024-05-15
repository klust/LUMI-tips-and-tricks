# Misc remarks for the login nodes

!!! Note "Message Ville on May 15, 2024:"
    We are also experimenting with a cgroup based CPU usage limit which is now in place on uan01. 
    It will limit the number of cores a user can consume on a single ssh session to 16 cores. 
    If this works well, we will put this into use on the other uan's as well. 
    We started also looking into limiting the memory usage and enabling the use of NVME as 
    /tmp and /var/tmp backing storage. We may have a path forward with that using 
    private namespaced /tmp mounts for each user. We could enable this later 
    on the LUST uan node if you'd like to test and see it action.

