# Memory Debug Tools

## [valgrind](http://www.valgrind.org/docs/manual/quick-start.html)
Compile the program with -g and -o0/-o1

valgrind --leak-check=yes --log-file=my.log exe (~20x on your programe)

- check for invalid read/write; read after write
- check for key word error
- check for definite lost


## ASAN
ASAN in GCC/CLANG


# Common memory errors

## always check asumption

Do you see problem in the following code? 
```c++
vector<int> res1;
vector<int> res2;
void get_result_grom_mgr(mgr, res1, res2);
for (size_t i = 0, ie = res1.size(); i < ie; ++i) {
	do_something(res1[i]);
	do_something(res2[i]);
}
```
You thought res1 and res2 are in pairs. However, someday some one change `get_result_grom_mgr` and res1 and res2 are no longer in pairs. So put your assertion there:

```c++
assert(res1.size() == res2.size());
```
