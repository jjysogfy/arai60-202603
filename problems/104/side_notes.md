
# step 1

- step 1のコードの再帰を、機械的にループに直す（コールスタックを手で書く感じ）のはちょっと面倒
  - というのは、`maxDepth`の再帰呼び出しが2回あり、呼び出し後の処理もあるから
  - 3値の`enum`みたいなものが要る

```java
// 「step 1 その2」の再帰をループに書き直す
class Solution {
  enum TraverseStatus {
    LEFT,
    RIGHT,
    END
  }

  record NodeAndStatus(TreeNode node, TraverseStatus status, int num) {
  }

  public int maxDepth(TreeNode root) {
    List<NodeAndStatus> stack = new ArrayList<>();
    stack.add(new NodeAndStatus(root, TraverseStatus.LEFT, -1));
    int depth = -1;
    while (!stack.isEmpty()) {
      NodeAndStatus top = stack.removeLast();
      switch (top.status()) {
        case TraverseStatus.LEFT -> {
          if (top.node() == null) {
            depth = 0;
            continue;
          }
          stack.add(new NodeAndStatus(top.node(), TraverseStatus.RIGHT, -1));
          stack.add(new NodeAndStatus(top.node().left, TraverseStatus.LEFT, -1));
        }
        case TraverseStatus.RIGHT -> {
          stack.add(new NodeAndStatus(top.node(), TraverseStatus.END, depth));
          stack.add(new NodeAndStatus(top.node().right, TraverseStatus.LEFT, -1));
        }
        case TraverseStatus.END -> {
          depth = 1 + Math.max(top.num(), depth);
        }
      }
    }
    return depth;
  }
}
```
