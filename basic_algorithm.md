## Fenwick Tree

Handle case when want to query the range sum.
`[1,2,3,4,5]`
How to know the range sum from 1->3, we can calculate the prefix sum `[1,3,6,10,15]`, then `1->3` is `6-1 = 5`.
However, when the array is dynamically changed, we need to update the prefix sum. To update one element, it's `O(n)`

Fenwick Tree is a data structure that does update in `O(lgN)`, and query prefixSum in `O(lgN)`

```c++
class FenwickTree
{
FenwickTree(int n): _sums(n+1, 0) {}
vector<int> _sums;
void update(int i, int val) {
  while(i < _sums.size()) {
    _sums[i] += val;
    i += lower_bit(i); // find parent in update tree
  }
}

int get_partial_sum(int i) {
  int sum = 0;
  while (i != 0) {
    sum += _sums[i];
    i -= lower_bit(i); // parent in query tree
  }
  return sum;
}

int lower_bit(int i) { return i&(-i);}

};
```
## Binary search
Given a sorted arrary [0,r), find the smallest `m` such that `g(m) == true`. Note that `g(m)` doesn't need to be increamental.
Only need to satify:
1. for x >= m, g(m) >=0, i.e true
2. for x <=, g(m) < 0, i.e. false

If there is no solution, then `return_value == arr.size()`

always remember `l = 0, r = vec.size()`左闭右开

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

## Tree inorder/preorder/postorder

```c++
vector<int> inorderTraversal(TreeNode* root) {
if (root == nullptr) return {};
vector<int> ans;
stack<TreeNode*> s;
TreeNode* curr = root;
while (curr != nullptr || !s.empty()) {
  while (curr != nullptr) {
    s.push(curr);
    curr = curr->left;
  }
  curr = s.top(); s.pop();
  ans.push_back(curr->val);
  curr = curr->right;
}    
return ans;
}
```

```c++
  vector<int> preorderTraversal(TreeNode* root) {
  vector<int> ans;
  stack<TreeNode*> s;
  if (root) s.push(root);
  while (!s.empty()) {
    TreeNode* n = s.top();
    ans.push_back(n->val);
    s.pop();
    if (n->right) s.push(n->right);
    if (n->left) s.push(n->left);            
  }
  return ans;
```
  
post visit is to simulate rev_post_visit
print node; visit_right_child; visit_left_child;
then reverse the answer;
So solution is very similar to preorder visit
```c++
vector<int> postorderTraversal(TreeNode* root) {
    if (root == nullptr) return {};

    deque<int> ans;
    stack<TreeNode*> s;
    s.push(root);
    while (!s.empty()) {
        auto* curr = s.top();
        s.pop();
        ans.push_front(curr->val);
        if (curr->left) s.push(curr->left);
        if (curr->right) s.push(curr->right);

    }

    vector<int> a(ans.begin(), ans.end());
    return a;

}
```

## Sliding Window

handling of the substring can use the following template
```c++
vector<int> need(128,0), map(128, 0);
// init need
int counter = 0;
for (auto c : sub_str) {
  need[c]++; // record 
}
// iterate string
int l = 0, r = 0;
for (; r < str.size(); ++r) {

  // increase window
  ++map[str[r]];
  if (map[str[r]] == need[str[r]]) // record, usually ++counter
  
  // window fit with any length
  // if (/* counter condition(/) 
  
  // shring window
  while (/* window condition */) {
    if (map[str[l]] == need[str[l]]) // pop, usually --counter
    ++l;
  }
  // possible window fit with min size
}

```

## monotonic queue
use to find min/max in sliding window with constant time

```c++
class MonoDecQueue {
      public:
        void push(int v) {
            while (!_q.empty() && v > _q.back()) 
                _q.pop_back();  
            _q.push_back(v);
        }
        
        int get_max() const {return _q.front(); }
        void pop(int v) {
            if (v == _q.front())
                _q.pop_front();
        }
        deque<int> _q;
    };
```


## combination/permutation/subset/backtrack

```c++
// Combination C(m, n)
void backtrace(int start, vector<int>& path, int m, const vector<int>& n)
{
  // termination
  if (path.size() == m) {
    res.push_back(path);
    return;
  }
  
  for (int i = start; i < n.size(); ++i) {
    // select
    path.push_back(n[i]);
    backtrack(i+1, path, m, n);
    // unselect
    path.pop_back();
  }
}
```
```c++
// Permutation P(m, n)
void backtrace(vector<int>& path, int m, const vector<int>& n)
{
  // termination
  if (path.size() == m) {
    res.push_back(path);
    return;
  }
  
  for (int i = 0; i < n.size(); ++i) {
    if (used[i]) continue;
    // select
    used[i] = true;
    path.push_back(n[i]);
    backtrack(path, m, n);
    // unselect
    path.pop_back();
    used[i] = false;
  }
}
```
subset can be treate as:
+ combination of selecting 0, 1, 2, ... n-1, n
```C++
for (int i = 0; i <= n.size(); ++i) {
  comb_backtrack(0, path, i, n);
}
```
+ select/not-select items
```c++
void backtrack(int start, vector<int>& path, const vector<int>& nums)
{
  if (start == nums.size()) {
      res.push_back(path);
      return;
  }
  // not select
  backtrack(start+1, path, nums);
  // select
  path.push_back(nums[start]);
  backtrack(start+1, path, nums);
  path.pop_back();
}
```

```c++
bool backtrack(start, path, )
{
  if (/* condition met*/) {
    record_path(path);
    return true;
  }
  
  for (auto choice : choices) {
    if(/*current_choice_invalid*/)
      continue/break; // depends
    path.push_back(choice);
    if (backtrack(start+1, path, ...))
      // we can terminate early, return true
    path.pop_back();
  }
  return false // to terminate early
}
```


    
