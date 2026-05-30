# ファイル構成
このファイルより先に、`memo.md`をご覧ください。

（脇にそれた話題・コードがこちらのファイルにあります。自分の復習や、他の方のレビュー・学習に役立つ場合もあるかもしれないと思い、このファイルも残してあります。）


# step 2
## JVMのスタックについて
- JVMのメモリの構造について、AIにきいてみよう
- きいた。自分なりのまとめ：
  - スタックマシンとは、算術演算が基本的にスタックトップだけを対象とするマシンのこと
  - JVMはスタックマシンで、そのスタックはオペランド・スタックとよばれる
  - メソッドが呼び出されるとフレームが作成されるが、その中にオペランド・スタックも置かれる
    - 各メソッドのオペランド・スタックの最大長はコンパイル時に決定済
  - メソッド呼び出しのフレームは、JVMスタックというものに積まれる
    - ローカル変数はスタックフレームの中にしまわれる
    - JVMスタックとは別に、ヒープ領域もちゃんとある
  - 実際の実行時には、バイトコードをインタプリタで実行するか、JITで機械語にコンパイルし実行する
    - JITでは、オペランド・スタックはレジスタに割り当てられることが多い
- ひとくちにスタックと言っても、ここでは、オペランド・スタック、JVMスタック、物理的なマシンのスタック、の3つがある
  - 振り返ると、そのせいでちょっと疑問に思ったということかも


## 上から作る方法
「上から配る + 再帰」を書いてみる
- これはJavaだとちょっと書きづらいのだった
  - TreeNodeではなくintを返すときも、AtomicIntegerを利用（乱用？）した
- TreeNodeの場合、`TreeNode.left`への参照を取れないのでさらに書きづらい
- ラムダを使う以下の方法を思いついた
  - けっこう変な書き方のような気もする？
```java
// step 2 その2 「上から配る + 再帰」の変な（？）書き方
class Solution {
  class TreeNodeRef implements Consumer<TreeNode> {
    TreeNode node;

    @Override
    public void accept(TreeNode node) {
      this.node = node;
    }
  }

  public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
    var mergedRef = new TreeNodeRef();
    mergeTreesHelper(root1, root2, mergedRef);
    return mergedRef.node;
  }

  private void mergeTreesHelper(TreeNode root1, TreeNode root2, Consumer<TreeNode> resultWriter) {
    if (root1 == null && root2 == null) {
      resultWriter.accept(null);
      return;
    }

    TreeNode node1 = root1 != null ? root1 : new TreeNode(0);
    TreeNode node2 = root2 != null ? root2 : new TreeNode(0);
    TreeNode merged = new TreeNode(node1.val + node2.val);
    resultWriter.accept(merged);
    mergeTreesHelper(node1.left, node2.left, node -> { merged.left = node; });
    mergeTreesHelper(node1.right, node2.right, node -> { merged.right = node; });
  }
}
// step 2 その2終わり
```

- 思いついた過程：
  - しばらく（30分以上？）かかった
  - はじめ`class TreeNodeRef { TreeNode node; }`としたが、`TreeNode.left`への参照が取れなくて困る
  - TreeNodeRefにgetterとsetterを追加し、これを匿名クラスでオーバーライドすればいいと考える
  - 再帰呼び出しの引数に、匿名クラスとして作ったTreeNodeRefの子クラスを渡して、Accepted
    - `mergeTreesHelper(node1.left, node2.left, new TreeNodeRef() { @Override void setNode(TreeNode node) { ... } })`
  - しかし匿名クラスを書くのが煩雑なので、整理したくなる
  - getterは要らないと気づいて削除。メソッド1つなので（匿名クラスでなく）ラムダで書ける。しばらく整理して上の形になった

