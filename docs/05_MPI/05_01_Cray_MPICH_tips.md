# Cray MPICH tips


## Remarks from the November 2023 hackathon

-   Using hugepages can improve intra-node performance (or performance overall?)

-   Intra-node performance can improve by setting

    ```
    export MPICH_SMP_SINGLE_COPY_MODE=NONE
    ```
 
    at least for large message sizes.
