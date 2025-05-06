# Some examples of stupid things to do on LUMI

-   Poll the scheduler at high frequency.

    Even `squeue` is an expensive command for Slurm, and when it is receiving too many
    such requests, scheduling jobs may actually be halted. Slurm runs on a single server,
    is multi-threaded but with large critical sections, so the amount of parallelism in 
    scheduling is limited.

    Example of something that should absolutely not be done:

    ```
    watch squeue --me
    ```

    which would poll the scheduler every 2 seconds. Don't poll more frequently than
    once a minute or even less. 

-   Use a different `scp` command for each file to transfer and use this to transfer
    50000 files, bombarding the login nodes with ssh login requests. It will make you
    look very suspicious and is not an efficient use of resources.

