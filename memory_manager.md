# Memory manager
memory manager can be applied through micro like the following

c++```
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
    
class MemoryManager {
  private:
    MemoryBlock* _first_block;
    MemoryBlock* _last_block;
    RecycleList* _recycle_list;
};
```

