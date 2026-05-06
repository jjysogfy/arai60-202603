問題：111. Minimum Depth of Binary Tree https://leetcode.com/problems/minimum-depth-of-binary-tree/

# step 1
- 手を動かす前に、何日か考えたりしていた
- いろいろと選択肢がある
  - DFS
  - BFS
  - 再帰かループか
- とりあえず、再帰でDFSを書く

```java
// step 1
class Solution {
  public int minDepth(TreeNode root) {
    if (root == null) {
      return 0;
    }
    return computeMinDepth(root);
  }

  int computeMinDepth(TreeNode node) {
    if (node == null) {
      return Integer.MAX_VALUE;
    }
    if (node.left == null && node.right == null) {
      return 1;
    }
    int leftDepth = computeMinDepth(node.left);
    int rightDepth = computeMinDepth(node.right);
    return Math.min(leftDepth, rightDepth) + 1;
  }
}
// step 1 コード終わり
```

書いてから思ったこと：
- 再帰の深さの問題がないか、あまり注意できていなかった
  - https://discord.com/channels/1084280443945353267/1235829049511903273/1236256946403807323
  - スタックは1Mくらいらしい。深さ`10^4`も行けば良いほうらしい
  - 今回の制約は、ノードの個数が`10^5`まで。まずい可能性がある
  - ただし、LeetCodeでは通った
    - 設定が変えてあるかもしれない。少なくとも他の言語ではそうらしい。Javaではどうなのかは調べていない
  - 再帰の深さの設定方法を調べておく
    - 「このあたりをとりあえず黙らせるために、Xss オプションをつけて起動するとかは、時々しますね。」とのこと
    - オプションを調べてみる。`java -X`を実行
    - 「`-Xss<size>        javaスレッドのスタック・サイズを設定します`」と出力された
- 今回の問題ではBFSのほうが自然だったか
  - leafにたどり着いた時点で探索を打ち切れる
  - ダイクストラ的な雰囲気
- nullの扱い
  - 内部のメソッドでは、`return Integer.MAX_VALUE;`とした
    - 微妙にややこしいが、番兵としてうまくいく
    - minDepthは`leaves.map(l -> depth to l).min()`という感じで、空集合の下限が無限大というのは自然
  - 他にも方法はありそう
    - あまり`Integer.MAX_VALUE`を持ち出したくない人もいそう？？
    - `computeMinDepth(null)`は呼び出さず、親の方ですべて処理するとか


# step 2
コメント集をざっくり見ておく
- 「上から数字を配っていくか、下から集めてくるかの2方向があって、それぞれ再帰で書くか、スタックとループで書くか、がありますね。」
  - https://discord.com/channels/1084280443945353267/1196472827457589338/1237988315781664770
  - step 1は、下から集める再帰。
    - （こういう再帰をループに直すのは大変。[前回やった](https://github.com/jjysogfy/arai60-202603/pull/10/changes#diff-de4e45038fc947e248157e8a2cedd6e179dcfcfaa6d64e31552eb3ee9bb94da3R10)）
  - 上から配るループも[前回書いた](https://github.com/jjysogfy/arai60-202603/pull/10/changes#diff-3b11adb131cf52e5af58d7d49fa7a4bd8b2e6253247288c0b981f545094247a4R98)
    - 再帰にするのはJavaだと少し面倒かも。Pythonだとnonlocalを使っているのを見かける
    - 書いておく。nonlocalの代わりにAtomicIntegerを使っておく

```java
// step 2 上から配る再帰
class Solution {
  public int minDepth(TreeNode root) {
    if (root == null) {
      return 0;
    }

    AtomicInteger result = new AtomicInteger(Integer.MAX_VALUE);
    computeMinDepth(root, 1, result);
    return result.get();
  }

  void computeMinDepth(TreeNode node, int depth, AtomicInteger minDepthRef) {
    if (node == null) {
      return;
    }
    if (node.left == null && node.right == null) {
      minDepthRef.getAndAccumulate(depth, Integer::min);
    }
    computeMinDepth(node.left, depth + 1, minDepthRef);
    computeMinDepth(node.right, depth + 1, minDepthRef);
  }
}
// step 2 上から配る再帰 終わり
```

もう少し人のコードを見ておく
- https://github.com/goto-untrapped/Arai60/pull/46/changes
- https://github.com/ryoooooory/LeetCode/pull/25/changes

気づいたこと：
- 上から配るDFSは、枝刈りができる
  - `if (depth >= minDepthRef.get()) { return; }`
- unreachableなのをコンパイラが検出できないとき、無駄なreturnを書く必要がある
  - そういうとき、無限ループに直してreturnを消す手がある
  - https://github.com/ryoooooory/LeetCode/pull/25/changes


## 清書
BFSで書いておく。

