## Binary search
Given a sorted arrary [0,r), find the smallest `m` such that `g(m) == true`. Note that `g(m)` doesn't need to be increamental.
Only need to satify:
1. for x >= m, g(m) >=0, i.e true
2. for x <=, g(m) < 0, i.e. false

If there is no solution, then `return_value = r`

Then code is:

```cpp
bool greater_or_equal_than(int i1, int i2)
{
  return i1 >= i2;
}

size_t my_binary_search(const vector<int>& vec, int elem)
{
    size_t l = 0;
    size_t r = vec.size();
    
    while (l < r) {
        size_t m = l + (r-l)/2; // shouldn't write (l+r)/2 because l+r can overflow!!!!
        if (greater_or_equal_than(vec[m], elem)) // g(m) is true
            r = m;
        else
            l = m + 1;
    }
    return l;
}
```
