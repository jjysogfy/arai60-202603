問題：105. Construct Binary Tree from Preorder and Inorder Traversal
https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/

# step 1

- 40分ぐらい
  - 問題は前に見て考えていた
  - スマホで書いていた
    - 前半20分は調べごとをしていた（スマホでの書き方など）ので、実際に書いたのは20分ぐらい

```java
class Solution {
  public TreeNode buildTree(int[] preorder, int[] inorder) {
    return buildTreeFromLists(
        Arrays.stream(preorder).boxed().toList(),
        Arrays.stream(inorder).boxed().toList());
  }
  
  TreeNode buildTreeFromLists(List<Integer> preorder, List<Integer> inorder) {
    if (preorder.isEmpty()) return null;
    int rootVal = preorder.get(0);
    int numLeftNodes = inorder.indexOf(rootVal);
    TreeNode left = buildTreeFromLists(
        preorder.subList(1, numLeftNodes + 1),
        inorder.subList(0, numLeftNodes));
    TreeNode right = buildTreeFromLists(
        preorder.subList(numLeftNodes + 1, preorder.size()),
        inorder.subList(numLeftNodes + 1, inorder.size()));
    return new TreeNode(rootVal, left, right);
  }
}
```

- 方針
  - preorderの先頭がrootになっている
  - もっといえばこんな感じ：
  - preorder: `root [ left ] [ right ]`
  - inorder: `[ left ] root [ right ]`
  - `inorder.indexOf(root)`の前後で分割すればいい


# step 2
Javaでたくさん解いている方。
- https://github.com/ryoooooory/LeetCode/pull/32/changes
- https://github.com/goto-untrapped/Arai60/pull/53/changes

思ったこと：
- goto-untrapped氏のstep 1のコードはちょっと大変
  - 変数 rootIndex が難しい
    - インスタンス変数になっていると変更を追いづらい気がする。Javaだとちょっと書き換えづらいけど（参照が取れないから）
    - `rootIndex--`もわかりづらいと感じる。基本的に進めるだけだが、時々つじつま合わせのために戻す、みたいに感じる
  - https://github.com/goto-untrapped/Arai60/blob/10206faf2ef26abf2cf23f23599753c08cbcba3e/105.%20Construct%20Binary%20Tree%20from%20Preorder%20and%20Inorder%20Traversal/ConstructBinaryTreeFromPreorderAndInorderTraversalStep1.java
- 再帰呼び出しで、配列をコピーしても間に合う
  - どのみちList.indexOfしているので、時間計算量は同じ（`O(n^2)`）
  - （subListはビューを作るだけ）
  - Solution2_1 https://github.com/goto-untrapped/Arai60/blob/10206faf2ef26abf2cf23f23599753c08cbcba3e/105.%20Construct%20Binary%20Tree%20from%20Preorder%20and%20Inorder%20Traversal/ConstructBinaryTreeFromPreorderAndInorderTraversalStep2.java
- `Map<Integer, Integer> valToInorderIndex`を作っておけば時間計算量が削減できる
  - `O(n^2)`から`O(n)`になる

`Map<Integer, Integer> valToInorderIndex`を使って書いてみる。

```java
// step 2 その1 `valToInorderIndex`と再帰によるコード
class Solution {
  public TreeNode buildTree(int[] preorder, int[] inorder) {
    Map<Integer, Integer> valToInorderIndex = new HashMap<>();
    for (int i = 0; i < inorder.length; ++i) {
      valToInorderIndex.put(inorder[i], i);
    }

    return buildTreeHelper(preorder, valToInorderIndex, 0, 0, inorder.length);
  }

  private TreeNode buildTreeHelper(
      int[] preorder,
      Map<Integer, Integer> valToInorderIndex,
      int preorderBegin, int inorderBegin, int size) {
    if (size <= 0) {
      return null;
    }

    int rootVal = preorder[preorderBegin];
    TreeNode node = new TreeNode(rootVal);

    int sizeOfLeft = valToInorderIndex.get(rootVal) - inorderBegin;
    node.left = buildTreeHelper(preorder, valToInorderIndex,
        preorderBegin + 1,
        inorderBegin, sizeOfLeft);
    int sizeOfRight = size - sizeOfLeft - 1;
    node.right = buildTreeHelper(preorder, valToInorderIndex,
        preorderBegin + 1 + sizeOfLeft,
        inorderBegin + 1 + sizeOfLeft, sizeOfRight);
    return node;
  }
}
// step 2 その1 終わり
// かかった時間：13:37
```

