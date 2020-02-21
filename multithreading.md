# C++ MultiThreading Developing Notes
1. [GDB Commands](#GDB-Commands)
2. [OMP](#OMP)

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
Basic OMP APIs
```c++
omp_set_num_threads
#pragma omp parallel for schedule(dynamic/static, 4)
for (int i = 0; i < n_jobs; ++i) {
   // each for item is one job 
}
#pragma omp critical(some_omp_mutex) 
{
    // atomic section
}
```
