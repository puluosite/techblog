# in this file, we will discuss how to debug memory issue in the C++, and common errors that will cause memory issues that are difficult to debug
valgrind --leak-check=yes --log-file=my.log exe (~20x on your programe)
check for invalid read/write; read after write

ASAN in GCC/CLANG
