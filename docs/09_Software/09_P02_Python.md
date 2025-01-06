# Python

Cray Python was built with the GNU compilers.

## mpi4py

-   Error messages about `PMPI_Mprobe`: Early versions of the Cassini provider don't support the
    MPI `MProbe` functions. The workaround is to set

    ``` bash
    export MPI4PY_RC_RECV_MPROBE='False'
    ```

-   mpi4py complains with
  
    ```
    Attempting to use an MPI routine before initializing MPICH
    ```

    According to the course materials it could be related to mpi4py being built with GCC,
    and the workaround is to use

    ```
    LD_PRELOAD=/opt/cray/pe/lib64/libmpi_gnu_91.so
    ```




