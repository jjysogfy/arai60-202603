98. Validate Binary Search Tree
https://leetcode.com/problems/validate-binary-search-tree/

# step 1
- （書くのにかかった時間は覚えていない）
- DFSでできるだろう、と考えた

```java
class Solution {
  public boolean isValidBST(TreeNode root) {
    return isValidBstContainedIn(root, Integer.MIN_VALUE, Integer.MAX_VALUE);
  }

  private boolean isValidBstContainedIn(TreeNode node, long left, long right) {
    if (node == null) {
      return true;
    }
    if (!(left <= node.val && node.val <= right)) {
      return false;
    }
    return isValidBstContainedIn(node.left, left, (long) node.val - 1)
        && isValidBstContainedIn(node.right, (long) node.val + 1, right);
  }
}
```

- オーバーフローの処理について
  - 制約を見て、意識しなければいけないことは念頭にあった
  - `isValidBstContainedIn(node, left, right)`のleft, rightがinclusiveなのは、その意識から
  - `node.val - 1`してるので、この方法ではintでなくlongにしておく必要がある
    - こちらは気づかず、1ミス
  - longなんか使わず、もっと良い方法がありそうとは思った

