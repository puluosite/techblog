# Common Memory Errors
1. [Check Any Compilation Warnings](#Check-Any-Compilation-Warnings)
2. [Design Copy Constructor When Destructor Free Items](#Design-Copy-Constructor-When-Destructor-Free-Items)
3. [Always Check Asumption Using *Assert](#Always-Check-Asumption-Using-*Assert)
4. [Debug the Stack Overflow that will cause Random Function Call](#stackoverflow)

## <a name="stackoverflow"/>Stack Overflow Causing Function Call Randomly
Check the following code:

```c++
void g()
{
    printf("now inside g()!\n");
}

void f()
{   
    void *x[1];
    printf("now inside f()!\n");
    // can only modify this section
    // cant call g(), maybe use g (pointer to function)
    x[-1]=&g;
}

int main (int argc, char *argv[])
{
    f();
    return 0;
}
```
`g()` is actually triggered, because the ebp+4 is the function return pointer. If stack is overflowed and change that address, program will no longer return to the previous call point. Check this: https://www.tenouk.com/Bufferoverflowc/Bufferoverflow4.html which explains very well. 

In the gdb, we can do: disass func_name to print the assemble code of the function.

So in compiler, always add: **-fstack-protector -fstack-protector-all** 


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
