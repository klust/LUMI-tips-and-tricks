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

    Error messgage:

    ```
    MPIDI_OFI_handle_cq_error(1062): OFI poll failed (ofi_events.c:1064:MPIDI_OFI_handle_cq_error:Input/output error - PTLTE_NOT_FOUND)
    ```

    HPE Cray recommendation:

    ```
    export FI_CXI_MSG_OFFLOAD=0
    export FI_CXI_REQ_BUF_COUNT=200
    ```

    These are undocumented environment variables.


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



