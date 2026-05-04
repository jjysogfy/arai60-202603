
# step 1

- step 1のコードの再帰を、機械的にループに直す（コールスタックを手で書く感じ）のはちょっと面倒
  - というのは、`maxDepth`の再帰呼び出しが2回あり、呼び出し後の処理もあるから
  - 3値の`enum`みたいなものが要る（コールスタックに積むアドレスに相当）

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


# step 2

- 再帰を別の書き方でループに直す
  - フラグを使わなくていい
  - https://discord.com/channels/1084280443945353267/1227073733844406343/1236324993839792149
- intへの参照に相当するものが必要
  - 今回は`List<Integer>`を使ってみた
  - 前にコメント集で`AtomicInteger`というものを使っている人がいた気がする

```java
class Solution {
  // `List<Integer>` is used as a pointer to an `int` value
  record StackFrame(TreeNode node, List<Integer> returnValue,
      List<Integer> leftDepth, List<Integer> rightDepth) {
    static StackFrame of(TreeNode node, List<Integer> returnValue) {
      return new StackFrame(node, returnValue,
        new ArrayList<>(), new ArrayList<>());
    }

    static StackFrame of(TreeNode node) {
      return of(node, new ArrayList<>());
    }
  }

  public int maxDepth(TreeNode root) {
    List<StackFrame> stack =  new ArrayList<>();
    StackFrame rootStackFrame = StackFrame.of(root);
    stack.add(rootStackFrame);
    while (!stack.isEmpty()) {
      StackFrame top = stack.removeLast();
      if (top.node() == null) {
        top.returnValue().add(0);
        continue;
      }
      if (top.leftDepth().isEmpty()) {
        stack.add(top);
        stack.add(StackFrame.of(top.node().left, top.leftDepth()));
        continue;
      }
      if (top.rightDepth().isEmpty()) {
        stack.add(top);
        stack.add(StackFrame.of(top.node().right, top.rightDepth()));
        continue;
      }
      int leftDepth = top.leftDepth().getFirst();
      int rightDepth = top.rightDepth().getFirst();
      top.returnValue().add(1 + Math.max(leftDepth, rightDepth));
    }
    return rootStackFrame.returnValue().getFirst();
  }
}
```