次は、スタックとループで書いてみる。
ついでに、preorderBegin（以下のコードではi）はスタックに載せずに持つことにしてみる。

```java
// step 2 その2 `valToInorderIndex`とスタックによるコード
class Solution {
  private record StackFrame(TreeNode node, int inorderBegin, int size) {
  }

  public TreeNode buildTree(int[] preorder, int[] inorder) {
    Map<Integer, Integer> valToInorderIndex = new HashMap<>();
    for (int i = 0; i < inorder.length; ++i) {
      valToInorderIndex.put(inorder[i], i);
    }

    Deque<StackFrame> stack = new ArrayDeque<>();
    TreeNode root = new TreeNode();
    stack.push(new StackFrame(root, 0, inorder.length));

    for (int i = 0; i < preorder.length; ++i) {
      StackFrame frame = stack.pop();
      TreeNode node = frame.node;
      int inorderBegin = frame.inorderBegin;
      int size = frame.size;

      node.val = preorder[i];
      int inorderIndex = valToInorderIndex.get(preorder[i]);
      int sizeOfLeft = inorderIndex - inorderBegin;
      int sizeOfRight = size - sizeOfLeft - 1;
      if (sizeOfRight > 0) {
        node.right = new TreeNode();
        stack.push(new StackFrame(node.right, inorderIndex + 1, sizeOfRight));
      }
      if (sizeOfLeft > 0) {
        node.left = new TreeNode();
        stack.push(new StackFrame(node.left, inorderBegin, sizeOfLeft));
      }
    }

    return root;
  }
}
// step 2 その2 終わり
// かかった時間：15:51
```

- iが主役の意識があったのでfor文で書いたが、`while (!stack.isEmpty())`のほうが見やすそう
- iを1ずつ増やしたおかげで、`preorderBegin + 1 + sizeOfLeft`のような計算は不要になった


## step 2.2 inorder の頭からの構成
コメント集によると、別の解法がある。inorderを前から読んで木を構成する。
- https://discord.com/channels/1084280443945353267/1478763507963924522/1492871022511390843
- 「こういうこともできるくらいのアルゴリズムで分からなくてもいいだろう。」とのことだが

せっかくなので理解したいと思った。考えたことを簡単にメモしておく
- 観察：
  - inorderは`[ left ] root [ right ]`のような感じだった
  - inorderで隣り合うノードA, Bは、一方がもう一方の祖先
    - （そうでなかったら、共通祖先がinorderでAとBの間に来る）
  - さらに、次のいずれか
    - (i) AがBの祖先のとき。`A.right.left.left.left. ... .left == B`
      - ただし、`.left`は0個のこともある：`A.right == B`
    - (ii) BがAの祖先のとき。`B.left.right.right.right. ... .right == A`
  - (i)(ii)のどちらなのかは、preorderにAとBが現れる順序を見るとわかる
- この観察をもとに、図を書いて、小さい例で動作を見るとわかるかなと思った
  - AIで図示のプログラムを書いている方がいた
    - https://discord.com/channels/1084280443945353267/1478763507963924522/1493048385123647568
  - せっかくなので自分もCodexにやらせてみた
    - [動画](./inorder-visualizer.mp4)
    - （いろいろ未完成だし、そんなに見やすくもないが、とりあえず動いた）
    - ノード「？」は、(i)でA.rightがまだ現れていない（確定していない）状況を表したつもり
- inorderを頭から見ていく
  - ノードAまで見て、次にノードBが来るとする
  - 図を見ながら、どうやって木の形を確定できるかを考える
  - (i)のとき、`A.right`がまだ確定していないので、「？」とおいておく
    - （コードでは、`A.right = B`と仮おきしている）
  - (ii)のとき、Bの子`B.left`はすでに現れているはずなので、それを探す
    - 親が確定していないノード（≒ 図で「？」と直接つながっているノード）の中のどれか
      - （動画の初めの状態なら、1, 2, 3のどれか）
    - 配列preorderを見ればノード同士の上下関係はわかるから、`B.left`を確定できる
      - （動画の初めの状態なら、B=4は1と2の間にあるので、B.leftは2）
    - `B.left.right.right.right. ... .right == A`だから、`B.left`より下の辺も確定する
      - （動画の初めの状態なら、`2.right == 3`が確定）
  - 「親が確定していないノード」は「ひとつながり」になっている
    - （`- A1 - ? - A2 - ? - A3 - ... - ? - An -`という列があり、他の場所に「？」は現れない）
    - 「ひとつながり」なので、スタックで管理できる
- ちょっと難しい。まだすっきりしないような。


## step 2.3 清書

