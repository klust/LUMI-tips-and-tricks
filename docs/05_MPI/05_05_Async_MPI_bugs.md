# Bugs caused by asynchronous communication


## Issue 1

See the issue raised in ticket 7719.

A user was running an in-house CFD code and got very strange issues when spreading over 2 nodes,
but using 4 GCDs on each node. For some combinations of GCDs, NaNs were frequently produced
(0, 2, 4, 6 worked correctly though which are the ones that are not connected directly to the
NICs). No issues were observed in the cPU-only version of the code.

In the end, it turned out to be a code error and not a bug in Cray MPI. 
There was a missing device synchronisation and sometimes the send buffers were not
completely updated in GPU memory before MPI performed the send.

To quote from the user: This would explain many of the observations:

-   MPI_WaitAll does not raise any error because MPI does actually complete the communications; simply the buffers were not updated. 
-   This obviously cannot occur on CPU.
-   This was a silent bug on their local supercomputer using NVIDIA GPUs. I read that CUDA is more robust with regards to synchronization aspects compared to ROCm. Nevertheless, this is still unsafe and we should fix it on CUDA side as well.
-   This can apparently also depends on MPI stacks, hence probably the reason why it started to appear after the maintenance on LUMI.
-   The results were dependent on the GCDs arrangement, and this would make sense considering that communications can be scheduled differently then.  
