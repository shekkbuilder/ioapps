

# Introduction #

IOprofiler is a GUI application that parses a list of IO system calls (currently recorded by `strace`) issued by an application and reports useful information about which files were accessed, when they were accessed, which parts of them, how fast etc.

Besides many advantages, usage of `strace` as a source of traces has its limitations. Please read [strace limitations](StraceLimitation.md) to review them.

# Features #
  * Very easy to use
  * More then 20 system calls processed, see [list](SystemCallList.md).
  * Ability to trace **already running programs**!
  * Ability to trace **all children processes** spawned by the traced parent process. You **can** trace complicated jobs that execute many child scripts.
  * Intelligent and fast file descriptor-to-file name mapping, written in C for performance
    * this includes handling of ` dup, dup2, dup3, pipe, socket, clone ` system calls.
  * Zoomable, easy-to-save plots (both in vector and scalar format)

# Screenshots #
A picture is worth a thousand words, so this is one example:
<img src='http://ioapps.googlecode.com/files/access_pattern3.png' alt='Access pattern diagram of Thunderbird - detail 2' />
Please see [screenshots](IOProfilerScreenshots.md) for more pictures with a detailed explanation.

# Usage #
Please see the step-by-step [manual](IOProfilerScreenshots.md) with screenshots and detailed explanation. Quick summary follows:


> ## Getting traces ##
    * The easiest way how to get traces is to use `ioprofiler-trace` wrapper around `strace`:
```
ioprofiler-trace APPLICATION ARGUMENTS
```
    * _optionally_ [convert](IOReplayConvert.md) the OUT\_FILE to a binary form with `-b` switch and select another output file name with `-o OUT_FILE` switch:
```
ioprofiler-trace -b -o OUT_FILE APPLICATION ARGUMENTS
```


### Deprecated method ###
  * ~~trace the application of interest using this command~~
```
strace -q -a1 -s0 -f -tttT -oOUT_FILE -e trace=file,desc,process,socket APPLICATION ARGUMENTS
```
  * ~~_optionally_ [convert](IOReplayConvert.md) the OUT\_FILE to a binary form using [ioreplay](ioreplay.md)~~:
```
ioreplay -c -f OUT_FILE -o OUT_FILE.bin
```

> ## Running ioprofiler ##
    * run
```
ioprofiler OUT_FILE
```
> > or run `ioprofiler` and load `OUT_FILE` from the File menu.

# Installation #
## Dependencies ##
IOProfiler depends on the following application and libraries:
  * python2.6
  * PyQt4
  * matplotlib

## Installation ##
### From source ###
Download latest tarball from Downloads sections or grab the source from svn and then:

To install just the IOProfiler and not the whole IOapps suite (see [installation instructions](IOAppsInstallation.md) for details), run following commands:
```
make profiler
```
and then
```
make install_profiler
```

This will install ioapps python module and ioprofiler.py to your system.

# Architecture #
IOProfiler is mainly written in Python, using PyQt4 as a GUI framework and Matplotlib library for plotting.

IOProfiler takes advantage of an intelligent file descriptors handling written in C originally for the [IOreplay](ioreplay.md). The code is converted to the python module called `ioapps`.

# Drawbacks #
  * Speed
    * Since the application and, most importantly, the plotting is written in Python, it is a bit slower than a native application. This is not noticeable unless you want to process and plot a graph with hundred of thousands lines (read/write requests), which may take minutes. I am not aware of any way how to avoid this using Matplotlib.
    * Please note that the time and memory consuming file parsing is written in C as `ioapps` python module to overcome any other unnecessary slow down.
  * Memory consumption -
    * All recorded traces are kept in memory, because of the way recorded traces are processed. It takes approximately the same amount of memory as the file size of binary form of the recorded data. This may be improved in the future.
    * Recorded traces are converted to the list of calls for each file. This list is kept only in Python hashmaps, that consumes more memory than strictly necessary. This may be improved in the future. It takes approximately 4x the size of binary form of recorded straces.
    * Again, the Matplotlib library is quite memory hungry in the case one wants a complicated plot with hundred of thousands of lines. It takes approximately 100MB of RAM for every 100 thousands lines to plot. It is not likely to be improved (unless the way we plot the data dramatically changes).