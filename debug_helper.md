# Techniques To Make Debugging Easier
1. [Templates](#temp)

## <a name="temp"/>C++ Templates
1. Instantiate template class methods that are not used in the code, *all the following code are written in CXX files*.

    1. all members of the template class
    ```c++
      template class TempClass<P1, P2, P3>;
    ```
  
    2. specific members
    ```c++
    template void TempClass<P1, P2, P3>::some_member(const P1& p1);
    ```
  
    3. Usually I will instantiate all members in the debug build, so that all methods can be called in GDB
  
  
