we can use perf to attach to a running program and profile the performance
```makefile
perf record -m 64 -g fp -F 599 -p pid_foo
```

After it's done, we can use firegraph to see which function is dominating.
```makefile
perf report -g
```
We can also show program performance in real time:

```makefile
perf top -m 64 -p PID -g --call-graph dwarf
```
