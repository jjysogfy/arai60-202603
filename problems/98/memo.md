98. Validate Binary Search Tree
https://leetcode.com/problems/validate-binary-search-tree/

# step 1
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
Javaでたくさん解いている人のコードを見る。
- https://github.com/ryoooooory/LeetCode/pull/31/changes
- https://github.com/goto-untrapped/Arai60/pull/52/changes

知ったこと・思ったこと：
- 変数名left, right
  - low, highとかのがわかりやすいか
  - 区間という意識が強かった
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
- inorder traversalの巧妙な書き方
  - https://github.com/goto-untrapped/Arai60/pull/52/changes#diff-703a030a8806530bbdad9e8f2d3e687104684fb7311e0f2a34d321c75ee7bcfbR69

