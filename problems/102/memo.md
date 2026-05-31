102. Binary Tree Level Order Traversal https://leetcode.com/problems/binary-tree-level-order-traversal/

# step 1
5:44（1ミス、`root == null`チェック忘れ）

```java
class Solution {
  public List<List<Integer>> levelOrder(TreeNode root) {
    if (root == null) {
      return new ArrayList<>();
    }
    List<List<Integer>> result = new ArrayList<>();
    Queue<TreeNode> frontier = new ArrayDeque<>();
    frontier.add(root);
    while (!frontier.isEmpty()) {
      Queue<TreeNode> nextFrontier = new ArrayDeque<>();
      List<Integer> values = new ArrayList<>();
      for (TreeNode node : frontier) {
        values.add(node.val);
        if (node.left != null) {
          nextFrontier.add(node.left);
        }
        if (node.right != null) {
          nextFrontier.add(node.right);
        }
      }
      frontier = nextFrontier;
      result.add(values);
    }
    return result;
  }
}
```
