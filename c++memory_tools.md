# Memory Debug Tools
1. [valgrind](#valgrind)
2. [massif](#massif)
2. [ASAN](#ASAN)


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
