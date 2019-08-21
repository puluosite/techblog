# Memory Debug Tools
1. [valgrind](#valgrind)
2. [massif](#massif)
3. [ASAN](#ASAN)
4. [GDB](#GDB)


## [valgrind](http://www.valgrind.org/docs/manual/quick-start.html)

Compile the program with -g and -o0/-o1

valgrind --leak-check=yes --log-file=my.log exe (~20x on your programe)

Check for the following keywords:

- **invalid** (write/read/free)
- **error**
- **definite lost**
- **uninitialised value**

## [massif](http://valgrind.org/docs/manual/ms-manual.html)

Massif will track the memory usage in a graph
- valgrind --tool=massif exe
- --time-unit=B to track by time elapse
- \# is the peak memory snapshot
- @ is the detailed memory usage, will show in the log file

## ASAN
ASAN in GCC/CLANG

## GDB
Search for: "Become a GDB Power User" Very awesome topics on using GDB.
```make
b main
b _exit
command 1 // which will let you program what to do when hit breakpoint main
> record
> continue
> end
command 2 // thing to do when hit end
> run
> end


```
