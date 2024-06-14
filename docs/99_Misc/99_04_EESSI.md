# EESSI

## Some background

The only currently supported way of distributing EESSI is via
[CernVM-FS](https://cernvm.cern.ch/fs/), a system based on web technology developed to distribute
the software for the LHC experiments.

It was later picked up by the organisation formerly known as Compute Canada, now part of the
[Digital Research Allicance of Canada](https://alliancecan.ca/) (called The Alliance further
in this text, as that is what they also often do), as a way to manage a central
software stack that is available on all their HPC clusters that are geographically spread over
Canada, but managed in a central (so people working on that stack also have access to all of
these clusters and their say in the specifications when the clusters are acquired, 
which becomes important later in the discussion).

To make this cross-cluster installation feasible, they had to abstract several aspects of the OS
also, as different versions of Linux could require different settings for software packages build
on top of it. So they first install a set of OS libraries as a basis. Currently they use 
[Gentoo Prefix](https://wiki.gentoo.org/wiki/Project:Prefix) for that as this enables installation
in a separate set of directories (and hence in the CernVM-FS-served tree and seemingly without coming in 
conflict with the vendor-provided OS and cluster management tools).

This setup served as the basis for the [EESSI architecture](https://www.eessi.io/docs/overview/).

*Though EESSI builds on the architecture used by The Alliance, it is not completely the same. E.g.,
both projects had to deal with not being allowed to spread certain libraries via CernVM-FS.
The primary example are the NVIDIA libraries. The Alliance [assumes that they are installed in
a certain location](https://docs.alliancecan.ca/wiki/Accessing_CVMFS#CUDA_location) while 
EESSI assumes that there is a subdirectory `/opt/EESSI` which will be used for a number of
site-specific adaptations that may be needed, including the injection of some CUDA libraries
that may not be distributed by them for sites with NVIDIA GPUs.*

*Another difference is the way they use EasyBuild. EESSI is on paper a separate project from EasyBuild,
but it is really mostly run by a subset of people running EasyBuild and it follows very closely 
EasyBuild. The Alliance however, hasn't been shy in the past of going for 
some of their own EasyConfigs](https://github.com/ComputeCanada/easybuild-easyconfigs) 
to solve problems specific to their users and sites, even though they do contribute back and
participate actively in the development of EasyBuild. (Notice that the linked repository, though
a clone of the central EasyConfigs repository, has its own main branch that can be a lot of
commits ahead from even the developers branch of the [main EasyBuild EasyConfigs repository](https://github.com/easybuilders/easybuild-easyconfigs/tree/develop))*

*What they have in common is that they also distribute their stack to other interested parties.
For The Alliance, this is [documented in their documentation](https://docs.alliancecan.ca/wiki/Accessing_CVMFS),
for EESSI this is inherent to their project as they have no control or access to even the 
majority of the clusters that use their software stack.*

Links

-   [Digital Research Alliance of Canada documentation](https://docs.alliancecan.ca/wiki/Technical_documentation)


## Why is EESSI not a very good idea on LUMI?

### CernVM-FS on a Cray system.

CernVM-FS is a remote file system based on web technology. A textbook setup requires:

1.  A daemon running on each compute node providing CernVM-FS (which requires the `/cvmfs` mount point),
2.  Cache space for each CernVM-FS daemon, and
3.  A web cache, certainly for larger sites.

However, Cray OS on the LUMI compute nodes starts from a very different idea. Though it is a derivative
of SUSE Enterprise Linux 15 (now SP4, after the planned system upgrade of August 2024 SP5), it has many 
daemons disabled to reduce background noise from the OS, also known as OS jitter, as this restricts 
scalability of very large parallel applications. In fact, even with those measures, during the acceptance testing
of the GPU partition on LUMI, scaling of some codes did not meet the acceptance criteria,
leading to the necessity to make one core unavailable to users to reserve it
for OS processes. (In essence, the AMD driver stack was causing too much OS jitter.)
As part of the measures to control OS jitter, HPE Cray also offers only one remote file system on the compute
nodes, Lustre, while all other remote file systems are offered through a system they call DVS or Data
Virtualisation Service, which essentially forwards all file system calls to management nodes that run 
the actual client (a bit like a remote procedure call).  

Note that the documentation of CernVM-FS does mention that some sites [have experimented with solutions to 
make CernVM-FS available on the compute nodes in other ways](https://cvmfs.readthedocs.io/en/stable/cpt-hpc.html#nfs-export-with-cray-dvs)
but the documentation itself mentions that it is not recommended for performance and in fact also mentions
[another difficulty (with inodes) for which there does seem to be a workaround](https://cvmfs.readthedocs.io/en/stable/cpt-configure.html#export-of-cvmfs-with-cray-dvs).

Both paths are not an option on LUMI. It is clear that adding a permanently enabled daemon on the compute nodes, and one that is
not supported by our vendor, is not an option. It endangers the scalability and it would slow down every system update
process on LUMI as LUMI would first have to be updated by the vendor to a vendor-supported configuration after
which the additional daemon would have to be reinstalled and integrated in the management environment. Our compute nodes
are diskless, which is not optimal for the caching requirements of CernVM-FS. There is a scheme though (via a loopback 
filesystem) to do that on shared
storage in a metadata server-friendly way, which is not optimal for performance, but the load on the shared file system would
not be worse than with traditionally installed software. Lastly, a number of Squid proxies and preferably a Stratum-1 server
would be needed to keep the load on the network connection to the internet low and to not put 
an unacceptable load on whatever external server the software is fetched from. 
[A presentation by one of the developers at the 6th EasyBuild User Meeting](https://easybuild.io/eum21/#cvmfs-talk) 
advised one server per 500 nodes, though that again
is a number from 4 years ago. Still, if the goal of the development project is to make EESSI available on all of LUMI to offer
the software from the MultiXScale project, it requires a considerable amount of hardware for which there is no budget in the 
LUMI project.

The second path is not an option either. Firstly, it is high risk as the CernVM-FS documentation recommends against it and
as experience has learned that getting something to work with DVS is non-trivial. Secondly,
it also requires a considerable investment in hardware as the necessary management nodes are needed to host the CernVM-FS
instances to which DVS will forward the requests. One could likely do with far less cache servers though as in this scenario each
CernVM-FS instance would likely be serving multiple nodes, reducing the number of instances. It is likely a stratum-1 server
(or two in some kind of high reliability setup) would be enough. However, as now we need CernVM-FS in the management domain,
it still interferes with system updates. 

Both scenarios also require a significant time investment from the sysadmins, both for the first installation (where the second
option, as it is more experimental, may be even worse) and to maintain the installation and keep it secure. Even if we would have
the necessary hardware and people resources to do the work, it is not something that could be done on the fly on a system 
like LUMI, but something that needs to be carefully tested before deployment,
and that deployment can only be done during a system maintenance interval.
As the testing can only be conclusive when done at scale, it is something that is not really feasible now that the pilot projects
have ended, as it would confront users with a potentially unstable or underperforming system during testing and as 
considerable capacity would be needed to put a realistic load on CernVM-FS.

Note also that SUSE Linux is only a B-platform for CernVM-FS (at least in the most recent information we could find on this),
so the question is whether you even want to run this with that level on support on a machine like LUMI where each day of downtime
is worth probably 150,000 EURO or more.

It is also possible to run CernVM-FS through a tool called [`cvmfsexec`](https://github.com/cvmfs/cvmfsexec). However, this
tool is also of no use on LUMI as (1) user name spaces are disabled due to security concerns and (2) fuse mounts are not 
available either, with no planned date for future availability as they will not be made available until a robust clean-up
script is developed to clean up a node when a job ends.

Technically speaking, it is possible to run CernVM-FS inside a singularity container on LUMI. However, doing so at scale
without proper caching infrastructure (as LUMI cannot offer that) has the potential to create an immense network load 
on both the outgoing network
connections of LUMI and on the stratum-1 server that would be used, which in turn may lead to complaints from the admins of
that server to CSC. Both are not desirable, and using containers at scale as a backdoor will be considered abuse of the
LUMI infrastructure.

There are enough fine ways of distributing software that are a better match with LUMI and requiring less effort from sysadmins
that CernVM-FS for software distribution should not be a priority for any future development.


### EESSI software on LUMI

EESSI is currently not fully compatible with LUMI.

A first - and major - problem is their current installation of OpenMPI. It does not use the proper libfabric library that
provides good performance on LUMI. Apart from that, another issue might be compatibility with our Slurm installation, but
that was not tested to write this review. The injection mechanism they are working on may or may not be a solution - depending
also on the versions of MPI they want to install - but that development is not part of this development project,
and it also requires access to a directory that is not managed by the application support team of LUMI but
by sysadmins (though a symbolic link might provide a solution here).
As the goal is to develop a workflow for CI/CD to evaluate performance and scalability, that is of no use as the 
answer is known: bad performance and bad scalability for many codes without proper MPI support for the Slingshot
interconnect. Prior development of a proper OpenMPI configuration would be needed first,
but is not part of the project.

Neither EESSI nor EasyBuild, that is used to install software in EESSI,
currently support ROCm, nor is the development of that support part of the project. Hence we can only conclude that 
the proposed software is not suitable for LUMI-G. Moreover, for the development of the actual CI/CD pipeline itself, it doesn't
matter if it is done on LUMI-C or LUMI-G as both offer the same environment apart from the GPUs. Hence there is no need
for an allocation on LUMI-G.

Combining the above two elements: Until the next system update, it is also impossible to build an OpenMPI configuration with proper support 
for GPU-aware MPI. After that update, the libfabric library and underlying Slingshot software should be OK
to install the OpenMPI 5 branch with ROCm support, but it is not clear yet if the integration with Slurm 
would work.

EESSI developers and users should be aware that some software in the compatibility layer conflicts with some system tools. 
The same may or may
not hold for software in the EESSI stack itself. As long as only software in the compatibility layer and EESSI stack is
used, this should not cause any problem. However, using software from the main system directories may not always work
and produce warnings or crashes. We have seen issues in the past with EasyBuild-installed software on SUSE 15 SP4. 
Though we were not able to try out a recent version of EESSI on a configuration equivalent to the current LUMI setup, we did note issues on a
SUSE 15 SP5 setup that should be similar to the setup LUMI will offer after the August 2024 system update, though
on that version of the OS, they did not lead to crashes.

EESSI developers should be aware that the compute nodes on LUMI do not offer all daemons running in the background that one would
expect on a regular Linux system. This may or may not have consequences for software installed in EESSI. We have run into
problems running regular EasyBuild-installed packages on LUMI. As the compatibility layer contains libraries that may be
different from those on LUMI, but not the daemons that may be needed, this layer does not solve all potential compatibility
problems and more software than expected may need LUMI-specific configurations rather than the generic EESSI build.

