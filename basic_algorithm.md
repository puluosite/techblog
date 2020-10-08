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
