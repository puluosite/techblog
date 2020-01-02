# Techniques To Make Debugging Easier
1. [Templates](#temp)
2. [GDB functions](#gdb_function)

## <a name="temp"/>C++ Templates
1. Instantiate template class methods that are not used in the code, *all the following code are written in CXX files*.

    1. all members of the template class
    ```c++
      template class TempClass<P1, P2, P3>;
      template class std::vector<P1>;
    ```
  
    2. specific members
    ```c++
    template void TempClass<P1, P2, P3>::some_member(const P1& p1);
    ```
  
    3. Usually I will instantiate all members in the debug build, so that all methods can be called in GDB
  
## <a name="gdb_function"/>GDB functions
We can write GDB debugging utils in the `.gdbinit`. For example,
```make
define find_obj_in_vec 
    set $path = $arg0
    set $target = $arg1
    set $size = $path._M_impl._M_finish - $path._M_impl._M_start
    #print $size
    set $i = 0
    print "BEGINNING OF VEC"
    while $i < $size
        #print name
        set $tp = ($path._M_impl._M_start + $i)
        if (*$tp == $target)
            print "found obj at"
            print $i
            loop_break
        end
        set $i++
    end
    print "END OF VEC"
end
```
GDB has its own set of programming commands, please check:
https://sourceware.org/gdb/onlinedocs/gdb/Command-Files.html
