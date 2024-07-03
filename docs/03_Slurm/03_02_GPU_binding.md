# GPU binding issues

## Nature of the problem

When using per-task cgroups, P2P IPC doesn't work as a communication mechanism as GPUs 
cannot find each other. It is used, e.g., by Cray MPICH for all but very short messages 
when communicating directly between GPUs in a node.

See also the [schedMD case #17875](https://support.schedmd.com/show_bug.cgi?id=17875).

## Setups that work

TODO
