# Memory manager
memory manager can be applied through micro like the following:

```c++
#define MEM_MGR_DECL(T) \
  public: \
    static void* operator new (size_t t) { return _get_mem_m()->alloc(t); } \
    static void* operator new[] (size_t t) { return _get_mem_m()->alloc_array(t); } \
    static void operator delete[] (void *p) { _get_mem_m()->free_array(p); } \
    // in place new \
    static void* operator new(size_t s, void* p) { return in_place_new(s, p); } \
    static void operator delete(void *, void*) { }
    static void operator delete (void *p, size_t t) { _get_mem_m()->free(p, t); } \
    static MemManager* get_mem_mgr() { return _get_mem_m(); } \
  private: \
    static MemManager* new_mem_m(); \
    static MemManager* _get_mem_m() { \
        static MemManager* mem_m = NULL; \
        if (mem_m==NULL) { \
            if (mem_m==NULL) \
                mem_m = new_mem_m(); \
        } \
        return mem_m; \
    }
    
struct RecycleList {
  void* pop() {
    auto ans = _first; _first = (*first); return ans;
  }
  void* _first; // _first is the pointer, it's next is in *_first(this is freed, so we can reused)
  void& _last;
};

struct MemoryBlock {
  void create(size_t t) { _block = malloc(t)}
  void destroy() { free(_block); }
  void* get_elm(size_t index, size_t t) {
    return (void*)(size_t(_begin) + index * t);
  }

  void* _block;
  MemoryBlock* _next;
};

class MemoryManager {
  public:
    void* alloc(size_t t) { either get from recycle_list, or allocate_mem_block; }
    void* alloc_arry(size_t t) { n = t / obj_size; adjust rlist_size and b_size; either get from recycle_list[n], or allocate_mem_block; }
    void free(void* p, size_t t) { _recycle_list[0]->insert(p); }
    void free_array(void* p) {}
    void allocate_mem_block() {
      new_block = new MemoryBlock(_obj_size * _block_size);
      check_if_have_free_in_previous_block; and add to free_list
      adjust _first_block and last_block
    }
  private:
    MemoryBlock* _first_block; // keep track of all allocated blocks, and free them up later
    unsigned _index; // index of the free id in the last_block
    MemoryBlock* _last_block; // when alloc, get memory from _index in the _last_block
    vector<RecycleList*> _recycle_list; // vector of recycle_list, 0 is for new T, 1 is for new T[2], etc
    unsigned _obj_size; // object_size, and with alignof
    unsigned _block_size; // how many objects in one block; must be greater than 2*rlist_size
    unsigned _rlist_size; // recycle list size
   
};
```

