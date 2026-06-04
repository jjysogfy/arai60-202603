105. Construct Binary Tree from Preorder and Inorder Traversal
https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/

# step 1

- 20分ぐらい
  - 問題は前に見て考えていた
  - スマホで書いていたので、本当ならもう少し短かったかも

```java
class Solution {
  public TreeNode buildTree(int[] preorder, int[] inorder) {
    return buildTreeFromLists(
        Arrays.stream(preorder).boxed().toList(),
        Arrays.stream(inorder).boxed().toList());
  }
  
  TreeNode buildTreeFromLists(List<Integer> preorder, List<Integer> inorder) {
    if (preorder.isEmpty()) return null;
    int rootVal = preorder.get(0);
    int numLeftNodes = inorder.indexOf(rootVal);
    TreeNode left = buildTreeFromLists(
        preorder.subList(1, numLeftNodes + 1),
        inorder.subList(0, numLeftNodes));
    TreeNode right = buildTreeFromLists(
        preorder.subList(numLeftNodes + 1, preorder.size()),
        inorder.subList(numLeftNodes + 1, inorder.size()));
    return new TreeNode(rootVal, left, right);
  }
}
```

- 方針
  - preorderの先頭がrootになっている
  - もっといえばこんな感じ：
  - preorder: `root [ left ] [ right ]`
  - inorder: `[ left ] root [ right ]`
  - `inorder.indexOf(root)`の前後で分割すればいい
