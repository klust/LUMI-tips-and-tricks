# Container tips and tricks

## Check how many loop devices are in use

``` bash
ls -la /dev/loop* | wc -l
```

A clean node on LUMI should have 28-31 in use. 257 is currently the maximum on compute nodes.
If you see this number on an exclusive node, something is clearly wrong.
