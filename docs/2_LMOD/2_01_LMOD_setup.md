# Cray Lmod setup

## 2 versions of LMOD

Since the **October-November 2023 update** there are two versions of LMOD on the system

1.  A version installed in the system directories, likely one that comes with SUSE linux.
    -   In November 2023 this is LMOD 8.3.1.
    -   Installation directory: `/usr/share/lmod/`
  
2.  A version installed with the PE. 
    -   In November 2023 this is LMOD 8.7.19 which comes with the 23.09 PE.
    -   Installation directory: `/opt/cray/pe/lmod`

The active one after login is the one the comes with the HPE Cray PE, so when re-initialising
be sure to call the initialisation functions of that version of LMOD.


## Problems

-   `module spider` does not work correctly. It does not always show all possible 
    combinations to access a module. This is because the HPE Cray PE doesn't use
    hierarchies as intended by the LMOD developer. Sometimes a combination of two other
    modules loaded in any order triggers a change to MODULEPATH.

-   Noise about problems with the cache. We do note occasional cache problems on LUMI
    but those don't seem to be associated with specific properties of the HPE Cray PE
    Lmod setup but rather with the system not detecting that modules have been added
    to the tree. I guess Linux with Lustre may have no notification system to detect
    the moment of change in a subdirectory to see if a cache file is still valid and
    to ignore (system cache) or regenerate (user cache) it otherwise.
