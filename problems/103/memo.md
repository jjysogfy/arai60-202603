問題：103. Binary Tree Zigzag Level Order Traversal
https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/


# step 1
```java
// step 1 その1
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

- 前に答えを読んだ
- 16分ぐらい
  - スマホから
- 大まかな方針は決めていたが、書き切るのはわりと悩んでしまった

## step 1.2
「step 1 その1」のコードを書いてから時間が空いてしまったので、改めて見ずに書いてみた。
（コードはside_notes.mdにあります。）


# step 2
Javaでたくさん解いている人のコードを読む
- https://github.com/goto-untrapped/Arai60/pull/51/changes
- https://github.com/ryoooooory/LeetCode/pull/30/changes

気づいたこと
- reverseの書き方
  - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/List.html#reversed()
    - `List.reversed`はJava 21以降の機能。ビューを返す。
      - 感想：特に、返り値はArrayListではない。気づかなかった
  - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/Collections.html#reverse(java.util.List)
    - `Collections.reverse`はリストを破壊する

- **DFSで書いている**人もいる
  - https://github.com/ryoooooory/LeetCode/pull/30/changes#diff-e0b1c5268a02e46b16c0f221400cd45efeaca417cccff8593ddaba80c537d1daR1

- reverseするタイミング
  - 各レベルで、順方向のリストを作った直後にreverseする
    - step 1のやりかた
  - そもそも逆方向のリストだけを（奇数レベルでは）作る
    - https://github.com/goto-untrapped/Arai60/pull/51/changes#diff-d83b4f3c60802130fcbcb68a6a649141d38d591843732126fdf1efbfabbec898R39
  - 全てのレベルを処理したあと、奇数レベルをreverseする
    - https://github.com/goto-untrapped/Arai60/pull/51/changes#diff-96f06b82af40361c4868bc6c29f499bfd6bd8a20144b2492d372436fc0b0adf2R25

- `List.addFirst`を使っている人がいる
  - https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/List.html#addFirst(E)
    - Java 21から。かなり新しめの機能。`List.reversed`と同時
  - Java 21になって追加された経緯は何かが気になって、ChatGPTにきいた
    - https://openjdk.org/jeps/431
    - `addFirst`も`reversed`も`interface SequencedCollection`のメソッド
    - `SequencedCollection`はList, Deque, SortedSet, LinkedHashSetなどを統一するインターフェイス
    - ちゃんと読めていないが。すごく急いで入れるべき機能でもないし、互換性の問題も大きいので、導入が遅くなったのだろうか？


## step 2.2 清書
```java
// step 2.2 清書
class Solution {
  public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    if (root == null) {
      return new ArrayList<>();
    }

    List<List<Integer>> result = new ArrayList<>();
    Deque<TreeNode> frontier = new ArrayDeque<>();
    frontier.add(root);
    int depth = 0;

    while (!frontier.isEmpty()) {
      Deque<TreeNode> nextFrontier = new ArrayDeque<>();
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

      if (depth % 2 != 0) {
        Collections.reverse(values);
      }
      result.add(values);
      frontier = nextFrontier;
      ++depth;
    }

    return result;
  }
}
```


# step 3
1回目（1ミス）：8:11、2回目：5:55、3回目（1ミス）：7:14、
4回目：4:25、5回目（1ミス）：6:41、6回目：5:57、
7回目：3:32、8回目：4:03

- 犯したミス：
  - キューにnullを入れるバージョンで書いたところ、ArrayDequeにnullをaddしてしまいNullPointerException
    - ArrayListはnull許容だが、ArrayDequeは不可
  - Stream APIを使って`values`を作る書き方をしたところ、`Collectors.toList`の使い方を間違えた
    - `.collect(Collectors.toList())`の`()`を忘れた
  - スペルミス
    - DequeをDepthと書いた

コードは清書とほぼ同じ。空行の入り方が違う程度。
```java
// step 3
class Solution {
  public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    if (root == null) {
      return new ArrayList<>();
    }

    List<List<Integer>> result = new ArrayList<>();
    Deque<TreeNode> frontier = new ArrayDeque<>();
    frontier.add(root);
    int depth = 0;
    while (!frontier.isEmpty()) {
      Deque<TreeNode> nextFrontier = new ArrayDeque<>();

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
      if (depth % 2 != 0) {
        Collections.reverse(values);
      }
      result.add(values);

      frontier = nextFrontier;
      ++depth;
    }

    return result;
  }
}
```


# step 4
時間を置いて解き直した。
```java
// step 4 その1
class Solution {
  public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    if (root == null) {
      return new ArrayList<>();
    }

    List<TreeNode> frontier = new ArrayList<>();
    frontier.add(root);
    List<List<Integer>> zigzagTraversal = new ArrayList<>();

    while (!frontier.isEmpty()) {
      List<TreeNode> nextFrontier = new ArrayList<>();

      List<Integer> values = frontier.stream()
          .map(node -> node.val)
          .collect(Collectors.toCollection(ArrayList::new));
      if (zigzagTraversal.size() % 2 != 0) {
        Collections.reverse(values);
      }

      zigzagTraversal.add(values);

      for (TreeNode node : frontier) {
        if (node.left != null) {
          nextFrontier.add(node.left);
        }
        if (node.right != null) {
          nextFrontier.add(node.right);
        }
      }

      frontier = nextFrontier;
    }

    return zigzagTraversal;
  }
}
```

- かかった時間：7:58
