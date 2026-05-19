問題（617. Merge Two Binary Trees）：https://leetcode.com/problems/merge-two-binary-trees/

```java
import java.util.Optional;

class Solution {
  public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
    if (root1 == null && root2 == null) {
      return null;
    }

    int value = 0;
    Optional<TreeNode> maybeRoot1 = Optional.ofNullable(root1);
    Optional<TreeNode> maybeRoot2 = Optional.ofNullable(root2);
    value += maybeRoot1.map(t -> t.val).orElse(0);
    value += maybeRoot2.map(t -> t.val).orElse(0);
    var leftNode = mergeTrees(maybeRoot1.map(t -> t.left).orElse(null),
        maybeRoot2.map(t -> t.left).orElse(null));
    var rightNode = mergeTrees(maybeRoot1.map(t -> t.right).orElse(null),
        maybeRoot2.map(t -> t.right).orElse(null));
    return new TreeNode(value, leftNode, rightNode);
  }
}
```
