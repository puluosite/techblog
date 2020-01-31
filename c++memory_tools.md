# Memory Debug Tools
1. [valgrind](#valgrind)
2. [massif](#massif)
3. [ASAN](#ASAN)
4. [GDB](#GDB)
5. [MALLOC_CHECK_](#MALLOC_CHECK_)
6. [MALLOC_PERTURB_](#MALLOC_PERTURB_)
7. [mprotect](#mprotect_)


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
### GDB_do_more_at_breakpoint
Search for: "Become a GDB Power User" Very awesome topics on using GDB.
(https://github.com/CppCon/CppCon2016/blob/master/Tutorials/GDB%20-%20a%20lot%20more%20than%20you%20realized/GDB%20-%20a%20lot%20more%20than%20you%20realized%20-%20Greg%20Law%20-%20CppCon%202016.pdf)
```make
b main
b _exit
command 1 // which will let you program what to do when hit breakpoint main
> record
```
### gdb_catch_exception (http://www.yolinux.com/TUTORIALS/GDB-Commands.html)
```make
catch throw 
catch catch 
```

## MALLOC_CHECK_
assume you have a double frease:
```c++
class BadMemCls
{
  public:
    BadMemCls() : _it(10) {}
    void print() { cout << "BadMemCls has integer of: " << _it << _i1 << _i2 << _i3 << endl; }
    int _it;
    int _i1 = 20;
    int _i2 = 30;
    int _i3 = 40;

};

void test_use_after_free()
{
    auto* iptr1 = new BadMemCls();
    cout << "class hasn't been freed.\n";
    iptr1->print();

    auto* iptr2 = iptr1;
    delete iptr1;
    iptr1 = NULL;

    cout << "class has been freed.\n";
    iptr2->print();

    cout << "class now being double freed.\n";
    delete iptr2;
}
```

In the normal case, the tool won't crash. However, when you setenv MALLOC_CHECK_ 3 it will do some memory check, and if there is error, it will crash. (https://support.microfocus.com/kb/doc.php?id=3113982#). It can help detect the issue quickly.

## MALLOC_PERTURB_

setenv MACLLOCK_PERTURB_ xxx (1-255) will set memory to that specific number after free/delete
assume you have a double frease:
```c++
class BadMemCls
{
  public:
    BadMemCls() : _it(10) {}
    void print() { cout << "BadMemCls has integer of: " << _it << _i1 << _i2 << _i3 << endl; }
    unsigned _it;
    unsigned _i1 = 20;
    unsigned _i2 = 30;
    unsigned _i3 = 40;

};

void test_use_after_free()
{
    auto* iptr1 = new BadMemCls();
    delete iptr1;
    iptr1 = NULL;
}
```

After delete, `_it2` and `_it3` will be, when MALLOC_PERTURB_ is 255

```make
(gdb) p/t *0x624c48
$36 = 11111111111111111111111111111111
(gdb) p/t *0x624c4C
$37 = 11111111111111111111111111111111
```
and when MALLOC_PERTURB_ 3, will be
```make
(gdb) p/t *0x624c48
$12 = 11000000110000001100000011
(gdb) p/t *0x624c4C
$13 = 11000000110000001100000011
```

Somehow, the first and second unsigned are 0. Don't know why they are not set.

To conclude, if we set this ENV, and we see a crash, whose memory is like `FFFF` or `3333`, then we know it's already freed. Anther point is that we have to turn off our own memory management. The behavior only triggers when system free is called.

## mprotect_
C has a system API mprotect to set the memory as read_only. 
```c
void segv_handler (int signal_number) 
{
    uti_printf("bad memory accessed!\n");
} 

int test_mprotect()
{
    static long PAGE_SIZE = sysconf(_SC_PAGESIZE);

    struct sigaction sa;

    /* Install segv_handler as the handler for SIGSEGV. */
    memset (&sa, 0, sizeof (sa));
    sa.sa_handler = &segv_handler;
    sigaction (SIGSEGV, &sa, NULL);


    char* p = (char*)malloc(1024+PAGE_SIZE-1);
    char* p_align = (char *)(((size_t) p + PAGE_SIZE-1) & ~(PAGE_SIZE-1));
    if (mprotect(p_align, PAGE_SIZE, PROT_READ)) {
        perror("Couldn't mprotect");
        exit(errno);
    }

    char c = p[2000];         /* Read; ok */
    p[2000] = 42;        /* Write; program dies on SIGSEGV */
    return;
}
```
In some cases, if we want to check the who touches memory accidentally, we can use this. For more info: https://linux.die.net/man/2/mprotect ; https://www.tutorialspoint.com/unix_system_calls/mprotect.htm



