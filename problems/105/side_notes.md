# ファイル構成
**先に`memo.md`のほうをご覧ください。**


## step 2.2 inorder の頭からの構成
コメント集によると、別の解法がある。inorderを前から読んで木を構成する。
- https://discord.com/channels/1084280443945353267/1478763507963924522/1492871022511390843
- 「こういうこともできるくらいのアルゴリズムで分からなくてもいいだろう。」とのことだが

せっかくなので理解したいと思った。考えたことを簡単にメモしておく
- 観察：
  - `inorder`は`[ left ] root [ right ]`のような感じだった
  - ノードA, Bが`inorder`で隣り合うとする。このとき、一方はもう一方の祖先
    - （そうでなかったら、AとBの共通祖先がinorderでAとBの間に来ることになり矛盾）
  - さらに、次のいずれかが成立
    - (i) AがBの祖先のとき。`A.right.left.left.left. ... .left == B`
      - ただし、`.left`は0個のこともある：`A.right == B`
    - (ii) BがAの祖先のとき。`B.left.right.right.right. ... .right == A`
  - (i)(ii)のどちらなのかは、`preorder`にAとBが現れる順序を見るとわかる
- この観察をもとに、図を書いて、小さい例で動作を見るとわかるかなと思った
  - AIで図示のプログラムを書いている方がいた
    - https://discord.com/channels/1084280443945353267/1478763507963924522/1493048385123647568
- せっかくなので自分もCodexにやらせてみた
  - [動画](./inorder-visualizer.mp4)
  - （いろいろ未完成だし、そんなに見やすくもないが、とりあえず動いた）
  - ノード「？」は、(i)でA.rightがまだ現れていない（確定していない）状況を表したつもり
- `inorder`を頭から見ていく
  - ノードAまで見て、次にノードBを見るとする
  - 図を見ながら、どうやって木の形を確定できるかを考える
  - (i)のとき、`A.right`がまだ確定していないので、「？」とおいておく
    - （リンクにあるコードでは、`A.right = B`と仮おきしている）
  - (ii)のとき、Bの子`B.left`はすでに見ているはずなので、それを探す
    - 親が確定していないノード（≒ 図で「？」と直接つながっているノード）の中のどれか
      - （動画の初めの状態なら、1, 2, 3のどれか）
    - 配列preorderを見ればノード同士の上下関係はわかるから、`B.left`を確定できる
      - （動画の初めの状態なら、B=4は1と2の間にあるので、B.leftは2）
    - `B.left.right.right.right. ... .right == A`だから、`B.left`より下の辺も確定する
      - （動画の初めの状態なら、2より下の辺`2.right == 3`が確定）
  - 「親が確定していないノード」は「ひとつながり」になっている
    - （`- A1 - ? - A2 - ? - A3 - ... - ? - An -`という列があり、他の場所に「？」は現れない）
    - 「ひとつながり」なので、スタックで管理できる
- なんとなくわかってきたが、ちょっと難しい。まだすっきりしないような。

コードでもあらわしてみる。

```java
// step 2.2 inorder の頭からの構成
class Solution {
  public TreeNode buildTree(int[] preorder, int[] inorder) {
    if (preorder.length == 0) {
      return null;
    }

    Deque<TreeNode> undetermined = new ArrayDeque<>();

    Map<Integer, Integer> valueToPreorderIndex = new HashMap<>();
    for (int i = 0; i < preorder.length; ++i) {
      valueToPreorderIndex.put(preorder[i], i);
    }

    for (int value : inorder) {
      TreeNode node = new TreeNode(value);

      Optional<TreeNode> maybeLeftChild = Optional.empty();
      while (!undetermined.isEmpty()) {
        TreeNode top = undetermined.peek();
        if (valueToPreorderIndex.get(top.val) < valueToPreorderIndex.get(value)) {
          break;
        }
        maybeLeftChild = Optional.of(undetermined.pop());
      }

      maybeLeftChild.ifPresent(leftChild -> node.left = leftChild);

      if (!undetermined.isEmpty()) {
        undetermined.peek().right = node;
      }

      undetermined.push(node);
    }

    return undetermined.peekLast();
  }
}
// step 2.2 inorder の頭からの構成 おわり
// かかった時間：27分ぐらい
```

