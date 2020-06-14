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
```c++
  operator T* () const & { return this->get(); } // we smart pointer is a lvalue, get the raw pointer whenever needs
  operator T* () const && = delete;  // when smart pointer is a rvalue, we shouldn't convert to raw pointer, 
                                     // smartPtr itself needs to catch it
```
6. natual use of
    + unique_ptr, tree/list structure wo shared references. 
    + shared_ptr, node based DAG, each node has a vec of shared_ptr other adjacent nodes., tree/list that shared references
7. smartptr usage in the class:
    + has-A, data_member, unique_ptr
    + PIMPL, const unique_ptr (meaning IMPL is constructued as the object is constructed), NOTE the destrcutor is tricky: https://www.fluentcpp.com/2017/09/22/make-pimpl-using-unique_ptr/
    + dynamic array member: unique_ptr<Data[]> array.
    + tree with encapsulation:, but the deletion of subtree is tricky, auto destructor will cann children's dtors recursively, therefore, we need to iteratively collector all subnodes into a vector and then delete them.
    ```c++
      class Tree {
        struct Node {
          vector<unique_ptr<Node>> _children;
          Node* _parent; // parent owns this, so parent must exist
        };
        unique_ptr<Node> root;
      };
    ```
    
    + double linked list, unique_ptr, List is a special Tree, then same as tree.
    ```c++
    class LList {
      struct Node {
        unique_ptr<Node> _next; // prev owns next
        Node* _prev; // when this is deleted, means prev is also deleted, therefore, prev is always valid
      }
      unique_ptr<Node> head;
    };
    ```
    + Tree with strong ref, shared_ptr and shared_ptr aliasing constructor.
    ```c++
    class Tree {
      struct Node {
      vector<shared_ptr<Node>> childrens;
      Node* _parent;
      Data _d;
      };
      shared_ptr<Node> root;
      shared_ptr<Data> find(/**/) { .. return {spn, &(spn->_d)}} // shared_ptr aliasing constructor
    };
    
    ```
    + Factory, unique_ptr or shared_ptr
    + Factory + Cache, shared_ptr + weak_ptr
    ```c++
    shared_ptr<widget> make_widget(int id) {
      static map<int, weak_ptr<widget>> cache;
      static mutex mu;
      lock_guard<mutex> g(mu);
      auto sp = cache[id].lock();
      if (!sp) cache[id] = (sp = load_widget(id));
      return sp;
    }
    ```
    + weak_ptr to register in the callback. Callback shouldn't be the owner. This is to avoid cycle ownership
    ```c++
    void good_fn(const shared_ptr<X>& x) {
      obj.on_draw([w = weak_ptr<X>(x)]{if (auto xx = w.lock()) xx->extra_work(); })
    }
    ```
    
    
