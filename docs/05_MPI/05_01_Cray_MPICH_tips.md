# Cray MPICH tips


## Remarks from the November 2023 hackathon

-   Using hugepages can improve intra-node performance (or performance overall?)

-   Intra-node performance can improve by setting

    ```
    export MPICH_SMP_SINGLE_COPY_MODE=NONE
    ```
 
    at least for large message sizes.

    (Partly) documented in [the `intro_mpi` man page](https://cpe.ext.hpe.com/docs/mpt/mpich/intro_mpi.html#smp-environment-variables).


## OFI error messages and potential workarounds

-   `PLTE_NOT_FOUND`

    Error message:

    ```
    MPIDI_OFI_handle_cq_error(1062): OFI poll failed (ofi_events.c:1064:MPIDI_OFI_handle_cq_error:Input/output error - PTLTE_NOT_FOUND)
    ```

    HPE Cray recommendation:

    ```
    export FI_CXI_MSG_OFFLOAD=0
    export FI_CXI_REQ_BUF_COUNT=200
    ```

    These are undocumented environment variables.

-   `UNDELIVERABLE`
   
    Error message:

    ```
    MPIDI_OFI_handle_cq_error(1067): OFI poll failed (ofi_events.c:1069:MPIDI_OFI_handle_cq_error:Input/output error - UNDELIVERABLE)
    ```

    One possible cause is a broken switch so that messages can no longer be delivered.

    -   Symptoms on the GPU nodes: A pattern of a number of subsequent either even-numbered or odd-numbered nodes.


## Other MPI error messages

-   `xpmem_attache failed on`

    Error message:

    ```
    MPIDI_CRAY_XPMEM_do_attach(482)...........: xpmem_attach failed on rank 5 (src_rank 20, vaddr 0x3abf5b20, len 2400)
    ```

    It is a memory registration cache bug. The default MR monitor is known to be buggy. The memory registration cache keeps track of what memory regions have been pinned and mapped to the network interface. A bug in the registration cache monitor can have some unexpected side effects, like memory regions deallocated by the code might not be correctly unregistered, which can result in future buffers that share the same virtual addresses being mapped incorrectly or thrown back. Steve Abbott (HPE Frontier support) thinks memhooks is better but hasn't gotten much testing yet, and long term (towards the end of 2023) HPE intends to implement a solution that makes it irrelevant and is better all around.

    Solutions:

    -   Preferred: 

        ```
        export FI_MR_CACHE_MONITOR=memhooks
        ```

        to use the memhooks memory monitor.

        Undocumented (or has this become irrelevant in the current versions)?

    -   Alternative:

        ```
        export FI_MR_CACHE_MAX_COUNT=0
        ```

        which turns off the memory registration cache all together but mibht be pretty punishing on performance (code-dependent).

        Documented in [the `intro_mpi` man page](https://cpe.ext.hpe.com/docs/mpt/mpich/intro_mpi.html).


## Investigating crashes

Based on tips from Juha Jaykka.

-   Use `sacct` to figure out what nodes a job is using and in what time interval it ran:
   
    ```
    sacct --job=$jobid --format=JobID,JobName,State,ExitCode,Start,End,NodeList%1000
    ```

-   Now use `sacctmgr` to list events on those nodes during the time window. You can 
    copy-paste date/times and the nodelist from the previous command:

    ```
    sacctmgr list event Start=<starttime> End=<endtimne> Nodes=<nodelist> format=NodeName,TimeStart,Reason%150
    ```

-   Sometimes you can get more information about a specific node with a fault via `scontrol show node`. 
    So for a faulty node from the previous command, you can run:

    ```
    scontrol show node <nodename or nodelist>
    ```

Note that after many node failures, a node goes automatically into drain mode, so it does not
make sense to manually exclude them in your job scripts.
