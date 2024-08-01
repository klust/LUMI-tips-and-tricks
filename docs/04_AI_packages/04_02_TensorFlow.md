# TensorFlow

## TensorFlow profiling

*Based on information from Orian.*

TensorFlow has a built-in profiler and this is probably the first thing to try in case of
performance problems with TensorFlow, rather than using the AMD profiling tools.
TensorBoard can then be used to analyse the profiler data.

There is an [article "TensorFlow Profiler in practice: Optinmizing TensorFlow models on AMD GPUs"](https://rocm.blogs.amd.com/software-tools-optimization/tf_profiler/README.html)
in the [AMD ROCm blogs](https://rocm.blogs.amd.com).

**Note:** Orian once said that the root profile log directory should contain a profile
file or TensorBoard will not detect any profile and not show the profile tab. The trick
is to move one of the profile logs to that directory.
