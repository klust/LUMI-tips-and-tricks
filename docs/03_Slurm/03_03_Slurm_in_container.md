# How to get Slurm into a container?


According to Mihkel:

-   Basic use: 
  
    ```
    SINGULARITY_BIND="$SINGULARITY_BIND,/usr/lib64/slurm/,/etc/slurm,/etc/passwd,/usr/lib64/libmunge.so.2,/run/munge,/var/lib/misc,/etc/nsswitch.conf"
    ```

-   More advanced, including all the executables:

    ```
    export SINGULARITY_BIND="$SINGULARITY_BIND,/usr/bin/sacct,/usr/bin/sacctmgr,/usr/bin/salloc,/usr/bin/sattach,/usr/bin/sbatch,/usr/bin/sbcast,/usr/bin/scancel,/usr/bin/scontrol,/usr/bin/scrontab,/usr/bin/sdiag,/usr/bin/sinfo,/usr/bin/sprio,/usr/bin/squeue,/usr/bin/sreport,/usr/bin/srun,/usr/bin/sshare,/usr/bin/sstat,/usr/bin/strigger,/usr/bin/sview,/usr/bin/sgather,/usr/lib64/slurm/,/etc/slurm,/etc/passwd,/usr/lib64/libmunge.so.2,/run/munge,/var/lib/misc,/etc/nsswitch.conf"
    ```


