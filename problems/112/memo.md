問題：112. Path Sum https://leetcode.com/problems/path-sum/

# step 1
```java
class Solution {
  public boolean hasPathSum(TreeNode root, int targetSum) {
    if (root == null) {
      return false;
    }
    if (root.left == null && root.right == null) {
      return root.val == targetSum;
    }
    return hasPathSum(root.left, targetSum - root.val)
        || hasPathSum(root.right, targetSum - root.val);
  }
}
```

- かかった時間：3:39


# step 2
Javaでたくさん解いている人のコードを見ておく。
- https://github.com/ryoooooory/LeetCode/pull/28/changes

見て思ったこと：
- L1キャッシュの話が詳しく書いてある
  - internet archiveで講義資料もざっと見てみた。あんまり理解しなかった


ループになおしておく
```java
// step 2 その1
class Solution {
  private record NodeWithNum(TreeNode node, int num) {
  }

  public boolean hasPathSum(TreeNode root, int targetSum) {
    if (root == null) {
      return false;
    }

    Deque<NodeWithNum> nodesWithNums = new ArrayDeque<>();
    nodesWithNums.push(new NodeWithNum(root, root.val));
    while (!nodesWithNums.isEmpty()) {
      NodeWithNum top = nodesWithNums.pop();
      TreeNode node = top.node;
      int total = top.num;
      if (node.left == null && node.right == null && total == targetSum) {
        return true;
      }
      if (node.left != null) {
        nodesWithNums.push(new NodeWithNum(node.left, total + node.left.val));
      }
      if (node.right != null) {
        nodesWithNums.push(new NodeWithNum(node.right, total + node.right.val));
      }
    }
    return false;
  }
}
```

- かかった時間：6:48
- ミスあり
  - `root == null`忘れ
  - `nodesWithNums.push(new NodeWithNum(root, 0));`（`root.val`にすべき）

スタックにnullを積むことを許してみるとどうなるだろう。書いてみる。

```java
// step 2 その2
class Solution {
  private record NodeAndSum(TreeNode node, int sum) {
  }

  public boolean hasPathSum(TreeNode root, int targetSum) {
    Deque<NodeAndSum> stack = new ArrayDeque<>();
    stack.push(new NodeAndSum(root, 0));
    while (!stack.isEmpty()) {
      NodeAndSum nodeAndSum = stack.pop();
      TreeNode node = nodeAndSum.node;
      if (node == null) {
        continue;
      }
      int total = nodeAndSum.sum + node.val;
      if (node.left == null && node.right == null && total == targetSum) {
        return true;
      }
      stack.push(new NodeAndSum(node.left, total));
      stack.push(new NodeAndSum(node.right, total));
    }
    return false;
  }
}
```

- かかった時間：4:14
- 前の書き方のほうが素直だと思った


# step 3
1回目：4:10、2回目：5:44、3回目：3:31

```java
class Solution {
  private record NodeAndSum(TreeNode node, int sum) {
  }

  public boolean hasPathSum(TreeNode root, int targetSum) {
    if (root == null) {
      return false;
    }
    Deque<NodeAndSum> stack = new ArrayDeque<>();
    stack.push(new NodeAndSum(root, root.val));
    while (!stack.isEmpty()) {
      NodeAndSum nodeAndSum = stack.pop();
      TreeNode node = nodeAndSum.node;
      int total = nodeAndSum.sum;
      if (node.left == null && node.right == null && total == targetSum) {
        return true;
      }
      if (node.left != null) {
        stack.push(new NodeAndSum(node.left, total + node.left.val));
      }
      if (node.right != null) {
        stack.push(new NodeAndSum(node.right, total + node.right.val));
      }
    }
    return false;
  }
}
```
