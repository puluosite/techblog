# C++ MultiThreading Developing Notes
1. [GDB Commands](#GDB-Commands)
2. [OMP](#OMP)
3. [Debug Tricks](#Debug-Tricks)

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
## OMP
simpel example: https://helloacm.com/simple-tutorial-with-openmp-how-to-use-parallel-block-in-cc-using-openmp/

very useful complete pdf: https://www.openmp.org/wp-content/uploads/omp-hands-on-SC08.pdf

omp single vs omp critical: https://stackoverflow.com/questions/33441767/difference-between-omp-critical-and-omp-single

Basic OMP APIs
```c++
omp_set_num_threads() // doesn't gaurantee you will get this much thread, think of set_thread_10000
omp_get_thread_num() // Note this only returns correct value in the parallel section, outside returns 1
omp_get_thread_num()
#pragma omp parallel for schedule(dynamic/static, chunk_size)
for (int i = 0; i < n_jobs; ++i) {
   // each for item is one job 
}
#pragma omp critical(some_omp_mutex) 
{
    // atomic section
}
```

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


