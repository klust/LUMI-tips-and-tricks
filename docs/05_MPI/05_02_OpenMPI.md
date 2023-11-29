# Open MPI on LUMI

## Build instructions Alfio from the hackathon

```
module load PrgEnv-gnu
module unload cray-mpich
OMPI_VERSION=4.1.6
echo "Install OpenMPI v"$OMPI_VERSION
wget https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-$OMPI_VERSION.tar.bz2
tar xf openmpi-$OMPI_VERSION.tar.bz2 && rm openmpi-$OMPI_VERSION.tar.bz2
cd openmpi-$OMPI_VERSION

# SLURM (default yes), OFI, PMI
make clean
./configure --prefix=$PWD/install_ofi_slurm_pmi/ \
--with-ofi=/opt/cray/libfabric/1.15.2.0 --with-slurm --with-pmi=/usr CC=gcc CXX=g++ FTN=gfortran CFLAGS="-march=znver2" \
--disable-shared --enable-static
make -j 10 install

rm -f install
ln -s $PWD/install_ofi_slurm_pmi install

module unload cray-mpich
export PATH=$PWD/install/bin/:$PATH
export LD_LIBRARY_PATH=$PWD/install/lib:$LD_LIBRARY_PATH

# Test with OSU
wget https://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-5.9.tar.gz
tar xf osu-micro-benchmarks-5.9.tar.gz
rm osu-micro-benchmarks-5.9.tar.gz
cd osu-micro-benchmarks-5.9

./configure --prefix=$PWD/install CC=$(which mpicc) CXX=$(which mpicxx) CFLAGS=-O3 CXXFLAGS=-O3
make -j 10 install

cd install/libexec/osu-micro-benchmarks/mpi/p2p 
srun -n 2 --ntasks-per-node=1 ./osu_bw

# Cannot remember the performance, try to compare to cray-mpich
```

It looks like you can now safely build shared libraries though.

Useful links:

-   [Use at NERSC](https://docs.nersc.gov/development/programming-models/mpi/openmpi/)



## Open MPI 5 for Slingshot 11

Useful links:

-   [CUG paper ORNL](https://www.osti.gov/biblio/1997634)

    The paper says that changes needed in Open MPI itself are present in the Open MPI main and v5.0.x branches,
    but not in branches for older versions.

    One problem with Open MPI on OFI+CXI was that the CXI provider has no special pathway for intra-node messaging.
    Rather than redeveloping the MTL to deal with multiple components concurrently, a new provider was made, LINKx,
    encapsulating CXI but adding the missing features.

-   [Use at NERSC](https://docs.nersc.gov/development/programming-models/mpi/openmpi/)
