# Custom paths

The information on this page is partly the result of studying
`/opt/cray/pe/lmod_scripts/default/scripts/lmodHierarchy.lua` and other 
module code.

The HPE Cray PE can load custom paths with system-installed or user-installed 
modules in a way that fits in the hierarchy defined by the PE. The following options
are available:

-   Modules that depend on the CPU target module only.

    In the PE, these are in the `cpu` subdirectory and used for the `cray-fftw`
    modules.

    As these modules require loading only one module to be made available, there is
    no `handshake_` routine involved in `lmodHierarchy.lua`.

-   Modules that depend on the network target module only, so not on the compiler or the
    CPU architecture.

    In the PE, these are in the `net` subdirectory and used for the `cray-openshmemx`
    modules.

    As these modules require loading only one module to be made available, there is
    no `handshake_` routine involved in `lmodHierarchy.lua`.

-   Modules that depend on the compiler only (so for a generic CPU)

    In the PE, these are in the `compiler` subdirectory and used for the `cray-hdf5` 
    modules.

    As these modules require loading only one module to be made available, there is
    no `handshake_` routine involved in `lmodHierarchy.lua`.

-   Modules that depend on the CPU target module and the compiler.

    There are currently no such examples in the PE. however, the code is prepared for 
    future addition of such directories in the PE itself also and the reserved directory
    name is `comcpu`. 

    As these modules require loading of two modules to be made available which in the PE
    can happen in any order as the hierarchy is not build in the typical Lmod way, 
    adding of both the PE and custom paths to `MODULEPATH` is managed by the
    `lmodHierarchy.handshake_comcpu` routine in `lmodHierarchy.lua`.

-   Modules that depend on the network target module and the compiler.

    In the PE, these are in the `comnet` subdirectory and used for the `cray-mpich`,
    `cray-mpich-abi`, 'cray-mpich-ucx` and `cray-mpich-ucx-abi` modules.

    As these modules require loading of two modules to be made available which in the PE
    can happen in any order as the hierarchy is not build in the typical Lmod way, 
    adding of both the PE and custom paths to `MODULEPATH` is managed by the
    `lmodHierarchy.handshake_comnet` routine in `lmodHierarchy.lua`.

-   Modules that depend on the network target module, the compiler and the MPI library.

    In the Cray PE, these are in the `mpi` subdirectory and used for the `cray-hdf5-parallel`
    and `cray-parallel-netcdf` modules.

    The addicion of the `mpi` subdiredctories is done in a way that is different from the
    other ones that depend on multiple modules, likely because the `cray-mpich-*` modules
    that add this level to the hierarchy themselves are at the `comnet` level and changed
    as soon as the network target or compiler module is changed, so that the regular Lmod
    approach of building a hierarchy can be used. The code that adds both the PE `mpi`
    directory and the corresponding custom path to the `MODULEPATH` is largely contained in 
    the `cray-mpich-*` modules but does use the `lmodHierarchy.get_user_custom_path`
    routine from `lmodHierarchy.lua`.

-   Modules that depend on both the network and CPU target modules, on the compiler and 
    on the MPI library.

    There are currently no such examples in the PE. however, the code is prepared for 
    future addition of such directories in the PE itself also and the reserved directory
    name is `cncm`. 

    As these modules require loading of two modules to be made available which in the PE
    can happen in any order as the hierarchy is not build in the typical Lmod way, 
    adding of both the PE and custom paths to `MODULEPATH` is managed by the
    `lmodHierarchy.handshake_cncm` routine in `lmodHierarchy.lua`.

```
module_root
├─ cpu
│  ├─ x86-64
│  │  └─ 1.0
│  ├─ x86-milan
│  │  └─ 1.0
│  ├─ x86-rome
│  │  └─ 1.0
│  └─ x86-trento
│     └─ 1.0
├─ net
│  ├─ ofi
│  │  └─ 1.0
│  └─ ucx
│     └─ 1.0
├─ compiler
│  ├─ aocc
│  │  ├─ 2.2
│  │  └─ 3.0
│  ├─ crayclang
│  │  └─ 10.0
│  └─ gnu
│     └─ 8.0
├─ comcpu
│  ├─ aocc
│  │  ├─ 2.2
│  │  └─ 3.0
│  ├─ crayclang
│  │  └─ 10.0
│  │     ├─ x86-64
│  │     │  └─ 1.0
│  │     ├─ x86-milan
│  │     │  └─ 1.0
│  │     ├─ x86-rome
│  │     │  └─ 1.0
│  │     └─ x86-trento
│  │        └─ 1.0
│  └─ gnu
│     └─ 8.0
├─ comnet
│  ├─ aocc
│  │  ├─ 2.2
│  │  └─ 3.0
│  ├─ crayclang
│  │  └─ 10.0
│  │     ├─ ofi
│  │     │  └─ 1.0
│  │     └─ ucx
│  │        └─ 1.0
│  └─ gnu
│     └─ 8.0
├─ mpi
│  ├─ aocc
│  │  ├─ 2.2
│  │  └─ 3.0
│  ├─ crayclang
│  │  └─ 10.0
│  │     ├─ ofi
│  │     │  └─ 1.0
│  │     │     └─ cray-mpich
│  │     │        └─ 8.0
│  │     └─ ucx
│  │        └─ 1.0
│  └─ gnu
│     └─ 8.0
└─ cncm
   ├─ aocc
   │  ├─ 2.2
   │  └─ 3.0
   ├─ crayclang
   │  └─ 10.0
   │     ├─ ofi
   │     │  └─ 1.0
   │     │     ├─ x86-64
   │     │     │  └─ 1.0
   │     │     │     └─ cray-mpich
   │     │     │        └─ 8.0
   ⋮     ⋮      ⋮   
   │     └─ ucx
   │        └─ 1.0
   └─ gnu
      └─ 8.0
```

Given that the custom paths are not set by a single environment variable pointing to the
roots of the custom installation directories and imposing the same directory structure
as used by the PE, but instead by separate environment variables for every leaf in the picture below,
the number of environment variables can explode.

On LUMI, as of July 2022, we have:

-   3 CPU target modules and the generic one if we want to support that also,
    so 4 CPU target modules.
-   1 network library (though one could argue that UCX can still be used on the 
    login, large memory and visualisation GPU nodes but these are not really meant
    for heavy MPI work unless they are integrating with jobs on the regular compute 
    nodes)
-   4 compiler ABIs, with one to be dropped soon (aocc/2.2, aocc/3.0, crayclang/10.0 and gnu/8.0)
-   1 MPI ABI version (cray-mpich/8.0)

This may result in:

-   4 `LMOD_CUSTOM_CPU_*` environment variables
-   1 `LMOD_CUSTOM_NET_*` environment variable (or 2 with UCX)
-   4 `LMOD_CUSTOM_COMPILER_*` environment variables
-   16 `LMOD_CUSTOM_COMCPU_*` environment variables
-   4 `LMOD_CUSTOM_COMNET_*` environment variables (8 with UCX)
-   4 `LMOD_CUSTOM_MPI_*` environment variables (8 with UCX)
-   16 `LMOD_CUSTOM_CNCM_*` environment variables (24 with UCX as two CPU architectures don't make sense with UCX
    on LUMI)

The names of the environment variables are derived from the above directory structure, but

-   All characters uppercase
-   `/`, `-`and `.` get substituted by an underscore character
-   The resulting name is prefixed with `LMOD_CUSTOM_` and postfixed with `_PREFIX`.

