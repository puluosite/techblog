# Memory Debug Tools
1. [valgrind](#valgrind)
2. [massif](#massif)
3. [ASAN](#ASAN)
4. [GDB](#GDB)
5. [MALLOC_CHECK_](#MALLOC_CHECK_)


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
(https://github.com/CppCon/CppCon2016/blob/master/Tutorials/GDB%20-%20a%20lot%20more%20than%20you%20realized/GDB%20-%20a%20lot%20more%20than%20you%20realized%20-%20Greg%20Law%20-%20CppCon%202016.pdf)
```make
b main
b _exit
command 1 // which will let you program what to do when hit breakpoint main
> record
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
