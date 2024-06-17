# EESSI and LUMI

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


## How relevant is EESSI to the LUMI user?

### Do large software stacks still make sense?

Python users have been using virtual environments for a long time already as for many it is not possible to create
a single environment that contains all packages they need due to version conflicts. 

Obviously any central Python installation will suffer from the same problem. Modules can help to deal with it, but
it could become very confusing to see which modules work with which other modules if a Python installation is not done
via a single Python packages module, but split across tens if not more of modules, and EESSI, being based on EasyBuild,
follows the latter approach.

However, the problems go further than Python packages. Researches use more an more poorly maintained codes, often remainders
from another Ph.D. that are no longer maintained when the Ph.D. is finished and the author moves on to a totally 
different job. Even some major packages don't always seem to 
function properly with the latest version of a library. As a result it becomes impossible to rely on a single version
of a package in a given toolchain, and long discussions about which versions should be included and for which versions
users should use an older toolchain, are really slowing down development of EasyBuild and hence EESSI.

This is currently already very visible as new toolchains often can only support CUDA at a much later date
because the version of GCC on which the toolchain is built, is not yet supported by CUDA.

Apart from that, large flat lists of modules are not really appreciated by users looking for their package, even though
proper use of Lmod search tools can help a lot.

At LUMI, we often see that 

1.  users have workflows that require specific versions of packages to be in a single toolchain so that they 
    can be loaded together, and

2.  more frequently actually, that users have a need for software packages with specific patches applied to them, and

3.  several packages that require licenses are in heavy use and these cannot be offered in EESSI.

Hence there are a lot of problems that EESSI does not solve at all. It would still require heavy building
on top of EESSI, which currently still lacks a mechanism to do that in a way that integrates as nicely as
with already installed software as we have on LUMI. Moreover, building on top of a software stack that you
don't control, is harder than building on top of a software stack that you have built yourself.

As EESSI uses standard EasyConfigs but plenty of hooks to modify them, one should be very aware of where
EasyBuild stores what version of the EasyConfig (the edited one which actually mirrors what is actually
installed, is stored in the `easybuild/reprod` 
subdirectory of each package installation) or check multiple files to understand what an installation
actually includes and does not include. Checking the log file is not that easy as it is compressed,
limiting the tools that can be used to view that file without first uncompressing in a separate
directory (not everybody is a fan of using `bzcat` to pipe through `less` and then use the `less` 
navigation commands to search in that file).


### Lack of support for AMD GPUs

At the time of writing of this paragraph (June 2024), CUDA support in EESSI is still in its infancy.

GPU support in a central, multi-system software stack is even harder than offering optimal binaries for CPUS,
as there are now basically two parameters to take into account. 

In the case of NVIDIA, you have to provide binaries built for various "Compute Capabilities", but also take into
account that depending on the version of the driver found on the system, certain versions of CUDA may not function,
disabling GPU functionality in certain toolchains or offering multiple CUDA versions in a single toolchain to
offer the full range of systems.

The problems are currently worse on AMD GPUs. There you have to take into account the series of the GPU for the
proper instruction set and optimisations, a bit as for CPUs, but moreover, each driver version has basically only 
support for 5 minor versions of ROCm. ROCm is still evolving rather quickly that certainly support for much newer
ROCm versions than the one from the driver is not functional. It is not clear how well older versions are supported.
The official support is very limited, but more versions may function. We do not yet have enough experience on LUMI
with that to offer an answer to this question. It also means that one may even be in a situation when every 
toolchain generation in EESSI supports only a single version of the ROCm libraries, that none of the toolchains
may be guaranteed to function correctly on AMD hardware, and it is more than likely that only one will given that
at the release rate of new ROCm versions the support windows seems to be smaller than one year of releases.
An additional complication may even be that depending on how the CPU and GPU are linked, different settings at
compile time may be beneficial for performance, leading, e.g., to different builds for MI250X on one side and 
MI210/MI250 on the other, or MI300A versus MI300X.

