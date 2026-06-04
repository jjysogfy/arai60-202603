103. Binary Tree Zigzag Level Order Traversal
https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/

# step 1
- 16分ぐらい
  - スマホから
- 大まかな方針は決めていたが、書き切るのはわりと悩んでしまった

```java
class Solution {
  public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    if (root == null) return List.of();
    List<List<Integer>> result = new ArrayList<>();
    
    List<TreeNode> frontier = new ArrayList<>();
    frontier.add(root);
    int depth = 0;
    while (!frontier.isEmpty()) {
      List<TreeNode> nextFrontier = new ArrayList<>();
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
      if (depth % 2 == 0) {
        result.add(values);
      } else {
        result.add(values.reversed());
      }
      frontier = nextFrontier;
      ++depth;
    }
    
    return result;
  }
}
```
