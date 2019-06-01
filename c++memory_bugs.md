# Common Memory Errors
1. [Check Any Compilation Warnings](#Check Any Compilation Warnings)
2. [Design Copy Constructor When Destructor Free Items](#Design Copy Constructor When Destructor Free Items)
3. [Always Check Asumption Using *Assert](#Always Check Asumption Using *Assert)


## Check Any Compilation Warnings

Compiler will also show the uninitialized local variables.
**HOWEVER, compiler doesn't show unintialized class members**

- always make sure constructors init all members

## Design Copy Constructor When Destructor Free Items

Check following code:

```c++
class A {
	A() { _iptr = new int(0);}
	~A() { delete _iptr; }
	int* _iptr;
};

vector<A> vec_A;
for (size_t i = 0; i < 100; ++i) {
	vec_A.push_back(A());
}
```

Then we will have a bad memory issue. During insertion, `vec_A` will resize, and copy constructor will called to move existing elements in vec. Then some As' ~_iptr~ are **already freed**!!!!

Need to add copy constructor:
```c++
A(const A& a) { _iptr = new int(*(a._iptr); }
```

## Always Check Asumption Using *Assert*

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