Note also that as of June 2024, there is no ROCm support at all in the official releases of EasyBuild hence 
also no ROCm support possible in EESSI which makes EESSI currently irrelevant for LUMI GPU users.


### Conflicting settings etc.

EESSI comes with its own LMOD environment that is initialised when running the EESSI initialisation script.
That LMOD may have some settings different from the settings chosen on LUMI, so that LMOD may behave differently
which could be confusing to LUMI users. Obviously it is always possible to build a variant of the initialisation
script specifically for a site, but that leads to two other problems:

1.  It may be impossible to enforce users to use that site-specific script (not clear though as it might be that
    EESSI could offer such a thing through site hooks that are enforced if they want to).

2.  It would then be confusing to EESSI users from another site. And the behaviour may no longer align with 
    the EESSI documentation.

Currently this is a rather hypothetical problem though as currently the behaviour of LMOD aligns rather well
with that of LUMI, albeit without the various modules that LUMI offers to customise the behaviour. Moreover,
all versions of the toolchains are thrown together while on LUMI we at least distinguish between various releases
of the HPE Cray PE, which in EESSI would mean separate stacks for the 2022b, 2023a and 2023b toolchains.

We can also never exclude that software installed with EESSI conflicts with software already installed in the OS.
As long as users use only software installed with EESSI, there is no problem, but when they call tools installed
through the system, this may cause problems. It is also less than trivial to use the debugging and profiling
tools that come with the system.


## Management issues

### Who in the LUMI organisation can manage EESSI?

EESSI is better matched with a model where sysadmins are also responsible for the applications and less with
the LUMI model where there is a strict wall between sysadmins and application support, with each their own
zone on the system that is managed differently.

As explained before, supporting CernVM-FS does require a considerable investment from the sysadmin team.
Moreover, EESSI also assumes that the staff integrating EESSI with the system have full control over
`/opt/eessi`, though this might be solvable on LUMI by making this a symbolic link to a `/appl/local` 
subdirectory.

Another issue is that the rhythm of work is not dictated by CSC or LUMI planning, but by the EESSI planning.
E.g., when they decide to make a new toolchain available, it is up to us to quickly build a proper MPI library
that can be injected in that installation as for the foreseeable future it is not possible for them to include
the necessary support for LUMI in their MPI build (as they simply start from specific versions of libfabric also
where the open source version may not yet include CXI support, and in fact, in the current public release - current is
June 2024 - this support is still incomplete according to the HPE CoE). An external organisation dictating when LUST
or CSC sysadmins should do certain work is not acceptable.


### How to organise user support?

The LUST cannot properly provide support for software it does not install as it does not have sufficient control
over the way it is installed, and as in fact it cannot even install directly in the stack without going through a
lengthy procedure assuring that it also works on all other sites, effectively turning the already understaffed LUST
in a support team for other centres also. Since the installation is not done with HPE-provided tools, we can also
not expect much support from the HPE side to solve problems. 

On the other hand, the EESSI community may not understand LUMI sufficiently enough and does not have enough control
over LUMI to deal with issues that are caused by the specific LUMI environment.

The EESSI community also significantly underestimates the impact of the diversity in the HPC landscape and the
combinatorial problem they are running into, with

1.  Support for three different processor architectures (x86, ARM and RISC-V) with all their subarchitectures requiring
    specific optimisations,

2.  Two or three GPU architectures, with different subarchitectures (NVIDIA Compute Capabilities, AMD CDNA generations and
    RDNA versions if you also want to offer compute support on some consumer GPUs) and restrictions in which versions of 
    the userland (that could be offered through EESSI with some license restrictions) can run on which versions of the drivers
    (of which you can have only a single on the system, and controlled by the sysadmins of that system),

3.  Different interconnect architectures that may be hard to support in a single MPI build, or may even be impossible to
    support with the chosen MPI implementation,

