# Support for huge pages

## Sources

-   ["HugeTLB Pages" in the Linux kernel docs](https://www.kernel.org/doc/html/latest/admin-guide/mm/hugetlbpage.html)

-   ["intro_hugepages" in the Cray CPE docs](https://cpe.ext.hpe.com/docs/latest/craype/intro_hugepages.html)

-   man pages for some of the tools

## Hardware pages

The x86 supports up to three page sizes:

-   4kB: Regular pages

-   2MB: Huge pages, mapped via the Page Directory entries

-   1GB: Gigantic pages, mapped via Page Directory Pointer Table entries which is not supported on
    all x86 processors and also requires kernel support besides hardware support.


## Software

Huge pages support in Linux is provided through `libhugetlbfs`.

Request which page sizes are supported on a system: USe `hugeadm --pool-list`.
E.g., on a LUMI login node:

```
$ hugeadm --pool-list
      Size  Minimum  Current  Maximum  Default
   2097152        0        0   515580        *
   4194304        0        0   257790
   8388608        0        0   128895
  16777216        0        0    64447
  33554432        0        0    32223
  67108864        0        0    16111
 134217728        0        0     8055
 268435456        0        0     4027
 536870912        0        0     2013
1073741824        0        0     1006
2147483648        0        0      503
```

The star denotes the default size which is 2M and corresponds to the hardware huge page size.

If a larger size is requested, it actually just creates multiple 2M pages for each 
of the larger pages, and the number of those supported, suggest that too. It may even make no sense
to use another size than 2M. I don't know if the 1G gigantic pages can be used on LUMI.

At link time, code and data segments are properly aligned according to the page sizes that are
used at that time.

At runtime, the binary will

-   Use hugetlbfs to back static data and heap.

-   Automatically allocate any malloc() or static segment allocations on huge pages (via libhugetlbfs).

-   Fall back to standard pages if huge pages of the requested size aren’t available.
