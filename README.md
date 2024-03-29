## straceD

Intended as an automated answer to "what programs are making my drives churn so hard, and why?"

- Periodically checks for processes that are in D state (uninterruptable sleep, typically "waiting on IO" and then usually "waiting on disk"), 
- ...and `strace`s them if they keep doing that. 
- Stops stracing and gives a summary when 
  - it stops being in D state, OR
  - the traced process stops running, OR
  - you press Ctrl-C on _this_ program (helps give summaries on processes that rarely or never exit, like databases and other daemons)

By default you get a summary (strace's `-c` argument), only once we stop tracing for any of those reasons.
If you want a more realtime (and much spammier) feed, add `-C` to get all the syscalls of the process. You may then also want to use `-e` to have strace filter a bit more.


## Considerations

You need root rights.

Programs run more slowly while having strace attached to it.

I'm fairly sure that a process can be straced by only one thing at a time, so consider possible clashes with similar tools, debuggers and such.

The "if it stays in D state" check is actually "if runs of `ps` reports D state more than once, some time apart", which is coarse and can miss things.
I consider this a feature because it ends up ignoring some (series of) shorter-but-not-instant syscalls (like smartd's), but for specific purposes you may wish to lower it.


## Example

```
# straceD
[20:24:56.17] Starting to trace process music-sync-disk(28324)
[20:25:45.51] music-sync-disk(28324):
[20:25:45.51] music-sync-disk(28324): % time     seconds  usecs/call     calls    errors syscall
[20:25:45.51] music-sync-disk(28324): ------ ----------- ----------- --------- --------- ----------------
[20:25:45.51] music-sync-disk(28324):  80.21    0.434608          30     14722           stat
[20:25:45.51] music-sync-disk(28324):  13.77    0.074588         174       429           getdents
[20:25:45.51] music-sync-disk(28324):   4.69    0.025420           5      5021           lstat
[20:25:45.51] music-sync-disk(28324):   0.58    0.003118          11       293           munmap
[20:25:45.51] music-sync-disk(28324):   0.29    0.001575           7       211           openat
[20:25:45.51] music-sync-disk(28324):   0.22    0.001198           6       214           close
[20:25:45.51] music-sync-disk(28324):   0.13    0.000721           3       211           fstat
[20:25:45.51] music-sync-disk(28324):   0.07    0.000360          10        35           write
[20:25:45.51] music-sync-disk(28324):   0.02    0.000125           8        16           read
[20:25:45.51] music-sync-disk(28324):   0.01    0.000068          11         6           mmap
[20:25:45.51] music-sync-disk(28324):   0.00    0.000015          15         1           rt_sigreturn
[20:25:45.51] music-sync-disk(28324):   0.00    0.000015          15         1           getpid
[20:25:45.51] music-sync-disk(28324):   0.00    0.000004           4         1           brk
[20:25:45.51] music-sync-disk(28324):   0.00    0.000004           4         1           rt_sigaction
[20:25:45.51] music-sync-disk(28324):   0.00    0.000000           0         1           sendto
[20:25:45.51] music-sync-disk(28324): ------ ----------- ----------- --------- --------- ----------------

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

* portablity, e.g. compatibility with non-linux `ps` implementations

* the subprocess code had zombie issues. I think I fixed it, but should still read up more on edge cases.

* add 'ignore process called X' parameter, e.g. to ignore the database on a database host

