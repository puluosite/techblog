# Memory Debug Tools
1. [valgrind](#valgrind)
2. [ASAN](#ASAN)


## valgrind
[website](http://www.valgrind.org/docs/manual/quick-start.html)
Compile the program with -g and -o0/-o1

valgrind --leak-check=yes --log-file=my.log exe (~20x on your programe)

Check for the following keywords:

- **invalid** (write/read/free)
- **error**
- **definite lost**
- **uninitialised value**


## ASAN
ASAN in GCC/CLANG
