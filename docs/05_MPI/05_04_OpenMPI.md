# Open MPI tips&tricks

From a user Tor worked with: The following environment variables help
to run on multiple nodes:

``` bash
#Required to launch softwares with mpirun and HPE CXI provider
export OMPI_MCA_ras_base_launch_orted_on_hn=1

#To avoid problem of sending too large message
export OMPI_MCA_coll_tuned_use_dynamic_rules=1
export OMPI_MCA_coll_tuned_allreduce_algorithm=5
export OMPI_MCA_coll_tuned_allreduce_algorithm_segmentsize=131072

#To avoid problem of sending too large message
export OMPI_MCA_coll_tuned_bcast_algorithm=4 # or 9
export OMPI_MCA_coll_tuned_bcast_segment_size=131072
```
