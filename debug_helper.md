# Techniques To Make Debugging Easier
1. [Templates](#temp)
2. [GDB functions](#gdb_function)
3. [GDB address](#gdb_address)
4. [GDB tricks](#gdb_tricks)

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

## <a name="gdb_address"/>GDB address
https://sourceware.org/gdb/onlinedocs/gdb/Symbols.html
1. We can print the function address in GDB by
    ```makefile
      info address func_foo
    ```
2. To print function or line number from address, we can do:
    ```makefile
    info symbol 0x871657a
    list *0x871657a
    ```
## <a name="gdb_tricks"/>GDB tricks
### rbreak can break functions by regex, e.g. ^package_name_get.*token&
### d 10-50, delete breakpoints from 10 to 50
### x/x ($rbp) print the address of register
