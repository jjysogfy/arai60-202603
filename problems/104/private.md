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

- これをループに直すのはちょっと面倒
  - コードは`side_notes.md`に書いておく


# step 2
コメント集を見る。
- https://discord.com/channels/1084280443945353267/1233603535862628432/1278690571132604460
  - https://github.com/goto-untrapped/Arai60/pull/45/changes
  - JavaのQueue vs Deque vs ArrayDeque (vs LinkedList)
  - add/remove、offer/pollはQueueのメソッド
    - 実装を見る限り、このどちらを使ってもArrayDequeでは同じ
      - https://github.com/openjdk/jdk/blob/master/src/java.base/share/classes/java/util/ArrayDeque.java
    - 要素数が制限されたキュー（そういうものがあるらしい）では挙動が違う。例外か、特別な値が返るか
      - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/Queue.html
  - DequeはQueueのサブ・インターフェイス。addFirst/addLastなどを持つ
    - スタックとして使うために、push/popも持つ
  - 「ArrayDequeはnullを処理せず、例外を投げる。LinkedListはnullも格納できる。」とのこと
- https://discord.com/channels/1084280443945353267/1227073733844406343/1236235351140339742
  - 再帰 vs スタックを使うループ
  - C++だと参照型を使ってもっと見やすく書ける
    - https://discord.com/channels/1084280443945353267/1227073733844406343/1236324993839792149
    - 再帰関数でいえば、引数に参照を渡して返り値の代わりにした感じ
    - フラグも使わずに済む。どの再帰呼び出しの返り値がnon nullか、を使う感じ
    - コードは`side_notes.md`に書いておく
- https://discord.com/channels/1084280443945353267/1227073733844406343/1236235351140339742
  - step 1のスタックとループのコード。再帰でも書けて、そのほうがわかりやすく感じる
- 再帰はいくつか欠点があるとコメント集で見た気がして、若干避けている
  - スタックサイズの制限はある
  - 他はあまり覚えてない、というか理解していない
  - https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.deivkzaqvetb
    - https://discord.com/channels/1084280443945353267/1350090869390311494/1355224179720458261
    - スタックトレースが役立たなくなる、という感じ？

## 清書
- step 1と同じ方針で清書する

```java
// step 2 清書
class Solution {
  record NodeAndDepth(TreeNode node, int depth) {
  }

  public int maxDepth(TreeNode root) {
    Deque<NodeAndDepth> stack = new ArrayDeque<>();
    stack.push(new NodeAndDepth(root, 1));
    int result = 0;
    while (!stack.isEmpty()) {
      NodeAndDepth top = stack.pop();
      TreeNode node = top.node();
      int depth = top.depth();
      if (node == null) {
        continue;
      }
      result = Math.max(result, depth);
      stack.push(new NodeAndDepth(node.left, depth + 1));
      stack.push(new NodeAndDepth(node.right, depth + 1));
    }
    return result;
  }
}
```

- スタックとしてArrayListとArrayDequeのどちらを使うのが良いか？ よくわかってない
  - 変数名`stack`は指摘を受けそうだが、今回はわかりづらくならない気がする（反論はあればありがたいです）

