# LUMI peak performance

*Thanks to Orian for some of the data.*

## Top-500 result

Preliminaries

-   The GPU clock for the computations is set to 1600 MHz, the same as the memory clock,
    instead of the theoretical maximum of 1700 MHz. It is also what is in practice used
    on LUMI?
-   The peak flop computation is based on the vector units rather than the matrix units.

Computations:

-   RPeak of 1 GPU = 220 CUs \* 128 FMA Flop/CU/clock \* 1600 MHz = 45.056 TFlops
-   RPeak of 1 CPU of LUMI-G = 64 core \* 16 FMA Flop/core/clock \* 2.0 GHz = 2.048 TFlops
-   RPeak of 1 LUMI-G node = 4 \* 45.056 TFlops + 2.048 TFlops = 182.272 TFlops

The result in the November 2023 and June 2024 Top500 listing was obtained on 2916 nodes
of LUMI-G.

-   Computation of RPeak: 2916 \* ( 4 \* 220 CUs \* 128 FMA Flop/CU/clock \* 1.600 GHz 
    + 64 core \* 16 FMA Flop/core/clock \* 2.0 GHz) = 531.51 PFlops
-   For the computation of the number of cores, each CU was considers a core, and each
    CPU core, so the total number is
    2916 \* (4 \* 220 + 64) = 2752704


## Single node HPL

-   1 x MI250x (2 GCDs) = 40.45 TFlops (21.07 on 1 GCD)
-   4 x MI250x (8 GCDs) = 161.97 TFlops

(which is 23% better than the per-node performance in the Top500 result)
