# ファイル構成
このファイルより先に、`memo.md`をご覧ください。

（脇にそれた話題・コードがこちらのファイルにあります。自分の復習や、他の方のレビュー・学習に役立つ場合もあるかもしれないと思い、このファイルも残してあります。）


# step 1 への補足
「step 1 その1」のコードを書いてから時間が空いてしまったので、改めて、見ずに解いてみた。
```java
// step 1 その2
class Solution {
  public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
    List<List<Integer>> zigzag = new ArrayList<>();
    List<TreeNode> nodes = new ArrayList<>();
    nodes.add(root);
    int depth = 0;

    while (!nodes.isEmpty()) {
      List<TreeNode> nextNodes = new ArrayList<>();
      for (TreeNode node : nodes) {
        if (node.left != null) {
          nextNodes.add(node.left);
        }
        if (node.right != null) {
          nextNodes.add(node.right);
        }
      }

      List<TreeNode> orderedNodes = depth % 2 == 0 ? nodes : nodes.reversed();
      List<Integer> values = orderedNodes.stream().map(node -> node.val).toList();
      zigzag.add(values);
      ++depth;
      nodes = nextNodes;
    }
    return zigzag;
  }
}
```

- 9分
- 方針の違い：
  - 前のバージョンでは、`values`をfor文で作った
  - 今回のバージョンでは、`values`をstreamで作った
    - `nodes`（前のバージョンでは`frontier`）が、ほとんどそのまま返り値の一部だ、という気分があった。そのせい？
    - 途中でうっかり`nodes`を書き換えないか、ちょっと怖かったかも
- ArrayDequeで書きはじめ、途中からArrayListに直した
  - メソッド名が一部違うことに注意すべきだった。今回はaddしかしてないからたまたま問題なかったが

