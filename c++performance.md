# Performance Profiling Tools
1. [perf](#perf)

## [perf](http://www.brendangregg.com/perf.html#FlameGraphs)
We can use perf to attach to a running program and profile the performance. Firstly, the program needs to be compiled with `-g`. Note we can run both debug and release build with perf. Secondly, make sure the perf is using the correct shared library, otherwise, symbols cannot be read correctly. 
```
ldd perf to check libelf.so
setenv LD_PRELOAD /usr/lib64/libelf.so.1
```
```makefile
perf record -m 64 -g fp -F 599 -p pid_foo

–g [type, min, order], being:
  * Type: Call chain type
    . flat: single column
    . graph: graph tree, with absolute rate
    . fractal: similar to graph, but with relative rate (default)
  * Min: Minimal percent threashold
    . 0.5: (default value)
  * Order: Call graph order
    . callee: callee based call graph (default)
    . caller: inverted caller based call graph
    
perf record -o <file name>
perf report -i <file name>
```

After it's done, we can see which function is dominating.
```makefile
perf report -g
```

And to be fancier, it can show a flamegraph: https://github.com/brendangregg/FlameGraph.
```makefie
  stackcollapse-perf.pl out.perf > out.folded
  flamegraph.pl out.kern_folded > kernel.svg
```
We can also show program performance in real time:

```makefile
perf top -m 64 -p PID -g --call-graph dwarf
```
