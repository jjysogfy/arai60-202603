問題（617. Merge Two Binary Trees）：https://leetcode.com/problems/merge-two-binary-trees/


# step 1 解く

```java
// step 1のコード
import java.util.Optional;

class Solution {
  public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
    if (root1 == null && root2 == null) {
      return null;
    }

    int value = 0;
    Optional<TreeNode> maybeRoot1 = Optional.ofNullable(root1);
    Optional<TreeNode> maybeRoot2 = Optional.ofNullable(root2);
    value += maybeRoot1.map(t -> t.val).orElse(0);
    value += maybeRoot2.map(t -> t.val).orElse(0);
    var leftNode = mergeTrees(maybeRoot1.map(t -> t.left).orElse(null),
        maybeRoot2.map(t -> t.left).orElse(null));
    var rightNode = mergeTrees(maybeRoot1.map(t -> t.right).orElse(null),
        maybeRoot2.map(t -> t.right).orElse(null));
    return new TreeNode(value, leftNode, rightNode);
  }
}
// step 1のコード終わり
```

- step 1のコードだけ書いてしばらく（1か月弱？）放置してしまった
  - 書いたときのことをそれほどはっきり覚えていない
- 大体の方針：
  - DFSで構成すればいい
  - 片方の木がnullのケースをちゃんと扱わないといけない
- Optionalを使ってみることにした
  - 煩雑になってしまった
  - mapなんかせず、三項演算子/ifなど使うほうが良さそう


# step 2 読む
Javaでたくさん解いている人のコード
- https://github.com/goto-untrapped/Arai60/pull/47/changes
- https://github.com/ryoooooory/LeetCode/pull/26/changes

見て気づいたこと：
- 片方の木がnullになったら、もう片方の木をreturnしてしまってもよい
  - 使いまわしたくなければ、deepcopyを作ってもよい
- 引数を戻り値に使い回すだけでなく、破壊する方法もある
  - root1を破壊し、`return root1;`する
  - そういう設計なら返り値`void`でも自然かも。root1がnullのとき困るけど
- スタックを使うループで書く方法もある
  - [前回](https://github.com/jjysogfy/arai60-202603/pull/11)でいう「上から配る + ループ」
  - 配るというか、結果の木を上から作る、という感じ
  - （TreeNodeが）可変なのを使っている
- 番兵`new TreeNode(0)`が便利
  - https://github.com/ryoooooory/LeetCode/pull/26/changes#diff-b215107754aa389ea091c7bce4323e2a33519e27d0e57bcd06b4f901f4f1c573R5
  - うまい。言われてみればそうか、という感じ
- 再帰なのでスタックがあふれないか気にするべき
  - また忘れていた気がする
  - https://discord.com/channels/1084280443945353267/1235829049511903273/1236256946403807323
    - 1MB、`10^4`回が目安
  - スタックと言ったが、そもそもJVMのメモリがどのような構造か知らない、と思った
    - JVMはスタックマシンというものだと、前にAIに教えられた。よくわかってないけど
      - そういう事情もあり気になった
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

step 1に近い方針で書く
- Optionalではなく三項演算子で書いてみる
  - 番兵を使うコードは「清書」で
```java
// step 2 その1 step1に近い方針
class Solution {
  public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
    if (root1 == null && root2 == null) {
      return null;
    }

    int val = root1 == null ? 0 : root1.val;
    val += root2 == null ? 0 : root2.val;
    TreeNode merged = new TreeNode(val);
    merged.left = mergeTrees(
        root1 == null ? null : root1.left,
        root2 == null ? null : root2.left);
    merged.right = mergeTrees(
        root1 == null ? null : root1.right,
        root2 == null ? null : root2.right);
    return merged;
  }
}
// step 2 その1終わり
```

「上から配る + 再帰」を書いてみる
- これはJavaだとちょっと書きづいのだった
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
  - TreeNodeRefにgetterとsetterを追加し、これをオーバーライドすればいいと考える
  - 再帰呼び出しの引数に、匿名クラスとして作ったTreeNodeRefの子クラスを渡して、Accepted
  - しかし匿名クラスを書くのが煩雑なので、整理したくなる
  - getterは要らないと気づいて削除。メソッド1つなのでラムダで書ける。しばらく整理して上の形になった

- もう少し他の書き方（root1を破壊、「上から配る + ループ」など）を試しても良いかもしれないけど、進みが遅いのでさっさと進むことに


## step 2.1 清書
清書もstep 1に近い方針でやる

```java
// step 2.1清書
class Solution {
  public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
    if (root1 == null && root2 == null) {
      return null;
    }

    TreeNode node1 = root1 != null ? root1 : new TreeNode(0);
    TreeNode node2 = root2 != null ? root2 : new TreeNode(0);
    TreeNode merged = new TreeNode();
    merged.val = node1.val + node2.val;
    merged.left = mergeTrees(node1.left, node2.left);
    merged.right = mergeTrees(node1.right, node2.right);
    return merged;
  }
}
// step 2.1清書終わり
```

- 番兵でかなりすっきりした


# step 3 覚える

