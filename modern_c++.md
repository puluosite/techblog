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
