# Miscellaneous topics

## Using the compilers without the wrappers

| Compiler module | C compiler        | C++ compiler                   | Fortran fixed format | Fortran free format |
|:----------------|:------------------|:-------------------------------|:---------------------|:--------------------|
| cce             | `craycc`, `clang` | `crayCC`, `craycxx`, `clang++` | `crayftn`            | `crayftn`           |
| gcc             | `gcc`             | `g++`                          | `gfortran`           | `gfortran`          |
| aocc            | `clang`           | `clang++`                      | `flang`              | `flang`             |
| rocm            | `amdclang`        | `amdclang++`                   | `amdflang`           | `amdflang`          |

Remarks

-   The Cray clang compilers need to be pointed to the version of gcc to use using the
    `--gcc-toolchain=<value>` option. The wrapper adds something like
    `--gcc-toolchain=/opt/cray/pe/gcc/8.1.0/snos`.
    Otherwise the compile will fail to find a suitable set of include files.

    TODO: Test this.

-   A good trick to figure out which options the PE wrappers add is to use the
    `-craype-verbose` flag on the command line of the wrappers.
