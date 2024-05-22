# Wrapper-induced issues

## Tests for Score-P failing on the GPU nodes

Reproducing is trivial:

```
#include <stdlib.h>
#include <omp.h>
#include <inttypes.h>

int main( void )
{
    return 0;
}
```

and compile with `LUMI/23.09 partition/G cpeCray` or `cpeAMD`.

Answer from Alfio:

-   The problem when using the wrapper `cc` is that the module craype-accel-amd-gfx90a promotes OpenMP to be offloaded to the target GPU. 
    Then, we have to mix with the ROCm files and this can be done after the preprocessing of the OpenMP pragmas.

    You can use the flag `-craype-verbose` to see what command is executed, e.g.

    ```
    $  cc -fopenmp -craype-verbose --preprocess test_with_includes.c

    clang -march=znver2 -fopenmp-targets=amdgcn-amd-amdhsa -Xopenmp-target=amdgcn-amd-amdhsa -march=gfx90a -dynamic -D__CRAY_X86_ROME -D__CRAY_AMD_GFX90A -D__CRAYXT_COMPUTE_LINUX_TARGET --gcc-toolchain=/opt/cray/pe/gcc/10.3.0/snos -isystem /opt/cray/pe/cce/16.0.1/cce-clang/x86_64/lib/clang/16/include -isystem /opt/cray/pe/cce/16.0.1/cce/x86_64/include/craylibs -Wl,-rpath=/opt/cray/pe/cce/16.0.1/cce/x86_64/lib -fopenmp --preprocess test_with_includes.c -I/opt/cray/pe/libsci/23.09.1.1/CRAY/12.0/x86_64/include -I/opt/cray/pe/mpich/8.1.27/ofi/cray/14.0/include -I/opt/cray/pe/dsmml/0.2.2/dsmml//include -I/opt/cray/xpmem/2.5.2-2.4_3.50__gd0f7936.shasta/include -L/opt/cray/pe/libsci/23.09.1.1/CRAY/12.0/x86_64/lib -L/opt/cray/pe/mpich/8.1.27/ofi/cray/14.0/lib -L/opt/cray/pe/mpich/8.1.27/gtl/lib -L/opt/cray/pe/dsmml/0.2.2/dsmml//lib -L/opt/cray/pe/cce/16.0.1/cce/x86_64/lib/pkgconfig/../ -L/opt/cray/xpmem/2.5.2-2.4_3.50__gd0f7936.shasta/lib64 -Wl,--as-needed,-lsci_cray_mpi_mp,--no-as-needed -Wl,--as-needed,-lsci_cray_mp,--no-as-needed -ldl -Wl,--as-needed,-lmpi_cray,--no-as-needed -lmpi_gtl_hsa -Wl,--as-needed,-ldsmml,--no-as-needed -lxpmem -Wl,--as-needed,-lstdc++,--no-as-needed -Wl,--as-needed,-lpgas-shmem,--no-as-needed -lquadmath -lmodules -lfi -lcraymath -lf -lu -lcsup -Wl,--as-needed,-lpthread,-latomic,--no-as-needed -Wl,--as-needed,-lm,--no-as-needed -Wl,--disable-new-dtags 
    ```

    As you said, it works if you don't directly use the wrappers (or you remove OpenMP, or if you don't load the GPU target module). 

    My suspicious is that the problem is due to LLVM (<=16) compatibility.

-   For the CCE,  we can apply the following trick:

    ```
    cc -fno-cray -fopenmp --preprocess test_with_includes.c > test_with_includes.prep.offload.c
    ```

    Here, the `-fno-cray` gives you the vanilla LLVM compiler without specific Cray optimizations. Note that we need this trick only for the CCE 16.0.1 installed on LUMI, I tried with CCE 17+ and we don't need this flag anymore.

    The next step is to compile the preprocessor output:

    ```
    $ cc -fno-cray -fopenmp --preprocess test_with_includes.c > test_with_includes.prep.offload.c && cc -nostdinc -fopenmp test_with_includes.prep.offload.c
    ```

    In this case I'm using the flag `-nostdinc` to avoid the compiler to reintroduce implicit headers that will conflict with the existing headers of the preprocessor output.

-   Concerning the AMD compiler, the version installed on LUMI relies on LLVM 14 (Rocm 5.2) and it gives you the same problem 
    (even if you specify `-nostdinc`). Also in this case, I've tested it with a new ROCM version (6.1) and it works. 
    I don't have a workaround for the version installed on LUMI though, the suggestion can be to directly use amdclang as 
    you already did (you can use the flag ` --cray-print-opts ` for the wrappers to get a list of the options and then 
    exclude the OpenMP offload ones, see `man cc`).

