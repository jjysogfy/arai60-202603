
# ファイル構成
このファイルより先に、`memo.md`をご覧ください。


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
  - `List<Integer>`を使うコード、`AtomicInteger`を使うコードの2つを書いた
    - `AtomicInteger`はコメント集でだけ見たことがあった。はじめて使った

```java
// `List<Integer>`を参照として使う
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

```java
// `AtomicInteger`を参照として使う
// nullの代わりに`UNDETERMINED = -1`を使う
class Solution {
  static final int UNDETERMINED = -1;

  record StackFrame(TreeNode node, AtomicInteger returnValue,
      AtomicInteger leftDepth, AtomicInteger rightDepth) {
    static StackFrame of(TreeNode node, int returnValue) {
      return new StackFrame(node, new AtomicInteger(returnValue),
        new AtomicInteger(UNDETERMINED), new AtomicInteger(UNDETERMINED));
    }

    static StackFrame of(TreeNode node) {
      return of(node, UNDETERMINED);
    }
  }

  public int maxDepth(TreeNode root) {
    List<StackFrame> stack =  new ArrayList<>();
    StackFrame rootStackFrame = StackFrame.of(root);
    stack.add(rootStackFrame);
    while (!stack.isEmpty()) {
      StackFrame top = stack.removeLast();
      if (top.node() == null) {
        top.returnValue().set(0);
        continue;
      }
      if (top.leftDepth().get() == UNDETERMINED) {
        stack.add(top);
        stack.add(StackFrame.of(top.node().left, top.leftDepth().get()));
        continue;
      }
      if (top.rightDepth().get() == UNDETERMINED) {
        stack.add(top);
        stack.add(StackFrame.of(top.node().right, top.rightDepth().get()));
        continue;
      }
      int leftDepth = top.leftDepth().get();
      int rightDepth = top.rightDepth().get();
      top.returnValue().set(1 + Math.max(leftDepth, rightDepth));
    }
    return rootStackFrame.returnValue().get();
  }
}
```

