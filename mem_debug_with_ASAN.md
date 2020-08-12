## Stories when I tried to apply ASAN/TSAN in project

### order of finding function for a program
1. Program will try to link function from `LD_PRELOAD`, 
2. if not found, then it's the program itself, 
3. shared_library has the last priorty.

Among the shared library, if multiple functions appears at the same time, it will use the first function in the order of `ldd binary`.
So, there might be issue: http://optumsoft.com/dangers-of-using-dlsym-with-rtld_next/
To get the correct function, need to **properly order the libraries** in the linking stage.


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

### calloc shim issue
So calloc is tricky. The following **won't** work:
```c
extern "C" void* calloc(size_t nmemb, size_t sz)
{
    void* ret = NULL;
    static void *(*orig_calloc)(size_t, size_t) = NULL;
    if (orig_calloc == NULL)
        orig_calloc = (void *(*)(size_t, size_t))dlsym(RTLD_NEXT, "calloc");

    ret = orig_calloc(nmemb, sz);
    printf("In my own calloc\n");
    return ret;
}
```

It's because dlsym will call calloc, and it goes into the infinite loop. So we need to have a global flag to indicate it's called by dlsym.
```c
/* THIS WORKS*/
#define CALLOC_DLSYM_BUF_SIZE 100
static char* calloc_dlsym_buf[CALLOC_DLSYM_BUF_SIZE];
static int in_dlsym_in_calloc = 0;
extern "C" void* calloc(size_t nmemb, size_t sz)
{
    void* ret = NULL;
    static void *(*orig_calloc)(size_t, size_t) = NULL;
    if (in_dlsym_in_calloc) {
        // This code will only execute once, to allocate 24 bytes
        // when we call dlsym to initialize static variable orig_calloc below.
        // To avoid an infinite recursion (since dlsym calls calloc), we need
        // to allocate space for dlsym from a special buffer, just this once.
        int i;
        size_t bytes = nmemb*sz;
        if (bytes == 0)
            return (void*) 0;
        for (i=0; i<bytes && i<CALLOC_DLSYM_BUF_SIZE; ++i)
            calloc_dlsym_buf[i] = 0;
        return calloc_dlsym_buf;
    }

    if (orig_calloc == NULL) {
        in_dlsym_in_calloc = 1;
        orig_calloc = (void *(*)(size_t, size_t))dlsym(RTLD_NEXT, "calloc");
        in_dlsym_in_calloc = 0;
    }
    ret = orig_calloc(nmemb, sz);
    printf("In my own calloc\n");
    return ret;
}
```

### dlsym
dlsym is used to load shared library, along with dlopen, dlclose. Somehow, dlopen/dlclose doesn't work for me on RedHat6.5. Some people suggest
it's a better to use dlopen/dlclose to choose the function in (shared lib)[http://optumsoft.com/dangers-of-using-dlsym-with-rtld_next/]

### Eli Bendersky's blog
This guy has very interesting blog, check this for the libraries:
https://eli.thegreenplace.net/2013/07/09/library-order-in-static-linking
https://eli.thegreenplace.net/2011/11/03/position-independent-code-pic-in-shared-libraries

He also has a blog series of async IO
https://eli.thegreenplace.net/2017/concurrent-servers-part-1-introduction/

### Building with ASAN/TSAN
Need to add `-fsanitize` for both compiler and linker, `-fsanitize-recover=address` will allow `setenv ASAN_OPTIONS halt-on-error=0` work.
```make
ifeq ($(USE_ASAN),yes)
	CFLAGS +=  -fsanitize=address -fno-omit-frame-pointer -fsanitize-recover=address -DUSE_ASAN
	LINKER_FLAG += -fsanitize=address -fno-omit-frame-pointer -fsanitize-recover=address
endif

ifeq ($(USE_TSAN),yes)
	CFLAGS +=  -fsanitize=thread -DUSE_TSAN
	LINKER_FLAG += -fsanitize=thread
endif
```

However, `__asan::AsanInitInternal` will call `dlsym` and then it will call `calloc`/`malloc`/`free`
If these functions are instrumented by ASAN, it will mark some ASAN memory address, while ASAN is still initilization!!
This will cause segmentation fault, and we can only see it by debugging the assembly code.

So, we need to have the following macro to stop instrumenting these functions
```c
#define NO_ASAN_TSAN 
#ifdef USE_ASAN 
    #undef NO_ASAN_TSAN
    #define NO_ASAN_TSAN __attribute__((no_sanitize_address)) 
#endif 
#ifdef USE_TSAN
    #undef NO_ASAN_TSAN
    #define NO_ASAN_TSAN __attribute__((no_sanitize_thread)) 
#endif


extern "C" void*
NO_ASAN_TSAN
malloc/calloc/free
{...
```

Another interesting thing is in some building system, it will execute binary in Makefile, so if ASAN fails, building system will stop.
So we will have: `setenv ASAN_OPTIONS detect_leaks=0` during building.

### Running ASAN/TSAN
Run with the following env in my project:
```csh
setenv ASAN_OPTIONS "detect_leaks=0:halt_on_error=0"
setent TSAN_OPTIONS detect_deadlocks=0
```
