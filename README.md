## straceD

Periodically checks for processes that are in D state (uninterruptable sleep, typically "waiting on IO"), straces them if they keep doing that, and stops stracing once they don't.

Meant as an automatic 'what programs are making my drives churn so hard, and more importantly, what for?', though it has other uses.


By default you get a summary (strace's -c argument), and only once either that process has exited, or we decided it's no longer worth following (because it's no longer in in D state).
The latter also so that you get such a summary at all on processes that rarely or never exit, like databases.

If you want a more realtime and much messier feed, use -C to get all the syscalls of the process. You may then also want to use -e to have strace filter them.


## Considerations

You need root.

Programs run more slowly while having strace attached to it.

I think a process can be straced only once at a time, so think about possible clashes with similar tools, debuggers and such.

The 'if they keep doing so' is 'if we see it ps output more than once, half a second apart' (with defaults) is pretty coarse and can miss small things. Though that's tweakable, and arguably a feature.


## Example

```
# straceD -v
Everything is behaving...
Everything is behaving...
Starting to trace process (PID 9673, 'music-sync-disk')
music-sync-disk(9673):
music-sync-disk(9673): strace: Process 9673 attached
music-sync-disk(9673): % time     seconds  usecs/call     calls    errors syscall
music-sync-disk(9673): ------ ----------- ----------- --------- --------- ----------------
music-sync-disk(9673):  76.43    0.270778          40      6767           stat
music-sync-disk(9673):  15.84    0.056105         204       275           getdents
music-sync-disk(9673):   4.93    0.017451           8      2216           lstat
music-sync-disk(9673):   1.66    0.005865          19       309           munmap
music-sync-disk(9673):   0.44    0.001545          11       142           openat
music-sync-disk(9673):   0.32    0.001129           8       144           close
music-sync-disk(9673):   0.21    0.000741           5       142           fstat
music-sync-disk(9673):   0.07    0.000263          10        26           write
music-sync-disk(9673):   0.06    0.000208          15        14           read
music-sync-disk(9673):   0.04    0.000138           4        31           mmap
music-sync-disk(9673):   0.00    0.000016           8         2           brk
music-sync-disk(9673):   0.00    0.000016          16         1           rt_sigreturn
music-sync-disk(9673):   0.00    0.000015          15         1           getpid
music-sync-disk(9673):   0.00    0.000000           0         1           rt_sigaction
music-sync-disk(9673):   0.00    0.000000           0         1           sendto
music-sync-disk(9673): ------ ----------- ----------- --------- --------- ----------------
music-sync-disk(9673) end-of-strace

```

...which was a process doing a large directory treewalk.


## Arguments

```
Usage: straceD [options]

Options:
  -h, --help            show this help message and exit
  -c                    use strace's -c, which prints a syscall summary but
                        not the individual ones (default)
  -C                    use strace's -C, which prints both individual syscalls
                        and a summary
  -f                    use strace's -f, which follows forked processes.
  --forget=FORGET       After how many seconds of behaving (no longer in D
                        state) we forget about a process. Default: 5
  -s SLEEP, --sleep=SLEEP
                        time to sleep between checks, in seconds. Default: 0.5
  -v, --verbose         How much extra detail to print (our own stuff, plus
                        e.g. whether to filter out strace's attach/detach
                        stuff). Can be repeated.
  -e EXPR               expression to pass through to strace -e, for example
                        -e file. Note that unlike strace itself, our option
                        parsing needs that space there. Note also that this
                        also filters the summary output.
```


## TODO

* the subprocess code had zombie issues. Though I think I fixed it, I should read up a bit more on edge cases.

* check whether the duplicate-line-suppression still works, and is sensible at all.

