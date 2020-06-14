## rvalue and lvalue
1. rvalue is a temporary value that doesn't have a name, while lvalue is the variable that has name
2. T&& to represent a rvalue reference
3. std::move convert lvalue/rvalue to a rvalue reference **unconditionally**, used when there is **no template**
4. std::farward convert rvalue to rvalue reference, and lvalue to lvalue reference, 
   **conditionally**, used in **template/perfect forwarding**

```c++
template<typename ARG>
void foo(ARG&& val) { make_unique<Y>(new Y(std::forward<ARG>(val))); }
```

5. type deduction `ARG &&`, when `ARG` is `T&`, `& &&` is `&`, only when `&& &&` is deduced to `&&`
6. Don't return `T&&`, it's only return the local reference. Crash! 

## what to define in class
1. ctor   ---> alloc resource
2. dtor   ---> delete resource
3. copy ctor (const T&)  ---> copy resource
4. copy assignment (T& operator=(const T&)) ---> copy resource 
5. move ctor (T&&) **noexcept**  ---> transfer resource, make things working, need noexcept!!
6. move assignment (T& operator=(T&&)) ---> transfer resource
7. friend void swap(T& a, T& b) that calls member function swap(T& other) **noexcept**, make things working, need noexcept!!
8. to ask compiler generate: using keyword = default; to disable, using keyword = delete

## smart pointer
1. make_unique/make_share over new
2. unique_ptr can take a DELETER to do resource management on other object
```
struct FileCloser { void operator()(FILE* fp) {fclose(fp); } };
FILE* fp = fopen("foo.txt", "r");
unique_ptr<FILE, FileCloser> uptr(fp);
```
3. rule of thumb of smart pointers
    + pass by value (**meaning ownership transfer**)
    + return by value
    + pass raw pointer/small pointer usually code smell/infer bug
4. use smart pointer related to ownership, otherwise, use raw pointers. i.e. when use raw pointer, no new/delete
5. In the smart pointer class, we can have the following 2 member functions:
    (https://www.youtube.com/watch?v=LYeFGiYaOaE)
  operator T* () const & { return this->get(); } // we smart pointer is a lvalue, get the raw pointer whenever needs
  operator T* () const && = delete;  // when smart pointer is a rvalue, we shouldn't convert to raw pointer, smartPtr itself needs to catch it
