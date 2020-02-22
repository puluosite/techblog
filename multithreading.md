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
simpel example: https://helloacm.com/simple-tutorial-with-openmp-how-to-use-parallel-block-in-cc-using-openmp/

very useful complete pdf: https://www.openmp.org/wp-content/uploads/omp-hands-on-SC08.pdf

omp single vs omp critical: https://stackoverflow.com/questions/33441767/difference-between-omp-critical-and-omp-single

Basic OMP APIs
```c++
omp_set_num_threads()
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
