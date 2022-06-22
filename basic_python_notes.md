## basic python knowledge
1. loop
    1. for item in l
    2. for idx, item in enumerate(l)
    3. for item in zip(l1, l2) # note zip takes iterables and return iterator, therefore it's memory efficient
    4. for key, value in some_map.items()
2. dividen / vs // (floor)
3. compare float number math.isclose(a, b, rel_tol=1e-5)
4. immutable(str,tuple) immutable(list, set, dict(**key in dict is immutable**)) 
5. function's default parameter is only evaluated once, therefore, **DON'T** use mutable as default parameter
6. scope LEGB(local, enclosure(non-local), global(global), builtin(\__builtins__))


## Relearn python

### variables and data model

### useful functions
`globals()` and `locals()` list the map of variables and builtins, etc

function has https://docs.python.org/3/library/inspect.html to check the local variables and other things


