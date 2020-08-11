## Stories when I tried to apply ASAN/TSAN in project

### shim malloc/calloc/free

In the program, we can intercept malloc/calloc/free using shim shared library.
```c
// in my_mem.c
extern "C" void* malloc(size_t sz)
{
    void* ret = NULL;
    void* (*orig_malloc)(size_t) = (void* (*)(size_t))dlsym(RTLD_NEXT, "malloc");

    Dl_info dlInfo;
    if(dladdr((void*)orig_malloc, &dlInfo)) {
        printf("malloc in: %s\n", dlInfo.dli_fname);
    }
    ret = orig_malloc(sz);
    return ret;
}
```
then we can compile `my_mem.c` into shared library:
```make
g++ -c -std=c++11 -Wall -g  -fPIC  my_mem.c
g++  -shared -o my_mem.so my_mem.o
```
and use `setenv LD_PRELOAD ./my_mem/so` to run the
NOTE **set environment LD_PRELOAD in GDB, not in term, otherwise GDB might fail**
