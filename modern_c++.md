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
5. move ctor (T&&) nonexcept  ---> transfer resource
6. move assignment (T& operator=(T&&)) ---> transfer resource
7. friend void swap(T& a, T& b) that calls member function swap(T& other) nonexcept
8. to ask compiler generate: using keyword = default; to disable, using keyword = delete
