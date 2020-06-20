# C++ MultiThreading Developing Notes
1. [GDB Commands](#GDB-Commands)
2. [OMP](#OMP)
3. [Debug Tricks](#Debug-Tricks)
4. [Debug Tools](#Debug-Tools)
5. [Guidelines](#Guidelines)
6. [OMP Issues](#Omp-issues)
7. [Basic Structures](#Basic-structures)

## GDB Commands
Once program is stopped, all threads are stopped. We can use
```
info thread #view all threads
thread n #select nth thread 
```

When type s/c/n in the current threads, other threads can either run freely or frozen by the following settings
```make
set scheduler-locking on #other threads will be frozen
set scheduler-locking off #other threads will run freely until current thread stops again
```

breakpoint and watchpoint can be set by thread
```make
break foo.cxx:121 thread 2
watch someVar thread 2
```

By default, when breakpoint is hit, all other threads are stopped, we can toggle this behavior by:
```make
set non-stop on
set non-stop off
```

## OMP
include <omp.h>, compile with -fopenmp (for pragma in code) and link with -fopenmp (for run_time_libraries linking)

simpel example: https://helloacm.com/simple-tutorial-with-openmp-how-to-use-parallel-block-in-cc-using-openmp/

very useful complete pdf: https://www.openmp.org/wp-content/uploads/omp-hands-on-SC08.pdf

omp single vs omp critical: https://stackoverflow.com/questions/33441767/difference-between-omp-critical-and-omp-single

Microsoft's doc is much easier to understand: https://docs.microsoft.com/en-us/cpp/parallel/openmp/2-directives?view=vs-2019#242-sections-construct

Basic OMP APIs
```c++
omp_set_num_threads() // doesn't gaurantee you will get this much thread, think of set_thread_10000
omp_get_num_threads() // Note this only returns correct value in the parallel section, outside returns 1
omp_get_thread_num()
omp_in_parallel() // function run in both serial and parallel region, can use this to differentiate
#pragma omp parallel for schedule(dynamic/static, chunk_size)
for (int i = 0; i < n_jobs; ++i) {
   // each for item is one job 
}
#pragma omp critical(some_omp_mutex) 
{
    // atomic section
}
```
This is the correct way to get the number of threads used by omp
```c++
    int n_thread_after_set = 0;
#pragma omp parallel 
    {
#pragma omp single
        {
            n_thread_after_set = omp_get_num_threads();
        }
    }
```
```c++
#pragma omp critical // big overhead
    {
      some_general_function() // to be executed atomically
    }
#pragma omp atomic // small overhead but limited to pritimive ops
    {
      ++some_int // use hardware atomic feature
    }
```

omp cluases and directives
```c++
default/share/private(local var not init)/firstprivate(init local data from parent)/lastprivate(send last thread local var to parent)
static int foo = 0;
#pragma omp threadprivate(int) // now int has a individual copy for each thread
```

Threadprivate is tricky in compile. For customerized class, we need to do
```c++
lass MyStrWrap
{
  public:
    MyStrWrap(const string& str) : _str(str) {}
    MyStrWrap(const MyStrWrap& other) : _str(other._str) {}
    string _str;
};

class B {
public:
    static MyStrWrap a;
    #pragma omp threadprivate(a)
};
MyStrWrap B::a("zzzz");
```
If we just write:
```c++
class A {
  public:
    int i;
    A()
    {
        i = 10;
    };
};

A a;
#pragma omp threadprivate(a)
```
There will be compile failure 
"t_step_in.cxx:172:29: error: 'a' declared 'threadprivate' after first use
 #pragma omp threadprivate(a)"

## Debug Tricks
### critical section doesn't mean thread safe
`foo()` and `bar()` can both call `non_mt()`, which causes issue
```c++
void foo()
{
#pragma omp critical(foo_mutex) 
{
    some_func1();
    non_mt();
}
}
void bar()
{
#pragma omp critical(bar_mutex) 
{
    some_func2();
    non_mt();
}
}
```

### stop at the constructor/destructor when you see something crash
When you find some crashes of an object, you can stop at the constructor/destructor in the critical session. Then luckily, we will see 2 threads calling the same object.

### use OMP default none to test if VAR is passed in by private/shared
```c++
#pragma omp parallel for default(none) // to check if variables are passed as expected
```

### static variable can instantiated at any time
```c++
static MyClass s_my = 1; 
void foo()
{
   xxxxx
   #pragma omp parallel for
   for (size_t i = 0; i < 10; ++i){
      bar(i, s_my);
   }
   yyyy

}
```
Note that `s_my` can be initialized before omp_parallel or can be init during omp_parallel. We cannot assume `s_my` ctor will do things before parallel region, and is threadsafe.

### finding determinism
1. associate an Unique ID with everything important, so when something went wrong, we can know exactly which object is failing, and set a breakpoint
2. ID can be a static unsigned, and increment by 1 for each new instance
3. add a counter to the API that has issue
4. print and flush for important message in the critical session

## Debug Tools

### helgrind
valgrind --tool=helgrind 
### sunstudio(https://docs.oracle.com/cd/E19205-01/820-0619/820-0619.pdf)
```csh
collector -s 20 # to profile the multithreading locking
-P xxx # to attach to a PID
```

## Guidelines
(https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-concurrency)
1. only lock for a small number of time
```c++
// bad
{lock_gaurd<mutex> g(mu);
int res = foo(); // heavy cal 
cache_val += res;}
// good
int res = foo(); // heavy cal
{lock_gaurd<mutex> g(mu);
cache_val += res;}
```
2. if a routine has multiple mutex, lock them at the same time to avoid deadlock
```c++
unique_lock<mutex> l1(m1, std::defer_lock);
unique_lock<mutex> l2(m2, std::defer_lock);
std::lock(l1, l2);
```
3. don't call unknown function while holding a lock (cycle in mutex) (https://www.modernescpp.com/index.php/c-core-guidelines-sharing-data-between-threads)
4. replace `global/static` with `local/thread_local` variables
5. `wait()` always with a condition because of spurious wake

## OMP issues
some OMP and fork are not competible in GCC (https://bisqwit.iki.fi/story/howto/openmp/#OpenmpAndFork) (https://github.com/pytorch/pytorch/issues/17199) if compile with clang, we don't have such issue

## Basic Structures
### double checking locking
**note the atomic<Widget*>** is very important!!! otherwise pinstance cannot be atomically created in the beginning.
```c++
atomic<Widget*> Widget::::pinstance {nullptr}; 
Widget* Widget::intance() {
  if (pinstance == nullptr) {
    lock_guard<mutex> l(mutW);
    if (pinstance == nullptr)
      pinstance == new Widget();
  }
  return pinstance;
}

```
