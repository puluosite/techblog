# Common Memory Errors
1. [Check Any Compilation Warnings](#Check-Any-Compilation-Warnings)
2. [Design Copy Constructor When Destructor Free Items](#Design-Copy-Constructor-When-Destructor-Free-Items)
3. [Always Check Asumption Using *Assert](#Always-Check-Asumption-Using-*Assert)
4. [Debug the Stack Overflow that will cause Random Function Call](#stackoverflow)
5. [Compiler Flags](#compiler-flags)
6. [Smart Pointers](#smart-pointers)
7. [Danger of temp objects](#tempobj)

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

So if we have `unique_ptr` then there is no issue.

Need to add copy constructor:
```c++
A(const A& a) { _iptr = new int(*(a._iptr); }
```

Another interesting thing is that, resize will prefer copy ctor over move ctor if move ctor is not declared as **noexcept**. (https://stackoverflow.com/questions/18085383/c11-rvalue-reference-calling-copy-constructor-too) So even if you have:
```c++
A(A&& a) { _iptr = a._iptr; a._iptr = NULL; }
```
It will still do memory copy. You need to have:
```c++
A(A&& a) noexcept { _iptr = a._iptr; a._iptr = NULL; }
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
## Compiler Flags

Do you see problem in the following code? 
```c++
class D : B {
  public:
    D() : _a(0) {}
    ~D() {}
    virtual void print() {...} // class B has a virtual function call print
  private:
    int _a;
    int _b;
}
```
Two issues: 1. `_b` is not ininitialized, and there is virtual function but destructor is non-virtual. In GCC, even you have `-Wall`, it doesn't show any warning. Have to have the following:
```make
CFLAGS=-c -Wall -Wextra -Weffc++ -g -std=c++1y
INCLUDEDIRS = -isystem$(BOOST_INC)
```

`Weffc++` will show the above 2 warnings. The `isystem` will suppress warning from the appended libs. 

## Smart Pointers
```c++
unique_ptr<A> make_a() { return make_unique<A>(xxx); }
const A& a = *make_a(); // BOOM, temp uniq_ptr is out of scope, and ref to unknown now
```
same issue for shared_ptr, never use a non-smart-ptr to catch the return value. ASAN can catch it. `-fsanitizer-address-use-after-delete`

## <a name="tempobj"/>Danger of temp object
```c++
template<T>
const T& min(const T& a, const T& b) { return a < b ? a : b; }
int x - 10, y = 20;
int a = min(10, 20); // ok
const int& aa = min(10, 20); // ok
const int& aaa = min(x+1,y); // BOOM, x+1 is temp object, and we are ref to a temp object out of scope
```
another exmaple for map when we pass a default value as function parameter then pass reference back.

**always copy a local shared_ptr when use get() to pass down the call tree when shared_ptr  callee**
```c++
void f(int* p) {xxx}
shared_ptr<int> gsp = make_shared<int>(1);
int main() {
  f(gsp.get()); // bad, in call tree of f, they can change gsp, because they can change gsp, and gsp.get()'s pointer is invalid
  auto sp = gsp;
  f(sp.get()); // good, sp add ref cnt by 1, so gsp cannot be invalidated. i.e. f cannot change sp.
}
```
