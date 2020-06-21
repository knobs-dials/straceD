## straceD

Periodically checks for processes that are in D state (IOwait),
straces them if they do so persistently, and stops once they behave.

Meant a relatively automatic 'what programs are making my drives churn so hard / are bothered by this?' ...though has other uses.


Defaults to only summarizing the calls (strace's -c parameter),
because when disk contention happens, it tends to create a choir of processes
and this gives you a little more hope at guessing which one was the cause.

...note this does mean you only get output when that process exits.
If you want all the syscalls, use the -C argument instead.

## Example

```
# straceD
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

...which is a process doing a large directory treewalk.


## Arguments

Usage: straceD [options]

Options:
  -h, --help       show this help message and exit
  -c               use strace's -c, which prints a syscall summary but not the
                   individual ones (is the default)
  -C               use strace's -C, which prints both individual syscalls and
                   a summary (default is -c)
  -f               use strace's -f, which follows forked processes.
  --forget=FORGET  How quickly to forget a process that is now behaving itself
                   (no longer in D state). Default: 5
  -s, --sleep      time in seconds to sleep between checks. Default: 1.0
  -q, --quiet      suppress some of our own stdout messages, and some of
                   strace's attach/detach stuff
