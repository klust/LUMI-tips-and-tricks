# Cray Lmod setup

TODO

## Problems

-   `module spider` does not work correctly.

-   Noise about problems with the cache. We do note occasional cahce problems on LUMI
    but those don't seem to be associated with specific properties of the HPE Cray PE
    Lmod setup but rather with the system not detecting that modules have been added
    to the tree. I guess Linux with Lustre may have no notification system to detect
    the moment of change in a subdirectory to see if a cache file is still valid and
    to ignore (system cahce) or regenerate (user cache) it otherwise.
