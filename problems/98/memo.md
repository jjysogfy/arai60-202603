問題：98. Validate Binary Search Tree https://leetcode.com/problems/validate-binary-search-tree/

# step 1 解く
- （書くのにかかった時間は覚えていない）
- DFSでできるだろう、と考えた

```java
class Solution {
  public boolean isValidBST(TreeNode root) {
    return isValidBstContainedIn(root, Integer.MIN_VALUE, Integer.MAX_VALUE);
  }

  private boolean isValidBstContainedIn(TreeNode node, long left, long right) {
    if (node == null) {
      return true;
    }
    if (!(left <= node.val && node.val <= right)) {
      return false;
    }
    return isValidBstContainedIn(node.left, left, (long) node.val - 1)
        && isValidBstContainedIn(node.right, (long) node.val + 1, right);
  }
}
```

- オーバーフローの処理について
  - 制約を見て、意識しなければいけないことは念頭にあった
  - `isValidBstContainedIn(node, left, right)`のleft, rightがinclusiveなのは、その意識から
  - `node.val - 1`してるので、この方法ではintでなくlongにしておく必要がある
    - こちらは気づかず、1ミス
  - longなんか使わず、もっと良い方法がありそうとは思った


# step 2
## step 2.1 読む
Javaでたくさん解いている人のコードを見る。
- https://github.com/ryoooooory/LeetCode/pull/31/changes
- https://github.com/goto-untrapped/Arai60/pull/52/changes

知ったこと・思ったこと：
- 変数名left, right
  - low, highとかのがわかりやすいか
  - 区間という意識が強かった
- low, highはexclusiveの方が自然だったかも
- 「気の所為ならばいいのですが、なんとなく、コーディングの練習が受け身であるのを感じています。」
  - どのあたりがそうなのかわからなかった。自分も気をつけたい
- 別解：inorderが（狭義）sorted
  - なるほど、全く気づかなかった
- 別解：`boolean isValidBstHigherThan(TreeNode node, long* min)`のような関数を書く
  - （nodeが表す木が、valid bstかつvalすべてがminより大きいとき、true）
  - minには、nodeが表す木の最大値を入れて返す
  - Javaに`long*`はないが、`long[]`や`AtomicLong`、インスタンス変数、返り値を増やすことなどで対応できる
    - インスタンス変数を使うと、2回呼べない、スレッドセーフでなくなる、といったデメリットがある。今回は避けたほうがいい
- Integerを使う方法
  - nullをinfinity / -infinity代わりにする
  - 不要なboxingを好まない人もいる
- longを使うことについて
  - ちょっと気持ち悪いと感じていた
  - でも、boxingよりパフォーマンスは良いと思うし、そんなに変な選択ではないかも？？

再帰による解法いくつかを整理しておく：
- どの解法も条件`inorder[i] < inorder[i + 1]`を確かめていることになる
  - （`int[] inorder`はinorderでnode.valを並べたもの）
- 実際にinorderを作る方法
- DFSで、`inorder[i + 1]`のノードで条件をチェックする方法
- DFSで、子のノードに条件をチェックさせる方法
  - step 1
  - BFSでも良い
  - これ、無駄な比較をけっこうしている

```java
// step 2.1 その1 再帰でinorderリストを作る方法
class Solution {
  public boolean isValidBST(TreeNode root) {
    List<Integer> inorder = new ArrayList<>();
    toInorder(root, inorder);

    for (int i = 0; i + 1 < inorder.size(); ++i) {
      if (!(inorder.get(i) < inorder.get(i + 1))) {
        return false;
      }
    }
    return true;
  }

  private void toInorder(TreeNode root, List<Integer> inorder) {
    if (root == null) {
        return;
    }
    toInorder(root.left, inorder);
    inorder.add(root.val);
    toInorder(root.right, inorder);
  }
}
// step 2.1 その1 終わり
```

- かかった時間は6:45
  - 再帰呼び出しでinorder渡し忘れて1ミス


inorder traversalのループによる書き方
- https://github.com/goto-untrapped/Arai60/pull/52/changes#diff-703a030a8806530bbdad9e8f2d3e687104684fb7311e0f2a34d321c75ee7bcfbR69
- 素直な再帰inorder traversalからの変形として理解できる
  - https://discord.com/channels/1084280443945353267/1200089668901937312/1213043194242015252
    - 「B を末尾再帰最適化してループにすると、」のコードは内側の`while node`が余計だと思う
    - `if node is None: return`も無駄だと思う
  - あまりわかった気がしない……。
    - 再帰のループへの書き換えについて、理解が浅いのかも
- いくつかの変形：
  - https://discord.com/channels/1084280443945353267/1200089668901937312/1213360267673473044
  - https://discord.com/channels/1084280443945353267/1200089668901937312/1213066250737946664
- 追加メモリO(1)の方法がある
  - Morris inorder traversalとよばれる
  - ただし TreeNode.right を一時的に書き換える
  - https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?pli=1&tab=t.0#heading=h.9q172vpc94vh


## step 2.2 清書
ループでinorder traversalを書くのをあまり理解できていない気がする。
慣れるためにその方針で練習してみる。

```java
// step 2.2 清書
class Solution {
  public boolean isValidBST(TreeNode root) {
    long low = Long.MIN_VALUE;
    Deque<TreeNode> nodes = new ArrayDeque<>();
    pushLeftChildren(root, nodes);
    while (!nodes.isEmpty()) {
      TreeNode node = nodes.pop();
      if (!(low < node.val)) {
        return false;
      }
      low = node.val;
      pushLeftChildren(node.right, nodes);
    }
    return true;
  }

  private void pushLeftChildren(TreeNode node, Deque<TreeNode> nodes) {
    while (node != null) {
      nodes.push(node);
      node = node.left;
    }
  }
}
// step 2.2 清書 終わり
```

今の理解：
- inorderはnode.left, node, node.rightの順の処理だった
- スタックからnodeをpopしたとき、
  - node.leftは処理済み
  - node自身と、node.rightを処理する
    - node自身は自分で処理して、node.rightはスタックに積んで子に任せる
- 再帰版との関係はよくわからなくてモヤモヤしている


# step 3 3回書く
1回目：7:21、2回目：3:11、3回目（ミスあり）：3:37

- ミス
  - return true忘れ