4.  Different configuration of the compute nodes with certain daemons disabled or configured differently as expected,
    including a Slurm installation that may not have support for certain features enabled that the Open MPI implementation
    expects to be present,

5.  And novel accelerator architectures that we may expect in the near future, especially for, e.g., more power-efficient
    support for inference in deep learning models.

Basically the project will have to find a middle ground between what they can support in a central installation, through a 
number of popular combinations of the elements above, and what would be easier to support with a fully local software 
build as too many changes are required (and may not even match with versions of software chosen by EESSI). It is impossible
for EESSI to support everything, basically leading to a number of system requirements for compatibility with EESSI that are 
currently not yet clearly formulated. 

The EESSI community has no clear support model yet. The only workable way for LUST would be the same model as used 
for the local software stacks, where the owner of the stack is responsible for support.

### Needed level of trust

With EESSI, you run binaries that are fetched from the internet and where the update policies are determined
by another organisation. This requires an even higher level of thrust in how the stack is managed than is
required for the current local software stacks also offered on LUMI. 

If users find something today but it is removed tomorrow without warning, then this is not a good thing
support-wise. Binaries should also not change or disappear while a job is using them (CernVM-FS should take care of
this?) as otherwise those jobs would crash.

Getting binaries from the internet is always a risk. Getting source code is also not completely safe
as someone may have tampered with it and introduced malicious code. It is a risk that is always present.
Extra for EESSI is that actually you cannot even blame the user or any organisation that is part of 
the LUMI consortium if EESSI is abused by someone to push in malicious code as the control is completely
in hands of an external project.

All this requires thrust in the way the EESSI organisation manages and secures the software stack.
That thrust has been damaged recently though this will not be discussed openly in this publicly available
document.


## Conclusion

EESSI has a few explicit system requirements - e.g., support for CernVM-FS and some kind of access to `/opt/eessi` - 
but likely more implied requirements that are not yet discovered or properly stated, e.g., support for certain
features in system daemons or Slurm, certain driver versions for GPUs or other hardware directly supported by EESSI, ...
Currently LUMI is not compatible with those requirements. Basically, a cluster has to be procured with specific
requirements and extra hardware (for the Squid caches) to be suitable for EESSI, and the importance if this is 
not yet recognised enough by the EESSI community. LUMI is not built for EESSI, due to its interconnect that is
not yet sufficiently supported in open source versions of libraries, due to its novel GPU architecture that is not
yet properly supported, due to the fact that we don't have the additional hardware needed to run CernVM-FS, 
and due to the fact that the way CernVM-FS is typically installed conflicts with the HPE Cray ideas to control
OS jitter for maximum scalability of large applications.

There are a lot of problems that EESSI does not yet solve. In its current incarnation, apart from the requirement for CernVM-FS
which is already a breaking point, it solves very few problems that we currently have and creates a lot of new problems. 
It gives many of the support problems that we already have with containers without the benefit that containers can give
us.

There is a user support problem to which the EESSI community does not yet have a clear answer. 

To us, having something that is easy to adapt to the specific requirements of LUMI, makes some sense but then
a local installation is required as changes may be in unexpected parts of the stack, which on their turn
may propagate to other packages that depend on it. But that is basically more a version of EasyBuild
with some EasyConfigs adapted to LUMI, where one can wonder if the software abstraction layer of 
EESSI is really needed or does more harm (in being an additional source of conflict with tools on the
system that users may want to use) than it is of benefit in having fewer EasyConfigs to modify.

The one other way of managing software that would really be of help to LUMI would be one where a user can
create a very personalised environment, if needed for file system performance in a container, otherwise
on the Lustre file system and in a way that the tools that we have for performance monitoring etc. can be used
as much as possible and not another tool that pushes users in very strict combinations of software that can
be run together without continuously reloading modules between calls to separate steps in the workflow,
and where you can then load that whole personalised environment in a single command (and have multiple
such environments if the need arises, just as users can have multiple Conda environments or multiple
Python virtual environments).


