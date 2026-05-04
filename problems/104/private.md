問題（104. Maximum Depth of Binary Tree）：https://leetcode.com/problems/maximum-depth-of-binary-tree/

# step 1
- かかった時間：12:34
- 発想：
  - DFSすればいい
  - スタックを使いループで書く
- 迷ったところ：
  - 深く考えず`List<TreeNode> nodes`と書き、whileの中身を書きはじめて迷った

```java
// step 1
class Solution {
  record NodeAndNum(TreeNode node, int num) {}

  public int maxDepth(TreeNode root) {
    if (root == null) {
      return 0;
    }
    List<NodeAndNum> nodeAndDepths = new ArrayList<>();
    nodeAndDepths.add(new NodeAndNum(root, 1));
    int result = 0;
    while (!nodeAndDepths.isEmpty()) {
      NodeAndNum nodeAndDepth = nodeAndDepths.removeLast();
      TreeNode node = nodeAndDepth.node();
      int depth = nodeAndDepth.num();
      result = Math.max(result, depth);
      if (node.left != null) {
        nodeAndDepths.add(new NodeAndNum(node.left, depth + 1));
      }
      if (node.right != null) {
        nodeAndDepths.add(new NodeAndNum(node.right, depth + 1));
      }
    }
    return result;
  }
}
```

- 再帰で書くほうが簡単だと思い、書いてみる
  - 実際こちらのが簡単だった
  - これをループに直すのはちょっと面倒
    - `side_notes.md`に書いておく

```java
// step 1 その2 再帰
class Solution {
  public int maxDepth(TreeNode root) {
    if (root == null) {
      return 0;
    }
    int leftDepth = maxDepth(root.left);
    int rightDepth = maxDepth(root.right);
    return 1 + Math.max(leftDepth, rightDepth);
  }
}
```


# step 2
コメント集を見る。
- https://discord.com/channels/1084280443945353267/1233603535862628432/1278690571132604460
  - JavaのQueue vs Deque vs ArrayDeque
  - add/remove、offer/pollはQueueのメソッド
    - 実装を見る限り、このどちらを使ってもArrayDequeでは同じ
      - https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/ArrayDeque.java
    - 要素数が制限されたキュー（そういうものがあるらしい）では挙動が違う。例外か、特別な値が返るか
      - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/Queue.html
  - DequeはQueueのサブ・インターフェイス。addFirst/addLastなどを持つ
    - スタックとして使うために、push/popも持つ
- https://discord.com/channels/1084280443945353267/1227073733844406343/1236235351140339742
  - 再帰 vs スタックを使うループ

