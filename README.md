## straceD

Periodically checks for processes that are in D state (IOwait),
straces them if they do so persistently, and stops once they behave.

Meant a relatively automatic 'what programs are making my drives churn so hard / are bothered by this?' ...though has other uses.

Defaults to only summarizing the calls (strace's -c parameter),
because when disk contention happens, it tends to create a choir of processes
and this gives you a little more hope at guessing which one was the cause.

```
# straceD
Everything is behaving...
Everything is behaving...
Everything is behaving...
Not stracing, probably a kernel process (PID 607, 'jbd2/sdc1-8')
Starting to trace process (PID 26099, 'smartctl')
smartctl(26098):
smartctl(26099):
smartctl(26098): strace: Process 26098 attached
smartctl(26098): % time     seconds  usecs/call     calls    errors syscall
smartctl(26098): ------ ----------- ----------- --------- --------- ----------------
smartctl(26098):   0.00    0.000000           0        22           write
smartctl(26098):   0.00    0.000000           0         1           close
smartctl(26098):   0.00    0.000000           0         1           brk
smartctl(26098):   0.00    0.000000           0         1           ioctl
smartctl(26098): ------ ----------- ----------- --------- --------- ----------------
smartctl(26098) end-of-strace
Reader thread for smartctl(26098) finished
Everything is behaving...
Everything is behaving...
```

...which frankly is an example of a less-useful output
