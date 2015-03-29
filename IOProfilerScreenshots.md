This page describes IOProfiler on a real life example - IO profiling of starting Thunderbird v17.0.4. Ever wondered what taking so long to open  Thunderbird? Maybe there is just too much IO activity.

# Getting traces #
At first, we need to get traces of thunderbird during start up. We could use `strace` directly, or ioprofiler-trace which is a convenient wrapper around it:

```
user@hostnaname $ ioprofiler-trace thunderbird
```

Don't be surprised if the loading takes even longer than usual - that's because thunderbird makes quite a lot of syscalls so `strace` has quite a big overhead. But that's ok since we are now interested in access pattern not in exact timings.

Once it loads, we just close the thunderbird window and check we have a trace log called "ioproftrace.log" because that's the default name of the log (one can specify another name using `-o` command line option):

```
user@hostnaname $ ls -l ioproftrace.log 
-rw-r--r-- 1 user user 74890554 Apr  4 22:04 ioproftrace.log
```

It seems OK, so we can run ioprofiler over it:
```
user@hostnaname $ ioprofiler ioproftrace.log 
```

# Main application window #
The main window lists all the files that were accessed during tracing of the program. The summary page contains number of reads/writes, total number of bytes transferred and time spent in IO operations for each file listed. In the screenshot below, the list fast sorted by number of calls.

<img src='http://ioapps.googlecode.com/files/main_window.png' alt='Main Window' />

As you can see, quite a lot of files are accessed. In this case, the most accessed files are .msf files, for example thunderbird read more than 300MiB from "qt-interest.msf", even though it is just a 16MiB file! It implies that some parts of the file must have been read multiple times.

After clicking the "Details button", the access pattern diagram is shown which should help us reveal what happened exacly.

# Detailed information about access to a file #
The access pattern diagram shown, although not visible at the firt glance, internally consists of thousand of lines - each line represents one read or write request, reads in this case. The length of each line represents number of bytes transferred during the IO request - the request's size. The horizontal angle of each line represents time of how long the request took. The longer it took the steeper the line will be. Very fast requests look like a straight horizontal line (of course, it all depends on the scale of the diagram).

There is a list of all read requests on the left. The selected reads in the list can be highlighted in the plot.

<img src='http://ioapps.googlecode.com/files/access_pattern1.png' alt='Access pattern diagram of Thunderbird - overview' />

The good news is that the file is read sequentially from the macroscopic view (see the pictures below for detailed view). The diagram consists mainly of 3 sections, the first one from start of the file to around 12MiB offset, which seems to be made of 8KiB sequential reads (see list of calls on the left), the second part with a logarithmic shape from 12MiB to around 13.6MiB, and the last part from 13.6MiB to the end of the file which took majority of time, more then 6 seconds.

So have a closer look at all these three parts using <img src='http://ioapps.googlecode.com/files/zoom_button.png' alt='zoom button' /> zoom button.

<img src='http://ioapps.googlecode.com/files/access_pattern2.png' alt='Access pattern diagram of Thunderbird - detail 1' />

This detailed view of the first part of the diagram (see offsets) confirms our theory that it consits of strictly sequential 8KiB reads. One of the read was highlighted by clicking on it in the graph. This looks fine.

<img src='http://ioapps.googlecode.com/files/access_pattern3.png' alt='Access pattern diagram of Thunderbird - detail 2' />

This time, we zoomed to the second part of the access diagram and clicked on one of the lines in the diagram. But there are three read requested highlighted in the list! Please note that they have the same offset. Let's have even closer look.

<img src='http://ioapps.googlecode.com/files/access_pattern4.png' alt='Access pattern diagram of Thunderbird - detail 3' />

So from "microscopic" view, the access pattern is not the sequential as it seemed to be. Some parts of the read three times. But this still not explain the difference of 300MiB reads of total and 16MiB file. So let's see the last part of the file.

<img src='http://ioapps.googlecode.com/files/access_pattern5.png' alt='Access pattern diagram of Thunderbird - detail 4' />

Again, we clicked on what looked like as one line in the plot. But multiple reads got selected, all of them with the same offsets. Having a closer look...

<img src='http://ioapps.googlecode.com/files/access_pattern6.png' alt='Access pattern diagram of Thunderbird - detail 4' />

we see again that some parts of the file are read multiple times, here 20 times and more...

# Histograms #
We can confirm that majority of reads are 4KiB even though 12MiB of file was read by 8KiB requests:
<img src='http://ioapps.googlecode.com/files/histsize.png' alt='Histogram size' />

or see speed histogram. It cleary shows that majority of reads were cached already (I don't have that fast harddrive :) so the main overhead was making the syscall itself.
<img src='http://ioapps.googlecode.com/files/histspeed.png' alt='Histogram speed' />